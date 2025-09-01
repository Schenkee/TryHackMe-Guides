# 🧠 TryHackMe Room: Net Sec Challenge

**Room URL:** [https://tryhackme.com/room/netsecchallenge](https://tryhackme.com/room/netsecchallenge)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  
**Report:** [Net Sec Challenge Penetration Test Report](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Reports/Net_Sec_Challenge_report.md)

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

This can be achieved via 
```bash
sudo nmap -sS TARGET_IP -T5 -p-
```  
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

By using the following command 
```bash
telnet TARGET_IP 80
``` 
To attempt to make a telnet connect to port 80 which we have seen above was running a http server.  
Once connected to the HTTP server we can proceed with the below commands to see if we can find the flag in the HTTP header.  
```bash
GET / HTTP/1/1
```  
Hit Enter.  
```bash
host: test
```
A host name must be provided in the GET request we are passing to the server to avoid receiving an error.  
Hit Enter.  

This should now return you the below information, which we can see contains for flag to answer question 4.  
![Question 4](./Images/Question%204.png)  

---

## 🛠️ TASK 5: What is the flag hidden in the SSH server header?  

We will once again utilise Telnet to attempt a connection to the SSH server and see if we can get the flag in the header.  

Connect via 
```bash
telnet TARGET_IP 22
```
To initiate the telnet connection to the default SSH port of 22, which we also found in our initial scan.  

This should then return you a simple one-line response which contains the flag to answer question 5.  
![Question 5](./Images/Question%205.png)  

---

## 🔄 Alterative method for Task 4 and 5.  

We can also get the flags for question 4 and 5 using the default scripts in nmap.  
Using 
```bash
nmap -sV -sC TARGET_IP
```  
#### ⚙️ **Options**  
**-sV** Tells nmap to perform service version detection which will aid in being able to run scripts.  
**-sC** Tells nmap to run the default scripts against the relevant services.  

This will then return the below, where we can see our flag for question 4 under the red box and our flag for question 5 twice under orange box.  
![Question 4,5 alt](./Images/Qeustion%204%2C5%20alt.png)  

---


## 🛠️ TASK 6: What is the version of the FTP server?  
For this task we can use the service version detection option in nmap to find our answer.
This is done via 
```bash
nmap -sV TARGET_IP -p PORT
```  
#### ⚙️ **Options**  
**-sV** Tells nmap to perform service version detection.  
**-p** Tells nmap to only scan the specified port or ports.  

This should return the below information for the nonstandard port running the ftp service.  
![Question 6](./Images/Question%206.png)  

---

## 🛠️ TASK 7: What is the flag hidden in one of these two account files and accessible via FTP?  

For this question we are tasked with logging into the ftp service via one of the two provided usernames. The names provided are *eddie* and *quinn*.  For this task we will use Hydra to discover the passwords, as we have two usernames we could run hydra twice against each user, or as in this case we can make a little wordlist with our usernames.  

To make this user list we can use 
```bash
echo -e "eddie\nquinn" > usernames.txt
```
This will pop *eddie* and *quinn* into a file called usernames.txt with each name on a separate line.  
#### ⚙️ **Options**  
**-e** Enables the interpretation of backslash escape characters. This tells echo to interpret \n as newline.  

We should end up with the below.  
![Question 7 username list](./Images/Quesiton%207%20username%20list.png)  

Now we can enter this list into Hydra to perform our attack on the ftp server.  
We will use 
```bash
hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt ftp://TARGET_IP -s PORT
```  
#### ⚙️ **Options**  
**-L** Provides hydra with the location of the username list.  
**-P** Provides hydra with the location of the password list.  
**-s** Sets the port for hydra to use. 

After a minute or so we should get our results with both passwords found as below.  
![Question 7 hydra](./Images/Question%207%20hydra.png)  

Now let’s see what we can find when logging into these two users ftp accounts. We can connect to the ftp server via 
```bash
ftp TARGET_IP PORT
```
Once connected we can provide the username in the first instance and then the relevant password. I tried first *eddie* which when using 
```bash
ls
```
To list the contents returned no results.  
![Question 7 eddie](./Images/question%207%20eddie.png)  

Not a problem, let’s try login in as *quinn* and see if we have any more luck. After running 
```bash
ls
```
Again when logged in as *quinn* we can see there is a file called 
```bash
ftp_flag.txt
```
Which is what we are after. Let’s download this file to our local machine using 
```bash
get ftp_flag.txt
```
Once downloaded we can enter 
```bash
quit
``` 
To disconnect the ftp session.  
![Question 7 quinn](./Images/question%207%20quinn.png)  

Now we have the flag file on our system we can use 
```bash
cat ftp_flag.txt
```
To output the result as below and answer question 7.  
![Question 7 flag](./Images/question%207%20flag.png)  

---

## 🛠️ TASK 8: What is the flag when you solve the challenge?

For the final question we need to open our web browser via ```http://MACHINE_IP:8080``` which will present us with a little challenge to perform a scan without being detected by the IDS.  

I found this to be buggy and had to close and open the page each time between scans if I was detected. Refreshing did not seem to reset the tool. After a couple of different scans, I managed to get the flag using a nmap Null Scan.    

We can perform a nmap Null Scan using 
```bash
sudo nmap -sN TARGET_IP
``` 
As below.   
![Question 8 scan](./Images/Question%208%20scan.png)   

If successful, the web browser should pop up a new message saying *Exercise Complete!* along with the flag.  
![Question 8 scan](./Images/Question%208%20flag.png)  

---

## 🧠 Takeaways

- Practiced identifying open ports with ```nmap```, using various scan types and parameters.   
- Learned to extract banner information from services like SSH and HTTP, via different methods.  
- Used ```Hydra``` to brute-force FTP credentials on a non-standard port.  
- Discovered how stealth scans like ```-sN``` can bypass detection and trigger unique behaviours.  
- Reinforced methodical enumeration skills that were learnt during the Network Security module and how they apply to real-world penetration testing scenarios.  

---

Thank you for following through my guide of the TryHackMe Net Sec Challenge.
