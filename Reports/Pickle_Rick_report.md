# Penetration Test Report  
**Engagement:** TryHackMe - Pickle Rick  
**Date:** September 2025  
**Tester:** Colin S  

---

## Summary 
A penetration test was conducted against the TryHackMe Pickle Rick environment to assess its security posture. The assessment identified several critical weaknesses, including exposed credentials, insecure web application functionality, and misconfigured access permissions. These weaknesses enabled full compromise of the system, including administrator-level access and extraction of sensitive files.

To reduce risk, it is recommended to strengthen credential management practices, enforce least-privilege access controls, and harden web application security configurations.
 
## Vulnerabilities  

### 1. Open Ports Detected  
**CVSS v4.0 Base Score:** 6.9 (Medium)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:N/VA:N/SC:N/SI:N/SA:N  

**Summary:** Multiple open ports were discovered which enabled service identification. This enables attackers to footprint the company’s infrastructure and services in use.  

**Background:** Open ports allowing service identification can aid attackers in building targeted attacks on known weak or outdated services. Service identification may also enable attackers to perform further targeted reconnaissance to gain further information about a service and possible vulnerabilities present.  

**Technical details & Evidence:** Scanning of open ports was performed with nmap via the following command:  
```bash
sudo nmap -sS 10.201.84.79 -T5 -o ports.txt
```
(-T5 used as scan conducted in a lab environment)   

This resulted in the below returned information  
![Recon - Nmap scan 1.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Recon%20-%20Nmap%20scan%201.png)  
Identified open ports:    
22/tcp – open – SSH  
80/tcp – open – HTTP  

As evident in the above the scan uncovered two open ports and their attached services.  

Further targeted scanning was then performed on the two identified open ports.  
```bash
sudo nmap -sS -sV -sC 10.201.84.79 -p 22,80
```

This provided further information about the services running on both ports.
 ![Recond - Nmap scan 2](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Recon%20-%20Nmap%20scan%202.png)    

**Impact:** If left unaddressed, the ability for attackers to identify open ports, services and exact service versions in use could enable detailed network footprinting. If a service is found to be vulnerable this could allow further access to the company’s network which may lead to data extraction, reputational damage and potential legal implications for failing to safeguard data.  

**Remediation Advice:** It is advised that any ports not required for business operations be closed and unused services be disabled or uninstalled. Update firewall rule ACLs to ensure only authorized IPs can connect to services, to reduce attack surface. It is also recommended that regular port scanning is conducted, and service reviews performed to ensure no ports are accidentally left open to the internet.

---

### 2. Exposed Credentials in Source Code & Robots.txt  
**CVSS v4.0 Base Score:** 8.8 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:L/VA:N/SC:N/SI:N/SA:N  

**Summary:**  Credentials were exposed in publicly accessible files and HTML source code, enabling unauthorised login to the web portal.  

**Background:** Hardcoding sensitive information in client-accessible files is a critical security misconfiguration. Attackers often review client accessible files and source code as part of initial reconnaissance.  

**Technical details & Evidence:** 
- ```http://10.201.84.79/robots.txt``` exposed the string ```Wubbalubbadubdub```
  
![Recon - Robots.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Recond%20-%20Robots.png)  
 
- HTML source code exposed the username ```R1ckRul3s```
  
![Recon - Sourcecode.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Recon%20-Sourcecode.png)

**Impact:** Exposed credentials allowed direct authentication to the application login page. This granted full access to the internal command execution panel. This would enable attackers to access file directories on the web server and possibly perform data exfiltration, website defacing. This could lead to website instability, exfiltration of sensitive data, possibly leading to legal consequences.  

**Remediation Advice:**  Remove all sensitive data from client-facing files. Perform rotation of all exposed credentials immediately. Implement secure credential storage and ensure no production secrets are hardcoded in web pages or client-facing text files.  

---

### 3. Command Execution via Web Portal  
**CVSS v4.0 Base Score:** 8.7 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N

**Summary:** The authenticated web portal provided a command panel that allowed arbitrary system command execution.   

**Background:** Arbitrary commands could be executed in the web portal. This allowed the bypassing of intended restrictions by using alternative tools to extract file contents from the target system.  

**Technical details & Evidence:** 
- After login, the portal allowed arbitrary system commands.
- Sensitive files were discovered and accessed, including:  
- ```/home/ubuntu/Sup3rS3cretPickl3Ingred.txt```
-   The above secret was downloaded to the attacking machine by starting a HTTP server on the target.
  ```bash
  python3 -m http.server 5050   # started a file server
  wget http://10.201.84.79:5050/Sup3rS3cretPickl3Ingred.txt  
  cat Sup3rS3cretPickl3Ingred.txt   # ran on attacking machine post file download
  ```
![Flag1 - server.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Flag1%20-%20Server.png)
![Flag1 - wget.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/flag1%20-%20wget.png)
![Flag1 - file.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Flag1%20-%20file.png)
  
- ```/home/rick/second ingredients```  
  The above secret is displayed using the ```strings``` command.
  ```bash
  strings "/home/rick/second ingredients"
  ```
![Flag2 - file.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Flag2%20-%20file.png)
  
- ```/root/3rd.txt (accessed with sudo)```  
  The above secret was displayed using the ```less``` command.
  ```bash
  sudo less /root/3rd.txt
  ```
![Flag3 - file.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Flag3%20-%20file.png)  

**Impact:** Arbitrary command execution provides attackers full control of the system, enabling privilege escalation, sensitive file access, lateral movement to other targets in the network and data exfiltration. Allowing attackers to extract data from the web server backend may lead to other users’ data being exposed or company secrets/private details stored on the server. This could expose the organisation to regulatory or legal consequences.  

**Remediation Advice:** Remove or restrict the command panel functionality. If the functionality must remain then it is critical that strict server-side input validation and sanitisation is implemented. Restrict portal access to trusted IP ranges if functionality cannot be removed. Apply the principle of least privilege to web application accounts, to reduce the attack scope of compromised. Monitor for suspicious system commands via SIEM/IDS integration.  

---

### 4. Excessive File Permissions & Misconfigured Sudo  
**CVSS v4.0 Base Score:** 8.5 (High)
CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  

**Summary:** Files containing sensitive data were world-readable, and the authenticated user had unrestricted ```sudo``` access on the system. This enables privilege escalation to the root user.  

**Background:** Misconfigured file permissions and unrestricted sudo access represents a major security risk. These misconfigurations allow attacks even with limited access to elevate their privileges to gain full ```root``` control and compromise the entire system.   

**Technical details & Evidence:** 
- World-readable files discovered in /home directories.  
- Running ```sudo -l``` confirmed full sudo privileges without requiring a password.  
- This enabled direct access to ```/root/3rd.txt```.

```bash
sudo -l
sudo less /root/3rd.txt
```
![Flag3 - file.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Pickle_Rick/Images/Flag3%20-%20file.png)  

**Impact:** Attackers can escalate from a limited user to full system access, bypassing security boundaries. This enables complete compromise of the host; this can also enable long-term persistence as attackers can create new users to use for monitoring of the system or continued data extraction. This allows attacks to fully take over the system and served web content, possibly inserting malicious content into the website to steal users’ personal data.   

**Remediation Advice:** Apply least-privilege principles to file permissions. Restrict sudo privileges to only necessary administrative users. Require multi-factor authentication for privileged accounts. Regularly audit file permissions and sudo configurations.  

---
