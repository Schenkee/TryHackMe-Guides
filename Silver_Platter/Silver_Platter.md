# 🧠 TryHackMe Room: Silver Platter

**Room URL:** [https://tryhackme.com/room/silverplatter](https://tryhackme.com/room/silverplatter)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

**Objective:**  
This guide covers the steps required to solve all tasks in the Silver Platter room. This challenge involves web enumeration, login panel brute forcing and exploiting a web vulnerability. Once initial access is gained the challenges changes to some Linux privilege escalation. Overall, this was an enjoyable room that added value to my skill set. There is a stronger focus on manual enumeration and research in this room rather than reliance on automated tools. I must admit I had a peek at guide once when nothing was returned for one of the HTTP ports, and I was a bit lost.  

---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- Gobuster
- ffuf
- CeWL
- Burp Suite  

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  If needed, you can add ```silverplatter.thm``` along with the assigned IP to your ```/etc/hosts``` file which can make life easier than remembering an IP all the time.  

---

## 🛠️ Recon & Enumeration 

Start by scanning the target for open ports. Here I used RustScan which will scan for open ports and then pass these to nmap for detailed port scans. 
```bash
rustscan -a TARGET_IP -- -A
```
#### ⚙️ **Options**   
**-a** Use to list the IPs, CIDRs or hosts to be scanned.  
**-A** Passes the IP and open ports to nmap to perform a full ```-A``` nmap scan.   
Note that we need to add the double dash ```--``` after the IP to tell RustScan that we want nmap to perform an ```-A``` scan.  
![Recon - Rustscan.png](./Images/Recon%20-%20Rustscan.png)  

After a little while nmap will finish and return a large chunk of information. Most of which is not that helpful for us.  
```
PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 60 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 be:27:25:10:11:a7:da:1a:ed:e8:0b:ed:3c:a5:12:37 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGsZbbNmEI4aDTtA75zX7bXuo/EOA1VnJjvTW0XN9xpDLiiIweMtikXV4/oTVVqpjV2usmDsRQZKJN9avEg7Y7c=
|   256 10:73:60:7e:9c:4f:bf:89:a8:aa:53:1d:53:37:43:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICogOFH8ttfCh9/T22fDzVfeYwPLZJyE6TUCTvr+6hvf
80/tcp   open  http       syn-ack ttl 60 nginx 1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy syn-ack ttl 59
```
Really the main thing we learnt here is that we have three ports open being ```22 - SSH, 80 - HTTP, 8080 - HTTP```  

As such I turned my attention to some web enumeration. Firstly, via Gobuster on port 80. As port 80 is the default http port we do not need to specify the port.  
```bash
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 200 -x .txt -o ~/THMCTF/files.txt
```
#### ⚙️ **Options**   
**-u** Provides the target url to Gobuster.    
**-t** Sets the concurrent threads to 200. Not advisable in a real engagement.  
**-x** Tells Gobuster to append .txt to the end of words to try locate .txt files such as ```robots.txt```  
**-o** Used to output the result of the scan to a text document.    

![Recon - gobuster.png](./Images/Recon%20-%20gobuster.png)  
This returned a couple of items to look at later.  
```
/images (Status: 301) [Size: 178] [--> http://silverplatter.thm/images/]
/assets (Status: 301) [Size: 178] [--> http://silverplatter.thm/assets/]
/README.txt  (Status: 200) [Size: 771]
/LICENSE.txt (Status: 200) [Size: 17128]
```
Next I scanned for subdomains using ffuf as below
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -H "Host: FUZZ.silverplatter.thm" -u http://TARGET_IP
```
#### ⚙️ **Options**   
**-w** Path to the wordlist used to input into the ```FUZZ``` position.  
**-H** Is used to add a header, in this instance the ```Host``` header.  
**-u** Provides the target url to ffuf.

This will return a huge amount of information so after a couple of seconds stop the command via ```CTRL+C``` and review the output. This will show responses in error have a size of ```14124``` as such we can run ffuf again and filter these out.  
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -H "Host: FUZZ.silverplatter.thm" -u http://TARGET_IP -fs 14124
```
#### ⚙️ **Options**   
**-w** Path to the wordlist used to input into the ```FUZZ``` position.  
**-H** Is used to add a header, in this instance the ```Host``` header.  
**-u** Provides the target url to ffuf.  
**-fs** Used to filter out the specific HTTP response size.  

Unfortunately even after this no tangible results were returned. I repeated the same on port ```8080``` and this also returned nothing of note.  

As such I manually navigated to ```http://silverplatter.thm``` after reviewing the different pages and the items found by Gobuster. You should come to the Contact page. This page contains our first two clues.  

Here we find something called ```Silverpeas``` and a username ```scr1ptkiddy``` 
![Recon - website.png](./Images/Recon%20-%20website.png)  
Having a look at the internet I found ```https://www.silverpeas.org/``` which looked to be the product mentioned on the content page. This must be what is running on port 8080.  

I had a look around the website and at the very bottom of the ```https://www.silverpeas.org/installation/installationV6.html``` page we can find the next clue. As suspected Silverpeas is running on port 8080 and can be reached via 
```url
http://silverplatter.thm:8080/silverpeas
```
![Recon - silverpeas login.png](./Images/Recon%20-%20Silverpeas%20login.png)  

---

## 🛠️ Initial Access

So, at this stage we have found a login page. We have a possible username, so let’s open up Burp Suite and see if we can capture a login request and brute force our way in.  

Once Burp Suite is running, open the Burp Suite browser and navigate to 
```url
http://silverplatter.thm:8080/silverpeas
```
Turn on Proxy and enter something into the ```Login``` and ```Password``` fields and try logging. Burp Suite should capture the below request
![Recon - burp request.png](./Images/Recon%20-%20burp%20request.png)  

Send this request to Intruder. Here we can edit the ```Login``` to be ```scr1ptkiddy``` and add a Position to the password input.  
![Recon - Burp setup.png](./Images/Recon%20-%20Burp%20setup.png)  

Now before we can start, we need a wordlist. The intro for the room states ```password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist- that's how 'cool' they are)```  

Ok so we need to generate our own password list, which can be done with a tool called CeWL (Custom Word List generator).
```bash
cewl http://silverplatter.thm > list.txt
```
![Recon - cewl.png](./Images/Recon%20-%20cewl.png)  
Perfect now we can pass this word list into Burp Suite.  

On the Burp Suite Intruder page in the ```Payload configuration``` select ```Load``` and find the created wordlist.  
![Recon - Burp Setup 2.png](./Images/Recon%20-%20Burp%20Setup%202.png)  

Now start the attack. This will take some time, but if you filter by ```Length``` you should eventually see a word with a much larger length and this is likely the password.  
![Recon - Burp Password.png](./Images/Recon%20-%20Burp%20Password.png)  

Try logging into the website
```url
http://silverplatter.thm:8080/silverpeas
```
If the password found was indeed the correct one login should work.  

![Recon - logged in.png](./Images/Recon%20-%20logged%20in.png)  

Perfect now the first thing I noticed was the ```1 unread notification``` which when clicked on did not contain anything much. But pay close attention to the URL
```url
silverplatter.thm:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=5
```
![Recon - messages.png](./Images/Recon%20-%20messages.png)  
I did a quick google for Silverpeas IDOR and found this [https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47323](https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47323)  

So we can just change the ```ID=X``` and read other messages.  If message 6 is read it returns a user's SSH details.  
![Recon - SSH found.png](./Images/Recon%20-%20SSH%20found.png)  

At this stage we have gained access to the Silverpeas instance and some SSH login details.  

---

## 🛠️ Flag 1 

Use the found SSH login details to try gaining access to the host machine.
```bash
ssh tim@TARGET_IP
```
This should have logged you into the machine as the user tim.  

Here we can locate the first flag ```user.txt```  
```bash
ls
cat user.txt
```
![Flag1 - flag.png](./Images/Flag1%20-%20flag.png)  

---

## 🛠️ Flag 2

Now we know the second flag will require root access as the question states ```What is the root flag?``` as such we will need to work on escalating our privileges.  

I ran through some basic Linux privilege escalation checks
```bash
sudo -l
find / -type f -perm -04000 -ls 2>/dev/null
id
```
I was hoping to find something with a SUID bit set as I always enjoy using GTFObins techniques. But the only thing that stood out was the odd extra group when running ```id```
```bash
id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
```
![Flag2 - ID.png](./Images/Flag2%20-%20ID.png)  

The ```(adm)``` group is not something normal to see for a low privilege user. After some online checking I found this writeup [https://medium.com/@evyeveline1/im-in-the-adm-group-can-i-escalate-yes-eventually-9475b968b97a](https://medium.com/@evyeveline1/im-in-the-adm-group-can-i-escalate-yes-eventually-9475b968b97a)  
The main takeaway here is that a user in the ```(adm)``` group can read log files stored in ```/var/log```
Navigate to this directory and we can see a load of log files. 
```bash
cd /var/log
/var/log$ ls
```
![Flag2 - var.log.png](./Images/Flag2%20-%20var.log.png)  

Now we need to search all these logs for passwords. Luckily for us Grep will do all the hard work here
```bash
grep -ir "password" 
```
#### ⚙️ **Options**   
**-i** Tells Grep to ignore case so it will check for "password", "PASSWORD" or any other combination of the word password and different case.   
**-r** Tells Grep to recursively look through all the files in the directory.  
Combined as **-ir**

This returns quite a bit of information. But towards the bottom is a mention of ```postgresql``` where a password has been entered in clear text into the terminal by a user called ```tyler``` 
![Flag2 log output.png](./Images/Flag2%20log%20output.png)  
Log output below. Password redacted  
```bash
uth.log.2:Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=XXXXXXXXX -v postgresql-data:/var/lib/postgresql/data postgres:12.3
```

Now we have located a possible password to the user ```tyler``` so let’s try and see if we can switch to this user.
```bash
su tyler
```
If successful you should now be logged in as the user ```tyler```   
![Flag2 - tyler.png](./Images/Flag2%20-%20tyler.png)  

Some quick recon shows this user has full ```sudo``` privileges on this machine.  
![Flag2 - sudo -l.png](./Images/Flag2%20-%20sudo%20-l.png)  

As such changing to root should be possible as below
```bash
sudo su
```
You should now be logged in as the root user.  
![Flag2 - sudo.png](./Images/Flag2%20-%20sudo.png)  

The root flag should be in the ```root``` folder and we can get the flag as below.  
```bash
cd root
ls
cat root.txt
```  
![Flag2 - flag.png](./Images/Flag2%20-%20flag.png)  

All done both flags have now been found!  

---

## 🧠 Takeaways  

- Learnt the need for and importance of manual research and enumeration and that one cannot always rely on automated tools.  
- Gained further knowledge about Linux groups in this case the ```(adm)``` group.  
- Hands-on experience with a new port scanning tool RustScan.  
- Continued experience in enumeration, initial access and privilege escalation.  

---

Thank you for following through my guide of the TryHackMe Silver Platter room.
