Summary:  
A penetration test was conducted against the TryHackMe NetSec Challenge endpoint to identify open ports, exposed services and vulnerabilities. Multiple services were discovered that allowed sensitive data extraction, account takeover, and intrusion detection system (IDS) bypass.
 Exploiting these services could enable attackers to map the organisation’s infrastructure, compromise accounts through weak SSH credentials, and exfiltrate sensitive data without detection. It is recommended to review the necessity of exposed services, harden configurations, enforce strong password or key-based authentication for SSH, and tune IDS signatures to improve detection capability.  
 
Vulnerabilities:  
1.	**Open Ports Detected**  
**Summary:** Multiple open ports were discovered which enabled service identification. This enables attackers to footprint the company’s infrastructure and services in use.

**Background:** Open ports allowing service identification can aid attackers in building targeted attacks on known weak or outdated services. Service identification may also enable attackers to perform further targeted reconnaissance to gain further information about a service and possible vulnerabilities present.  

**Technical details & Evidence:** Scanning of open ports was performed with nmap via the following command:  
```bash
“sudo nmap -sS 10.10.79.185 -T5 -p-“
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
“sudo namp -sV 10.10.104.144 -p 10021”
```
(note change it IP due to target restart required

This provided further information about the service running on port 10021  
 ![Question 6.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Net_Sec_Challenge/Images/Question%206.png)  
Identified Service:  
10021/tcp – open – FTP – VSFTPD 3.0.5  

**Impact:** If left unaddressed, the ability for attackers to identify open ports, services and exact service versions in use could enable detailed network footprinting and research or development of targeted attacks on specific found services. If a service is found to be vulnerable this could allow further access to the company’s network which may lead to data extraction, reputational damage and potential legal implications for failing to safeguard data.  

**Remediation Advice:** It is advised that any ports not required for business operations be closed and unused services be disabled or uninstalled. Update firewall rule ACL’s to ensure only authorized IPs can connect to services, to reduce attack surface. It is also recommended that regular port scanning is conducted, and service reviews performed to ensure no ports are accidentally left open to the internet.

