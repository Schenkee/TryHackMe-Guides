# 🧠 TryHackMe Room: Rabbit Store

**Room URL:** [https://tryhackme.com/room/rabbitstore](https://tryhackme.com/room/rabbitstore)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

**Objective:**  This guide walks through solving the Rabbit Store room. This challenge involves multiple rounds of web enumeration. The exploitation of three different web vulnerability to gain initial access to the host system. Once initial access has been gained privilege escalation is achieved via enumerating a RabbitMQ instance.


---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- dirsearch  
- ffuf  
- Caido  
- rabbitmqctl

Durning this challange I experimented with a couple of different tools I have not used before being dirsearch and Caido. Dirsearch is another directory bruteforce tool like Gobuster and Caido is a Burp Suite alternative. It is always good to have hands-on experience with multiple tools even if they serve the same purpose.  

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. The room author reccomands to use a personal machine connected via VPN to tackle this challange, rather than the AttackBox. For this room I would also highly reccomand adding `rabbitstore.thm` to your `/etc/hosts` file as there will be multiple further domains that need to be accessed via the same IP address later on in the room.

---

## 🛠️ Recon & Enumeration 

As always I started out with some simple port and directory scans against the targt.  

RustScan as below.  
```bash
rustscan -a rabbitstore.thm -- -A
```
#### ⚙️ **Options**   
**-a** Use to list the IPs, CIDRs or hosts to be scanned.  
**-A** Passes the IP and open ports to nmap to perform a full ```-A``` nmap scan.   
Note that we need to add the double dash ```--``` after the IP to tell RustScan that the following arguments should be passed to nmap. In this instance ```-A```  
![Recon - RustScan](./Images/Recon%20-%20RustScan.png)  

This scan resulted in a view interesting items to add to our initial notes.
```
PORT      STATE SERVICE REASON         VERSION                                
22/tcp    open  ssh     syn-ack ttl 60 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)                                                  
80/tcp    open  http    syn-ack ttl 60 Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://cloudsite.thm/
4369/tcp  open  epmd    syn-ack ttl 60 Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown syn-ack ttl 60
```
Here we can see a few open ports.  
Port 22 `SSH` on testing this was configured via password login. So maybe usable later if we find a credential.  

Port 80 `HTTP` the main interesting item to note here is that visiting `http://rabbitstore.thm` caused a redirect to `http://cloudsite.thm` as such we can add `cloudsite.thm` to our `/etc/hosts` file.  

Port 4369 `epmd` the Erlang Port Mapper Daemon `epmd` is a built-in component that helps Erlang-based applications (including RabbitMQ) discover each other’s distribution ports.

Port 25672 `rabbit` RabbitMQ is an open-source message broker software, also known as message-oriented middleware, that facilitates communication between distributed applications using a message queue model. 

We will come back to `epmd` and `rabbit` later on in the challange once initial access has been achived. 

Now I ran a dirsearch scan against the new found redirect address `http://cloudsite.thm` as below
```bash
dirsearch -u http://cloudsite.thm
```
#### ⚙️ **Options**   
**-u** Use to list the IPs, CIDRs or hosts to be scanned.  
Note that dirsearch has a default wordlist that will be used if no specific wordlist is provided. This wordlist is comparable to notable lists in finding directories.  
![Recon - dirsearch-cloud.png](./Images/Recon%20-%20dirsearch-cloud.png)  
This did not return a great deal of useful information.  

Lastly before manually checking the website I ran ffuf to check for Vhosts.
```bash
ffuf -H "Host: FUZZ.cloudsite.thm" -H "User-Agent: Test" -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://cloudsite.thm -fc 302
```  
#### ⚙️ **Options**   
**-H** Used to set the `HOST` header. Required discover vhost servered by the same IP/URL as provided via `-u`  
**-H** Not stricktly needed, but sometimes filtering will requrie the `User-Agent` header has been set and has a value.   
**-c** Tells ffuf to colourise the output. Makes it easier to see results.  
**-w** Provides the path to the wordlist to use in the `FUZZ` position.  
**-u** Provides the url to the target.  
**-fc** Used to list HTTP response codes to ignore. In this case `302` as we don't need to see redirects. 
![Recon - ffuf.png](./Images/Recon%20-%20ffuf.png)  

Here we get a nice hit on `storage` and we can now add this to our `/etc/hosts` file as `storage.cloudsite.thm`  

At this stage I navigated to `http://cloudsite.thm` in my browser and looked around the website for clues. Overall it was very uneventful as 99% of content was just placeholder. But there was a login/signup panel, as such I created myself an account and logged in to see what would be returned.  When logging in I was presented with the below message.
![Recon - login error.png](./Images/Recon%20-%20login%20error.png)  

At the url `http://storage.cloudsite.thm/dashboard/inactive` so my first instinct here was just to change the url to `http://storage.cloudsite.thm/dashboard/active` and see what this would do. 

Now we got an more helpful error.  
![Recon - Token error.png](./Images/Recon%20-%20Token%20error.png)  
So it seems we need to have a Cookie/Token set with some specific parameters to allow access to the `http://storage.cloudsite.thm/dashboard/active` portion of the website. 

This for me is where I ended the recon section as we now know the most likley way into the website for initial access.  

---

## 🛠️ Initial Access

### 🔑 Mass Assignment Vulnerability

Here I started Caido, (Burp Suite can be used) to capture our login request and see what is going on.  

Below is the request and response sent to the website when capturing the login request.  
![MAV - inactive login request.png](./Images/MAV%20-%20inactive%20login%20request.png)  
![MAV - inactive login response.png](./Images/MAV%20-%20inactive%20login%20response.png)  
Here we can see the server response with inactive and also with a `jwt` cookie.  

There are also a couple of other items to note here. First we can see the `X-Powered-By` value is Express and we also have a new endpoint `/api/login` which can be added to our notes to check later.  

If we inspect this `jwt` via [www.jwt.io/](www.jwt.io/)  we can see that the subscription value is inactive `"subscription":"inactive"`  
![MAV - JWT.IO - inactive.png](./Images/MAV%20-%20JWT.IO%20-%20inactive.png)  

So here I decided to see if I can register a new account and capture the request in Caido to modify and pass in the argumrnt `"subscription":"active"` in the HTTP request.  
![MAV - active register request.png](./Images/MAV%20-%20active%20register%20request.png)  
![MAV - active register response.png](./Images/MAV%20-%20active%20register%20response.png)  

Note here on another endpoint `/api/register` to add to our notes.  

Success the server accepted our passed in value `"subscription":"active"` and returned us a `jwt` with an active status.  
![MAW - JWT.IO - active.png](./Images/MAV%20-%20JWT.IO%20-%20active.png)






---


### 🌐 Server-Side Request Forgery (SSRF) Vulnerability

---

### 🧩 Server-Side Template Injection (SSTI) Vulnerability


---

## 🛠️ Flag 1 



---

## 🛠️ Flag 2


---

## 🧠 Takeaways  


---

Thank you for following through my guide of the TryHackMe Rabbit Store room.
