# 🧠 TryHackMe Room: Net Sec Challenge

**Room URL:** [https://tryhackme.com/room/netsecchallenge](https://tryhackme.com/room/netsecchallenge)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all 8 questions in this challange room, focused on the skills obtained via the Network Security module.  

---

## 🧰 Tools I Used  
- Nmap  
- Telnet  
- Hyrda  

---

## 🛠️ TASK 1: What is the highest port number being open less than 10,000?  

The first few questions will all focus on basic nmap scans of the target. We can get the answers to all 3 of the first question in one scan. It is worth to not that by default nmap will scan the most common 1,000 ports for each protocol. This does not mean it will scan ports 0-1000 but the most common 1,000 ports in use. As question 2 asks for the port open above 10,000 we can just do a full port scan from 0 to 65,535 to get underway.

This can be achived via ```sudo nmap -sS 10.10.79.185 -T5 -p-```  
#### ⚙️ **Options**  
**sudo** We need to run this command as a privilged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.  
**-T5** Tells Nmap perform the scan at the fastest level of "Insane" Definatly not recommanded, but works for this case and reduces time.  
**-p-** Tells Nmap to scan all ports.  

---

## 🛠️ TASK 2: There is an open above 10,000. What is it?  

---

## 🛠️ TASK 3: How many TCP ports are open?  

---

## 🛠️ TASK 4: What is the flag hidden in the HTTP server header?  

---

## 🛠️ TASK 5: What is the flag hidden in the SSH server header?  

---

## 🛠️ TASK 6: What is the version of the FTP server?  

---

## 🛠️ TASK 7: What is the flag hidden in one of these two account files and accessible via FTP?  

---

## 🛠️ TASK 8: What is the flag when you solve the challenge?

---
