# 🧠 TryHackMe Room: Net Sec Challenge

**Room URL:** [https://tryhackme.com/room/netsecchallenge](https://tryhackme.com/room/netsecchallenge)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all 8 questions in this challenge room, focused on the skills obtained via the Network Security module.  

---

## 🧰 Tools I Used  
- Nmap  
- Telnet  
- Hydra  

---

Firstly lets start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  

---

## 🛠️ TASK 1: What is the highest port number being open less than 10,000?  

The first few questions will all focus on basic nmap scans of the target. We can get the answers to all 3 of the first question in one scan. It is worth to not that by default nmap will scan the most common 1,000 ports for each protocol. This does not mean it will scan ports 0-1000 but the most common 1,000 ports in use. As question 2 asks for the port open above 10,000, we can just do a full port scan from 0 to 65,535 to get underway.

This can be achieved via ```sudo nmap -sS TARGET_IP -T5 -p-```  
#### ⚙️ **Options**  
**sudo** We need to run this command as a privileged user otherwise it will perform a 3-way-handshake and be a TCP Connect Scan -sT.  
**-sS** Tells Nmap to perform a TCP SYN Scan.  
**-T5** Tells Nmap perform the scan at the fastest level of "Insane" Definitely not recommended but works for this case and reduces time.  
**-p-** Tells Nmap to scan all ports.  

  
Once the scan completes, we can see details as to the port number, state and service.  
![Question 1,2,3](./Images/Question%201%2C2%2C3.png)  
We should now have our answer to question one - the port here is the one running the http-proxy service.  

---

## 🛠️ TASK 2: There is an open above 10,000. What is it?  

We can answer this question based on the scan we have already undertaken for question 1.  

The port here is the one listed as having an unknown service.  

---

## 🛠️ TASK 3: How many TCP ports are open?  

Again, we can answer this question based on the scan we have already undertaken for question 1.  

If the scan has correctly returned all open tcp ports you can count them for the answer to this question.  

---

## 🛠️ TASK 4: What is the flag hidden in the HTTP server header?  

As was demonstrated during the Network Security Module we can use Telnet to connect to different protocols to try and grab some basic information. This is what we shall do in this instance.

By using the following command ```telnet TARGET_IP 80``` we attempt to make a telnet connect to port 80 which we have seen above was running a http server.  
Once connected to the HTTP server we can proceed with the below commands to see if we can find the flag in the HTTP header.  
```GET / HTTP/1/1```  
Hit Enter.  
```host: test``` - we need to provide a host: in the GET request we are passing to the server to avoid receiving an error.  
Hit Enter.  

This should now return you the below information, which we can see contains for flag to answer question 4.  
![Question 4](./Images/Question%204.png)  

---

## 🛠️ TASK 5: What is the flag hidden in the SSH server header?  

We will once again utilise Telnet to attempt a connection to the SSH server and see if we can get the flag in the header.  

Connect via ```telnet TARGET_IP 22``` to initiate the telnet connection to the default SSH port of 22, which we also found in our initial scan.  

This should then return you a simple one-line response which contains the flag to answer question 5.  
![Question 5](./Images/Question%205.png)  

---

## 🔄 Altervative methoed for Task 4 and 5.  

We can also get the flags for question 4 and 5 using the default scripts in nmap.  
Using ```nmap -sV -sC TARGET_IP```  
#### ⚙️ **Options**  
**-sV** Tells nmap to perform service version detection which will aid in being able to run scripts.  
**-sC** Tells nmap to run the default scripts against the relevant services.  

This will then return the below, where we can see our flag for question 4 under the red box and our flag for question 5 twice under orange box.  
![Question 4,5 alt](./Images/Qeustion%204%2C5%20alt.png)  

---


## 🛠️ TASK 6: What is the version of the FTP server?  
For this task we can use nmaps service version detection option to find our answer.
This is done via ```nmap -sV TARGET_IP -p PORT```  
#### ⚙️ **Options**  
**-sV** Tells nmap to perform service version detection.  
**-p** Tells nmap to only scan the specified port or ports.  

This should return the below information for the non standard port running the ftp service.  
![Question 6](./Images/Question%206.png)  

---

## 🛠️ TASK 7: What is the flag hidden in one of these two account files and accessible via FTP?  

For this question we are tasked with loging into the ftp service via one of the two provided usernames. The names provided are *eddie* and *quinn*.  For this task we will use Hydra to discover the passwords, as we have two usernames we could run hydra twice against each user, or as in this case we can make a little wordlist with our usernames.  

To make this userlist we can use ```echo -e "eddie\nquinn" > usernames.txt``` this will pop eddie and quinn int a file called usernames.txt with each name on a seperate line.  
#### ⚙️ **Options**  
**-e** Enables the interpretation of backslash escape characters. This tells echo to interperate \n as newline.  

We should end up with the below.  
![Question 7 username list](./Images/Quesiton%207%20username%20list.png)  

Now we can enter this list into Hydra to perform our attack on the ftp server.  
We will use ```hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt ftp://TARGET_IP -s PORT```  
#### ⚙️ **Options**  
**-L** Provides hydra with the location of the username list.  
**-P** Provides hydra with the location of the password list.  
**-s** Sets the port for hydra to use. 

After a minute or so we should get our results with both passwords found as below.  
![Question 7 hydra](./Images/Question%207%20hydra.png)  

Now lets see what we can find when loging into these two users ftp accounts. We can connect to the ftp server via ```ftp TARGET_IP PORT``` once connected we can provide the username in the first instance I tried *eddie* which when using ```ls``` to list the contents returned no results.  
![Question 7 eddie](./Images/Question%207%20eddie.png)  

Not a problem, lets try login in as *quinn* and see if we have any more luck. After running ```ls``` again when logged in as quinn we can see there is a file called ```ftp_flag.txt``` which is what we are after. Lets download this file to our local machine using ```get ftp_flag.txt``` once downloaded we can enter ```quit``` to discconect the ftp session.  
![Question 7 eddie](./Images/Question%207%20quinn.png)  

Now we have the flag file on our system we can use ```cat ftp_flag.txt``` to output the result as below and answer question 7.
---

## 🛠️ TASK 8: What is the flag when you solve the challenge?

---
