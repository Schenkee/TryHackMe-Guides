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

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. I using the TryHackME AttackBox lets start this up aswell, or alternativly as in my case connect to the TryHackMe VPN to use your own attacking machine. You can even download the VM for offline usage if required.

---

## 🛠️ Recon: 
First let us start with a straight forward nmap scan to see which ports are open and what possible path we have into the target machine.  
We will run nmap via the following ```sudo nmap -sS TARGET_IP```   
#### ⚙️ **Options**  
**sudo** We need to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.  

This should return the below information.
![Recon - Nmap](./Images/Recon%20-%20Nmap.png)
This will give us enough information to answer the first question: How many ports are open with a port number under 1000?  


---

## 🛠️ Gain Access: 


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
