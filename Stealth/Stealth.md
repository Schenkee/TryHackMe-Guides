# 🧠 TryHackMe Room: Stealth

**Room URL:** [https://tryhackme.com/room/stealth](https://tryhackme.com/room/stealth)  
**Author:** Schenkee  
**Profile:** [https://tryhackme.com/p/schenkee](https://tryhackme.com/p/schenkee)  

---

## 🧩 Overview  

**Objective:**  
This guide walks through solving the Stealth room. This challenge involves initial exploitation of a web upload vulnerability. Once initial access has been gained privilege escalation is achieved via exploiting the SeImpersonatePrivilege to gain full administrator access.
This rooms initial access portion challenged me and I had to get some guidance to understand the correct approach, it looked like a simple upload vulnerability at first but needed some clever tricks to take full advantage of. Once initial access had been gained I had wanted to create a sliver session with GodPotato to escalate my privileges but ran into issues with my Sliver `NT AUTHORITY\SYSTEM` shell constantly breaking, as such I reverted to a different tactic. I have kept the Sliver details in the guide to show my thought process.

---

## 🧰 Tools I Used  
- RustScan  
- nmap  
- dirsearch  
- P0wny-shell - [https://github.com/flozz/p0wny-shell/tree/master](https://github.com/flozz/p0wny-shell/tree/master)
- GodPotato - [https://github.com/BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato)
- Sliver
- CyberChef
- FreeRDP


---

Firstly, let’s start our target machine and give it a few minutes to start all relevant services. We can also either start the AttackBox or as in my case connect our own machine to the TryHackMe VPN.  If needed, you can add ```stealth.thm``` along with the assigned IP to your ```/etc/hosts``` file which can make life easier than remembering an IP all the time.  

---

## 🛠️ Recon & Enumeration 



---

## 🛠️ Initial Access



---

## 🛠️ Privilege Escalation

### 🛰️ Sliver Method


---

### 🖥️ User Creation and RDP Method

---

## 🛠️ Flags


---

## 🧠 Takeaways  


---

Thank you for following through my guide of the TryHackMe Stealth room.
