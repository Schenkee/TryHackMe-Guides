# 🧠 TryHackMe Room: File Inclusion (Challenge Task Only)

**Room URL:** [https://tryhackme.com/room/fileinc](https://tryhackme.com/room/fileinc)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)

---

## 🧩 Challenge Task Overview

**Objective:**  
This guide will cover the steps required to gain the flags to complete the Challenge task of the room. We will cover 2 different methods to gain RCE access to /playground.php for question 4. Please read through the initial tasks to gain an understanding of Local File Inclusion (LFI), Remote File Inclusion (RFI), and directory traversal Vulnerability.

---

## 🧰 Tools I Used
- Kali Linux
- Metasploit
- Curl
- Python3 http server
- Netcat
- PHP Reverse Shell By Pentestmonkey which can be found here: [https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

---

## 🛠️ TASK 1: Capture Flag1 at /etc/flag1
Navigate to the first task at **http://MACHINE_IP/challenges/chall1.php** Once the page loads we are greeted with a important message to aid us in capturing the flag.  
The page reads "The Input form is broken! You need to send **'POST'** request with **'file'** parameter! This message gives us a great starting point as we know we need to send a POST request. This can be done in a couple of different manners such as BurpSuite or Curl.

In this instance I will use Curl to generate the POST request and receive the flag. Open up your terminal either on your VM or in the AttackBox using CTRL+ALT+T and input the below command 

```curl http://MACHINE_IP/challenges/chall1.php -X POST -d "file=../../../../etc/flag1" ```  
### ⚙️ **Options**  
-X is used with curl to change the request method from the default of GET to something else such as POST in this case. -X is not techincally required as a Curl request passed with -d should automtically become a **POST** request.     
-d is used to set the data to send with the POST request in this case file=../../../../etc/flag1  
  
You should end up with something like the below  
![lab1 curl](./Images/lab1_curl.png)  
  
This will then return a raw response from the web server which will contain our flag as below.
![lab1_flag](./Images/lab1_flag.png)


---

## 🛠️ TASK 2: Capture Flag2 at /etc/flag2  
Now lets move onto the second challenge at **http://MACHINE_IP/challenges/chall2.php** This time once the page loads we are greeted with a message asking us to refresh the page.  
Lets refresh as requested and then we are given some more important information as to the likely path for capture the flag. The page reads as below  
![lab2_start](./Images/lab2_start.png)  
  
Lets inspect the page via the browsers Developer Tools to see if we can work out what’s going on here and how we can change ourselves to be an **admin** on investigation of the cookie in the **Network** tab we can see that the cookie has a parameter of **THM=Guest** which we might be able to edit.  
![lab2_cookie view](./Images/lab2_cookie%20view.png)  

Lets see if we can edit this cookie parameter to make ourselves an admin. Navigate to the **Storage** tab to modify the cookie. Lets adjust the cookie value from Guest to **admin** 
![lab2_cookie](./Images/lab2_cookie.png)

Once we have adjusted the cookie refresh the page and if successful we should revise a new message as below.  
![lab2_cookie_success](./Images/lab2_cookie_success.png)  

Now we might think, that once we are the admin we can simply request the flag via the address bar as in previous tasks. But in this instance that will not yield any results, but what we did learn via the above steps is that we are able to tamper with the cookie.  
So lets use that knowledge and see if we can input the flag2 path as the cookie value. Navigate back to your browsers **storage** tab in the Developer Tools and adjust the cookie value to ```../../../../etc/flag2%00```  
![lab2_cookie_path](./Images/lab2_cookie_path.png)  

Refresh the page and we can see that we have been able to successfully view the contents of flag2 by tampering with the cookie value.  
![lab2_flag](./Images/lab2_flag.png)


---

## 🛠️ TASK 3: Capture Flag2 at /etc/flag3  
Task 3 was arguably the one I spent the most time working through and via a process of elimination/trial and error I managed to get the flag. Once we load up the challange at **http://MACHINE_IP/challenges/chall3.php** we will not see anything of not such as with the two previouse tasks. So as a start lets just pop in our file path and see what is returned.  

Entering ```../../../../etc/flag3``` into the input form will not return our flag. But the result does contain some useful information for us. Below is the returned information.  
![lab3_start](./Images/lab3_start.png)  

We can see from this image that there appears to be some input sanitisation occuring which has modifed our input from ```../../../../etc/flag3``` to simply ```etcflag.php``` This indicates that the web server is removing dots and slashes and also appending the input with .php indicating the developer has specified the file type to pass to the include function. We can bypass this last part via a null byte.

After some trial and error I was still not able to find any success with the website and decided to try using Curl again ro make the request.  
I first attempted a Curl request similar to what we used in Task 1  ```curl http://MACHINE_IP/challenges/chall1.php -X POST -d "file=../../../../etc/flag3%00" ```     
Which returned an error messaged indication that the output needed to be passed to a fail, as such I modified my input to output the returned data into a file using the below  

```curl http://MACHINE_IP/challenges/chall1.php -X POST -d "file=../../../../etc/flag3%00" -o file.txt```  
-X is used with curl to change the request method from the default of GET to something else such as POST in this case. -X is not techincally required as a Curl request passed with -d should automtically become a **POST** request. 
-d is used to set the data to send with the POST request in this case file=../../../../etc/flag3%00  
-o is used to output the returned result into a file of the specified name and type.
