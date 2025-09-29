# 🧠 TryHackMe Room: Summit 

**Room URL:** [https://tryhackme.com/room/blue](https://tryhackme.com/room/summit)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all questions in the Summit room which is the penultimate challenge in the Cyber Defence Frameworks module. This room requires the analysis of malware samples and the implementation of defence steps, with each stage being a step up on the Pyramid of Pain. It is recommended to have completed the [Pyramid of Pain](https://tryhackme.com/room/pyramidofpainax) and [MITRE](https://tryhackme.com/room/mitre) rooms.

---

## 🧰 Tools I Used  
- [MITRE ATT&CK](https://attack.mitre.org/)

---

Firstly, let’s start the target machine and give it a few minutes to start all relevant services. This room does not require the use of a personal VM or AttackBox. Once the target machine has started, navigate to the link provided: ```https://LAB_WEB_URL.p.thmlabs.com``` which will have populated with an IP address to match the running machine.  
![Sample 1 - email](./Images/Sample%201%20-%20email.png)  


When the page loads at ```https://LAB_WEB_URL.p.thmlabs.com``` the website looks like an email inbox, which other menus including an EDR simulation called PicoSecure. Read through the first email and at the bottom there is a file called ```sample1.exe``` this is where all our files to analyse will be. Each in a new email that is received when completing a task.  

Also of note is the menu on the left side which can be opened in the top left corner and displays the below items. Which we will need to use to complete tasks.  
![Initial Menu](./Images/Initial%20Menu.png)  

---

## 🛠️ Sample 1:  
To analysis the first sample click on the  ```sample1.exe``` file in the above-mentioned email.  

This will load up the Malware Sandbox where the sample can be analysed.  

Click on ```Submit for Analysis```  
![Sample 1 - Analysis](./Images/Sample%201%20-%20Analysis.png)  

This will pop up some information on the right pane of the website with a bar tracking the analysis completion. Once complete the page will populate various details about ```sample1.exe``` which we will need to use to block the malware and complete task1.  
![Sample 1 -info](./Images/Sample%201%20-%20info.png)

This file does not provide us with much detail other than General file info and Behaviour Analysis. Per the Pyramid of Pain and the side menu the best option to block this sample is to add one of the hashes to the blocklist. Navigate to Manage Hashes in the left-side menu.  

Once on the hashes page paste in one of the 3 provided hashes and click submit hash.  
![Sample - 1 Hash](./Images/Sample%201%20-%20Hash.png)  

Once added the hash will be included on the hash block list in the right-side pane and a message returned confirming we have completed task one.
![Sample 1 - Hash added](./Images/Sample%201%20-%20Hash%20added.png)

Navigate back to the inbox, where a new email with the flag for question one.
![Sample 1 Flag](./Images/Sample%201%20Flag.png)

---

## 🛠️ Sample 2:  
The email received with the flag for question once also contained a new file called ```sample2.exe``` for question two. 
![Sample 2 file](./Images/Sample%202%20%20file%20.png)  

Load ```sample2.exe``` into the Malware Sandbox and select Submit for Analysis.  
![Sample 2 - analysis](./Images/Sample%202%20-%20analysis.png)  

Once the file has been analysed, then below details will be returned. This time there is a little more information when compared to ```sample1.exe```  
![sample 2 - info](./Images/sample%202%20-%20info.png)  

The information returned now also includes Network Activity. Grab the hash value of the file and navigate to the Manage Hashes section and try to add the hash to the blocklist. This will result in an error as the goal is to work up the Pyramid of Pain with each file sample.  
![sample 2 - hash fail](./Images/sample%202%20-%20hash%20fail.png)

After hashes in the pyramid of pain we have IP Addresses which have been provided for ```sample2.exe``` 

As such navigate to the Firewall Manager in the left-hand side menu, where the IP address related to the file will need to be added to the block list. As below  
```general
Type: Egress
Source IP: any
Destination IP: 154.35.10.113 (IP from file analysis)
Action: Deny
```
![sample 2 - firewall rule](./Images/sample%202%20-%20firewall%20rule.png)  

Success the rule has been added, and another email has been received.  
![sample 2 - rule add](./Images/sample%202%20-%20rule%20add.png)  

The new email contains our flag for question 2.  
![sample 2 - flag](./Images/sample%202%20-%20flag.png)  

---

## 🛠️ Sample 3:  

The flag email from question two again contained a new file called ```sample3.exe``` which we will need to load into the Malware Sandbox as previously.  
![sample 3 - file](./Images/sample%203%20-%20file.png)  

Once the file has been analysed the below information is returned which now also contains DNS request details.  
![sample 3 - info](./Images/sample%203%20-%20info.png)  

As per the Pyramid of Pain the next item in the list after Hashes and IP Addresses is Domain Names. As such navigate to the DNS filter in the left-hand menu.

Add the suspicious domain name to the blocklist as below.  
```general
Rule Name: Block emudyn.bresonicz.info (can be any name)
Category: Malware (Not mandatory)
Domain Name: emudyn.bresonicz.info (Suspicious domain from file analysis)
Action: Deny
```
![sample 3 - dns rule](./Images/sample%203%20-%20dns%20rule.png) 

Once the rule has been correctly saved, it will be listed in the right hand blocklist.  
![sample 3 - rule add](./Images/sample%203%20-%20rule%20add.png)

Another email will be received with our flag for question three.  
![sample 3 - flag](./Images/sample%203%20-%20flag.png)  

---

## 🛠️ Sample 4:  

Move onto the next file ```sample4.exe``` and load it into the Malware Sandbox.  
![sample 4 - file](./Images/sample%204%20-flag.png)  

Once the file has been analysed some new information will be returned in the form of Registry Activity.  
![sample 4 - info](./Images/sample%204%20-%20reg%20key.png)  

Now as per the Pyramid of Pain the next item on the list is Network/Host Artifacts which includes registry keys. For this we will need to get more creative in our method to detect the file and start creating Sigma rules, as such navigate to the Sigma Rule Builder.  

On the Sigma Rule Builder there are a few different options. This rule needs to be added to the Registry Modifications via the below navigation  
```Create Sigma Rule -> Sysmon Event Logs -> Registry Modifications```  

The rule that needs to be added is in relation to the item with a PID of ```3806``` which has been attributed to ```sample4.exe```
```general
Registry Key: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection
Registry Name: DisableRealtimeMonitoring
Value: 1
ATT&CK ID: Defense Evasion (TA0005)
```
The ATT&CK ID for this item would be TA0005 as an attacker attempting to disable monitoring on a host is a clear sign of attempting to evade the machines defence. [Defense Evasion](https://attack.mitre.org/tactics/TA0005/)  
![sample 4 - sigma rule](./Images/sample%204%20-%20sigma%20rule.png)  

If successful, the rule will be displayed in the right-hand pane.
![sample 4 - sigma rule add](./Images/sample%204%20-%20signam%20rule%20add.png)  

Another email with the flag for task four will have arrived on successful rule addition.  
![sample 4 - flag](./Images/sample%204%20-%20flag.png)  

---

## 🛠️ Sample 5:  

Now onto question five for which we are not given an ```.exe``` file but a network log file called ```outgoing_connections.log```    
![sample 5 - file](./Images/sample%205%20-%20file.png)  

Upon opening the file, a log of network traffic is provided to try and identify any suspicious patterns. When inspecting this outgoing traffic, we can see repeated small messages being sent to the same IP Address.  
![sample 5 - logs](./Images/sample%205%20-%20logs.png)  

Every 30 minutes a message of 97 bytes is being sent to ```51.102.10.19``` which looks very odd and possibly could be a device reaching out to a command and control(C2) server. This would also correlate with our progress up the Pyramid of Pain as the next level would be Tools which includes Command and Control Servers.  

Navigate back to the Sigma rule builder and navigate to the below entry.  
```Create Sigma Rule -> Sysmon Event Logs -> Network Connections```  
Add the below rule to detect this network traffic.  
```general
Remote IP: any
Remote Port: any
Size(bytes): 97
Frequency(seconds): 1800(30 minutes)
ATT&CK Id: Command and Control (TA0011)
```
As mentioned above the ATT&CK ID is TA0011. [Command and Control](https://attack.mitre.org/tactics/TA0011/)  
![sample 5 - sigma rule](./Images/sample%205-%20sigma%20rule.png)

Rule Validation if correctly added. 
![sample 5 - rule add](./Images/sample%205%20-%20rule%20add.png)  

Another email will have arrived in the inbox with the flag and our final challenge.  
![sample 5 - flag](./Images/sample%205%20-%20flag.png)

---

## 🛠️ Final sample:  

Onto the final challenge in which a file called ```commands.log``` which is a log of system commands used by the attacker.  
![sample 6 - file](./Images/sample%206%20-%20file.png)  

Open the file and review the below commands. This ties in perfectly with the last and top item on the Pyramid of Pain Tactics, Techniques, Procedures (TTPs). This log file displays the attackers TTP in extracting data from the target system.
![sample 6 - log](./Images/sample%206%20-%20log.png)

Navigate back to the Sigma Rule builder and proceed to add a new rule into the below location.  
```Create Sigma Rule -> Sysmon Event Logs -> File Creation and Modification```  
Add the below rule to detect this activity.  
```general
File Path: %temp%
File Name: exfiltr8.log
ATT&CK ID: Exfiltration (TA0010)
```
The ATT&CK ID for this item will be TA0010. [Exfiltration](https://attack.mitre.org/tactics/TA0010/)  
![sample 6 - sigma rule](./Images/sample%206%20-%20sigma%20rule.png)  

Once the rule has been added the below rule validation will be presented and an email received with our final flag.  
![sample 6 - rule add](./Images/sample%206%20-%20rule%20add.png)  

The final flag.  
![sample 6 - flag](./Images/sample%206%20-%20flag.png)  

---


## 🧠 Takeaways  

- During this room the Pyramid of Pain framework was reinforced with practical examples.
- Practiced in adding Hashes, IPs and Domain Names to our block lists.
- Created Sigma rules to detect three further files.
- Used the MITRE ATT&CK framework to correlate attack stages.


---

Thank you for following through my guide of the TryHackMe Summit.  
