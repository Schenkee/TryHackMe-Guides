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

**Impact:** If left unaddressed, the ability for attackers to identify open ports, services and exact service versions in use could enable detailed network footprinting and research or development of targeted attacks on specific found services. If a service is found to be vulnerable this could allow further access to the company’s network which may lead to data extraction, reputational damage and potential legal implications for failing to safeguard data.  

**Remediation Advice:** It is advised that any ports not required for business operations be closed and unused services be disabled or uninstalled. Update firewall rule ACL’s to ensure only authorized IPs can connect to services, to reduce attack surface. It is also recommended that regular port scanning is conducted, and service reviews performed to ensure no ports are accidentally left open to the internet.

---

### 2. Exposed Credentials in Source Code & Robots.txt  
**CVSS v4.0 Base Score:** 

**Summary:**  

**Background:** 

**Technical details & Evidence:** 

**Impact:**   

**Remediation Advice:**  

---

### 3. Command Execution via Web Portal  
**CVSS v4.0 Base Score:**

**Summary:**   

**Background:** 

**Technical details & Evidence:** 

**Impact:** 

**Remediation Advice:** .   

---

### 4. Excessive File Permissions & Misconfigured Sudo  
**CVSS v4.0 Base Score:** 

**Summary:** 

**Background:** 

**Technical details & Evidence:** 

**Impact:** 

**Remediation Advice:** 


---
