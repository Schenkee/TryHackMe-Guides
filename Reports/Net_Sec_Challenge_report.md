# Penetration Test Report  
**Engagement:** TryHackMe - Net Sec Challenge  
**Date:** August 2025  
**Tester:** Schenkee  

---

## Summary 
A penetration test was conducted against the TryHackMe NetSec Challenge endpoint to identify open ports, exposed services and vulnerabilities. Multiple services were discovered that allowed sensitive data extraction, account takeover, and intrusion detection system (IDS) bypass.
 Exploiting these services could enable attackers to map the organisation’s infrastructure, compromise accounts through weak SSH credentials, and exfiltrate sensitive data without detection. It is recommended to review the necessity of exposed services, harden configurations, enforce strong password or key-based authentication for SSH, and tune IDS signatures to improve detection capability.  
 
## Vulnerabilities  

### 1. Open Ports Detected  
**CVSS v4 Base Score:** 6.9 (Medium)

**Summary:** Multiple open ports were discovered which enabled service identification. This enables attackers to footprint the company’s infrastructure and services in use.  

**Background:** Open ports allowing service identification can aid attackers in building targeted attacks on known weak or outdated services. Service identification may also enable attackers to perform further targeted reconnaissance to gain further information about a service and possible vulnerabilities present.  

**Technical details & Evidence:** Scanning of open ports was performed with nmap via the following command:  
```bash
sudo nmap -sS 10.10.79.185 -T5 -p-
```
(-T5 used as scan conducted in a lab environment)   

This resulted in the below returned information  
![Questions 1,2,3.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%201%2C2%2C3.png)  
Identified open ports:    
22/tcp – open – SSH  
80/tcp – open – HTTP  
139/tcp – open – NETBIOS-SSN  
445/tcp – open – MICROSOFT-DS  
8080/tcp – open – HTTP-PROXY  
10021/tcp – open – UNKNOWN  

As evident in the above the scan uncovered six open ports and attached services of five ports.  

Further targeted scanning was then performed on the port with an unknown service using the following command   
```bash
sudo nmap -sV 10.10.104.144 -p 10021
```
(note change it IP due to target restart required)

This provided further information about the service running on port 10021  
 ![Question 6.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%206.png)  
Identified Service:  
10021/tcp – open – FTP – VSFTPD 3.0.5  

**Impact:** If left unaddressed, the ability for attackers to identify open ports, services and exact service versions in use could enable detailed network footprinting and research or development of targeted attacks on specific found services. If a service is found to be vulnerable this could allow further access to the company’s network which may lead to data extraction, reputational damage and potential legal implications for failing to safeguard data.  

**Remediation Advice:** It is advised that any ports not required for business operations be closed and unused services be disabled or uninstalled. Update firewall rule ACL’s to ensure only authorized IPs can connect to services, to reduce attack surface. It is also recommended that regular port scanning is conducted, and service reviews performed to ensure no ports are accidentally left open to the internet.

---

### 2. Information Disclosure via HTTP Headers  
**CVSS v4 Base Score:** 6.9 (Medium)

**Summary:** The HTTP service disclosed sensitive information within response headers accessible without authentication. Attackers could use this information to aid further intrusions.   

**Background:** Service banners and verbose headers often contain information not required for normal operations but can assist attackers in reconnaissance. In this case, sending a simple GET request caused the server to reveal sensitive information in its HTTP response.  

**Technical details & Evidence:** A connection to the HTTP service was established using Telnet, and the following request was issued:  

```bash
telnet 10.10.104.144 80
GET / HTTP/1.1
host: test
```
![Question 4](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%204.png)  
The server responded with header information that included sensitive data.  

**Impact:** An attacker could passively harvest sensitive information about the system or application stack. This knowledge may facilitate more targeted exploits, reduce the time needed for successful attacks, and damage the organisation’s reputation.   

**Remediation Advice:** Configure the web server to suppress unnecessary or verbose headers to prevent data leakage. Implement monitoring within the organisation’s SIEM or IDS/IPS for unusual direct connections to web services and perform regular configuration reviews to ensure sensitive data is not exposed.  

---

### 3. Information Disclosure via SSH Banner 
**CVSS v4 Base Score:** 6.9 (Medium)

**Summary:** The SSH service disclosed sensitive information, including version details in the banner, accessible without authentication. Attackers could use this information to aid further intrusions.   

**Background:** Service banners often contain information not required for normal operations but can assist attackers in reconnaissance. In this case, an unauthenticated connection was able to view sensitive information disclosed in the banner.  

**Technical details & Evidence:** A connection to the SSH service was established using Telnet:  
```bash
telnet 10.10.104.144 22
```  
![Question 5](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%205.png)  
Upon connection, sensitive data was presented in the banner and detailed version information. SSH Version – ```SSH-2.0-OpenSSH_8.1p1```  

**Impact:** An attacker could passively harvest sensitive information about the system or application stack. This knowledge may facilitate more targeted exploits, reduce the time needed for successful attacks, and damage the organisation’s reputation.   

**Remediation Advice:** Perform configuration changes to the SSH service to suppress SSH banners. Implement firewall rules to only allow authorised endpoints to connect to the SSH service and ensure regular reviews are conducted to confirm SSH is up to date and not leaking information.   

---

### 4. FTP Weak Authentication & Sensitive File Disclosure 
**CVSS v3.1 Base Score:** 8.8 (High)  

**Summary:** The FTP service used weak passwords, allowing attackers to easily log in and access sensitive files, putting the organisation’s data at risk.  

**Background:** Performing a brute force attack on the FTP service running on port 10021 yielded multiple user account passwords. These passwords were very weak and could be easily brute forced by attackers to gain login access to the FTP service.  

**Technical details & Evidence:** Usernames for two staff members were found via social media reconnaissance. Using hydra, the passwords to the FTP service for the found usernames were discovered.  Usernames identified ```quinn``` and ```eddie``` which were loaded into a wordlist called ```usernames.txt```  

Hydra was used as below to discover the passwords.
```bash
hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt ftp://10.10.79.185 -s 10021
```  
![Question 7 Hydra](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%207%20hydra.png)  
After logging into the FTP service as the user ```quinn``` sensitive files were accessible without restriction. (note change it IP due to target restart required)  
![Question 7 quinn](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/question%207%20quinn.png)  

**Impact:** Weak credentials enable unauthorised access to files and services. Attackers can leverage this access to escalate their privileges or conduct data theft. This could cause significant reputational damage to the organisation and may also result in legal consequences from data breaches.  

**Remediation Advice:** Immediately disable all anonymous accounts and those with weak credentials. Enforce strong password policies requiring sufficient complexity. Configure the FTP service to lock out accounts after multiple failed login attempts to thwart brute force attacks. Implement monitoring of the FTP service within the organisation’s SIEM or IDS/IPS.  

---

### 5.	IDS Bypass via Stealth Scanning  
**CVSS v4 Base Score:** 6.9 (Medium)

**Summary:** The organisation’s IDS could be bypassed using stealth port scans, allowing attackers to perform reconnaissance without detection.  

**Background:** Using stealth scanning techniques, a full port scan was conducted without being detected by the organisation’s IDS solution. This could allow attackers to map the organisation’s infrastructure and prepare further attacks unnoticed.  

**Technical details & Evidence:** Access to the IDS was provided for monitoring during the scan. Using nmap a successful stealth port scan was conducted via the below commands.  
```bash
 nmap -sN 10.10.104.144
```
![Question 8 scan](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%208%20scan.png)  
No alerts were generated during or after the scan, confirming the IDS did not detect the activity.  
![Question 8 flag](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%208%20flag.png)



**Impact:** An IDS system configured with weak detection rules may fail to identify stealth scans, allowing attackers to map networks undetected. If stealth scans are not detected, attackers can enumerate services and identify vulnerabilities without alerting defenders. This increases the risk of successful intrusion, prolonged dwell time, and undetected lateral movement.  

**Remediation Advice:** Implement anomaly-based IDS/IPS with improved detection capabilities. Correlate IDS logs with firewall and system logs to improve detection accuracy. Regularly test IDS rules against common evasion techniques to identify and close detection gaps.

---
