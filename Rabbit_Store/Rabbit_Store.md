# 🧠 TryHackMe Room: Rabbit Store

**Room URL:** [https://tryhackme.com/room/rabbitstore](https://tryhackme.com/room/rabbitstore)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Challenge Task Overview  

**Objective:**  This guide walks through solving the Rabbit Store room. This challenge involves multiple rounds of web enumeration. The exploitation of three different web vulnerability to gain initial access to the host system. Once initial access has been gained privilege escalation is achieved via enumerating a RabbitMQ instance.


---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- dirsearch  
- ffuf  
- Caido  
- rabbitmqctl  

---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. The room author reccomands to use a personal machine connected via VPN to tackle this challange, rather than the AttackBox. For this room I would also highly reccomand adding `rabbitstore.thm` to your `/etc/hosts` file as there will be multiple further domains that need to be accessed via the same IP address later on in the room.

---

## 🛠️ Recon & Enumeration 



---

## 🛠️ Initial Access

### 🔑 Mass Assignment Vulnerability

---


### 🌐 Server-Side Request Forgery (SSRF) Vulnerability

---

### 🧩 Server-Side Template Injection (SSTI) Vulnerability


---

## 🛠️ Flag 1 



---

## 🛠️ Flag 2


---

## 🧠 Takeaways  


---

Thank you for following through my guide of the TryHackMe Rabbit Store room.
