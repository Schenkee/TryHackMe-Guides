# 🧠 TryHackMe Room: Summit 

**Room URL:** [https://tryhackme.com/room/blue](https://tryhackme.com/room/summit)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview

**Objective:**   
This guide will cover the steps required to gain the answer to all questions in the Summit room which is the penultimate challange in the Cyber Defense Frameworks module. This room requires the analysis of malware samples and the implemenataion of defense steps, which each stage being a step up on the Pyramid of Pain. It is reccomended to have completed the [Pyramid of Pain](https://tryhackme.com/room/pyramidofpainax) and [MITRE](https://tryhackme.com/room/mitre) rooms.

---

## 🧰 Tools I Used  
- [MITRE ATT&CK](https://attack.mitre.org/)

---

Firstly, let’s start the target machine and give it a few minutes to start all relevant services. This room does not require the use of a personal VM or AttackBox. Once the target machine has started navigate to the link provided: ```https://LAB_WEB_URL.p.thmlabs.com``` which will have populated with an IP address to match the running machine.  
![Sample 1 - email](./Images/Sample%201%20-%20email.png)  


When the page loads at ```https://LAB_WEB_URL.p.thmlabs.com``` the website looks like an email inbox, which other menus depicting an EDR called PicoSecure. Read through the first email and at the bottom there is a file called ```sample1.exe``` this is where all our files to analyis will be. Each in a new email that is recived when completing a task.  

Also of note is the menu on the left side which can be opened in the top left corner and displays the below items. Which we will need to use to complete tasks.  
![Initial Menu](./Images/Initial%20Menu.png)  

---

## 🛠️ Sample 1:  
To analyis the first sample click on the  ```sample1.exe``` file in the above mentioned email.  

This will load up the Malware Sandbox where the sample can be analyised.  

Click on ```Submit for Analysis```  
![Sample 1 - Analysis](./Images/Sample%201%20-%20Analysis.png)  

This will pop up some information on the right pane of the website with a bar tracking the analysis completion. Once complete the page will populate variose details about ```sample1.exe``` which we will need to use to block the malware and complete task1.  
![Sample 1 -info](./Images/Sample%201%20-%20info.png)

This file does not provide us with much detail other than General file info and Behavior Analysis. Per the Pyramid of Pain and the side menu the best option to block this sample is to add one of the hashes to the blocklist. Navigate to Manage Hashes in the leftside menu.  

Once on the hashes page paste in one of the 3 provided hashes and click submit hash.  
![Sample - 1 Hash](./Images/Sample%201%20-%20Hash.png)  

Once added the hash will be included on the hash block list in the right side pane and a message returned confirming we have completed task 1.
![Sample 1 - Hash added](./Images/Sample%201%20-%20Hash%20added.png)

Navigate back to the inbox, where a new email with have arrived with the next sample to analyis and our flag for question 1.
![Sample 1 Flag](./Images/Sample%201%20Flag.png)

---

## 🛠️ Sample 2:  


---

## 🛠️ Sample 3:  



---

## 🛠️ Sample 4:  



---

## 🛠️ Sample 5:  


---

## 🛠️ Final sample:  


---


## 🧠 Takeaways  



---

Thank you for following through my guide of the TryHackMe Summit.  
