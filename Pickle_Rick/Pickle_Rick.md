# 🧠 TryHackMe Room: Pickle Rick  

**Room URL:** [https://tryhackme.com/room/picklerick](https://tryhackme.com/room/picklerick)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

**Objective:**  
This guide covers the steps required to solve all tasks in the Pickle Rick room. The challenge involves web enumeration, exploiting a command panel, and retrieving three hidden ingredients (flags). Due to one of the most common commands to output file contents being blocked, this guide will cover three alternative ways to view the flags.  

---

## 🧰 Tools I Used  
- nmap  
- gobuster
- Python3 http server

---

Firstly let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  

---

## 🛠️ Initial Recon & Enumeration 

Once the target machine is up and running, start with an nmap scan to identify open ports:  


```bash
sudo nmap -sS TARGET_IP -T5 -o ports.txt
```

#### ⚙️ **Options**   
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells nmap to perform a TCP SYN Scan.  
**-T5** Tells nmap to perform the scan at the fastest speed possible. This is acceptable for basic CTF rooms, but likely not in real scenarios.  
**-o** Tells nmap to output the scan results into a file.  

Once the scan has completed the below result is returned.  
![Recond - nmap scan 1.png](./Images/Recon%20-%20Nmap%20scan%201.png)  

This indicates port 80 is open for the web server and port 22 for SSH.  

A further nmap version scan can be performed to identify services and any vulnerabilities.  

``` bash
sudo nmap -sS -sV -sC TARGET_IP -p 22,80
```
#### ⚙️ **Options**   
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells nmap to perform a TCP SYN Scan.  
**-sV** Tells nmap to perform version scanning, which is required for running scripts.  
**-sC** Tells nmap to run default scripts against the ports.  
**-p** Tells nmap which ports to scan.  

This scan yields the below results.  
![Recon - nmap scan 2.png](./Images/Recon%20-%20Nmap%20scan%202.png)

Here we learn the web server is ```Apache HTTP Server``` version 2.4.41 and upon some research this does not yield any tangible exploits to use.

Now to turn our attention to the webserver. Firstly, with Gobuster for some directory enumeration.

```bash
gobuster -u TARGET_URL -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -t 200 -x .php -o files.txt  
```
#### ⚙️ **Options**   
**-u** Used to provide the target url to scan.  
**-w** Path to wordlist used for enumeration.   
**-t** Instructions gobutser to scan using 200 concurrent threads. This is acceptable for basic CTF rooms, but likely not in real scenarios. 
**-x** Used to append .php onto directories. This will be helpful as the severs is running PHP and can help to yield results like ```login.php```, which may otherwise be missed.
**-o** Outputs the results into a text file.  
![Recon - Gobuster scan.png](./Images/Recond%20-%20Gobuster%20scan.png)  

This yielded good results, for further investigation.  
```text
denied.php
login.php
portal.php
robots.txt
```
![Recon - Gobuster result.png](./Images/Recond%20-%20Gobuster%20result.png)

Open the website and review the source code for anything interesting. 
![Recond - Sourcecode.png](./Images/Recon%20-Sourcecode.png)  

Within the source code we find a username has been left ```R1ckRul3s``` this is most likely to be used for the ```login.php``` page.


For the final item of initial recon navigate to the ```robots.txt``` page via
```http
http://TARGET_IP/robots.txt
```
![Recon - Robots.png](./Images/Recond%20-%20Robots.png)  
This looks promising, possible a password to go with the found username. 


Navigate to the ```login.php``` page
```http
http://TARGET_IP/login.php
```
![Recond - Login Page.png](./Images/Recon%20-%20Login%20Page.png)  

Try entering the found username and possible password.  
```text
Username: R1ckRul3s
Password: Wubbalubbadubdub
```

Success! The found login details worked and there is now a usable ```Command Panel``` to use for flag capturing.  
![Recon - Logged in.png](./Images/Recon%20-%20Logged%20in.png)

---

## 🛠️ Flag 1 

Now that login has been achieved and a ```Command Panel``` has been provided to use.  

Perform some simple recon via commands like   
```bash
id
whoami
pwd
ls
```    
```ls``` provides some good information about the files in the current working directory. Where there are two files of interest ```Sup3rS3cretPickl3Ingred.txt``` and ```clue.txt```  
![Flag1 - ls.png](./Images/Flag1%20-%20ls.png)  

The first flag has been found. Try to output the file via  
```bash
cat Sup3rS3cretPickl3Ingred.txt
```  
Access is denied because the ```cat``` command has been disabled in the challenge.    
![Flag1 - cat fail.png](./Images/Flag1%20-%20cat%20fail.png)  

As the ```cat``` command has been disabled, alternative methods will need to be used to output the file contents. In this first example a python3 HTTP server will be started to download the files to the attacking machine.  

Start the server via
```bash
python3 -m http.server 5050
```
#### ⚙️ **Options**   
**-m** Allows python module as a script directly from the command line, using the module's name rather than its file path.  
**5050** Is the chosen port to open for the HTTP server.  
![Flag1 - Server](./Images/Flag1%20-%20Server.png)  
  
Now on the attacking machine use ```wget``` to download both files of interest via.
```bash
wget http://TARGET_IP:5050/Sup3rS3cretPickl3Ingred.txt
wget http://TARGET_IP:5050/clue.txt
```
![Flag1 - wget](./Images/flag1%20-%20wget.png)  

This method works because the command panel allows execution of Python, so we can spin up a simple HTTP server to transfer files to our attacking machine.  


Now that the files are on the attacking machine, use ```cat``` to output the content.
```bash
cat Sup3rS3cretPickl3Ingred.txt
```
![Flag1 - file](./Images/Flag1%20-%20file.png)  

The same can be repeated for the ```clue.txt``` file if required or if you feel you are stuck.   


---

## 🛠️ Rabbit Hole



----

## 🛠️ Flag 2



---

## 🛠️ Flag 3



---

## 🧠 Takeaways  

---

Thank you for following through my guide of the TryHackMe Pickle Rick room. 


