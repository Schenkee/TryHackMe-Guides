# Penetration Test Report  
**Engagement:** TryHackMe - Silver Platter  
**Date:** September 2025  
**Tester:** Colin S    

---

## Summary 
A penetration test was conducted against the TryHackMe Silver Platter environment to assess the security of the deployed Silverpeas application and underlying system. The assessment identified three high-impact vulnerabilities: weak password controls that allowed brute-forcing, an insecure direct object reference (IDOR) in the message view functionality, and local privilege escalation that enabled full root compromise. 

Immediate recommendations are to update the Silverpeas application to the latest version, which addresses the IDOR vulnerability. Strong password policies should be enforced, and multi-factor authentication (MFA) enabled where feasible. User privileges should be reviewed to ensure low-privileged users cannot inadvertently access high-privilege actions on the host system.   
 
## Vulnerabilities  

### 1. Improper Access Control
**CVSS v4.0 Base Score:** 7.8 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:N/VA:N/SC:H/SI:H/SA:N  

**Summary:** The login page to Silverpeas accepted weak and easily guessable passwords. This allows attackers to use automated brute-force techniques to gain unauthorised access.  

**Background:** The organisation’s current password policy, which prevents the reuse of passwords that have previously been breached and are publicly available, provides only a minimal additional layer of security.  

As the application still allows weak and easily guessable passwords, attackers can quickly generate a custom wordlist for brute-forcing attempts. Improper Access Control is detailed in CWE-284 and includes multiple sub-categories of access control failures. Applications that fail to enforce strong authentication controls expose themselves to unauthorised access risks.   

**Technical details & Evidence:** A custom wordlist was created to attempt brute-forcing, see the CeWL output below.  
```bash
cewl http://silverplatter.thm > list.txt
```
![Recon - cewl.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20cewl.png)  

Burp Suite Proxy was used to intercept a login request to ```/silverpeas/AuthenticationServlet```
![Recon - burp request.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20burp%20request.png)

This request was then passed to Burp Suite Intruder to perform the brute-force attack. 
```bash
Login=scr1ptkiddy
Password=§§  - location to inject passwords from wordlist into POST request
```
![Recon - Burp setup.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20Burp%20setup.png)  

The custom generated wordlist was then loaded into Burp Suite before the brute-force attack was commenced. This attack located the required password to log in as the ```scr1ptkiddy``` user.  
![Recon - Burp Password.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20Burp%20Password.png)  

Successful application login.  
![Recon - logged in.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20logged%20in.png)  

**Impact:** Attackers can quickly gain unauthorised access to the web application via any located user accounts. Allowing unauthorised access to an internal collaboration application can lead to theft of data or company secrets. This can lead to further privilege escalation or lateral movement within the organisation’s internal network. The attacker’s actions can also impact the availability of the application and the information it contains.  

**Remediation Advice:** The application should enforce strong password policies, implement account lockout after a limited number of failed attempts to limit the ability for brute-force attacks. Where possible, Multi-Factor Authentication should also be enabled to strengthen the application’s authentication mechanisms.      

---

### 2. Insecure Direct Object Reference in Messages  
**CVSS v4.0 Base Score:** 8.3 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:L/VA:N/SC:H/SI:N/SA:N  

**Summary:** The version of Silverpeas in use has a known Insecure Direct Object Reference (IDOR) vulnerability present in the message viewing endpoint. This enables a logged in user to view all messages from all users on the platform.  

**Background:** Silverpeas versions 6.3.1 and prior are vulnerable to CVE-2023-47320. This allows any logged in user to modify the ```ID``` parameter of the message they are viewing. This allows an attacker to read all messages sent between other users, including those sent only to administrators.  

**Technical details & Evidence:** To verify the vulnerability the ```ID``` parameter was adjusted to view different messages.
The logged in user account had a preexisting message at ```ID=5``` 
```url
silverplatter.thm:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=5
```
![Recon - messages.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20messages.png)  

The ```ID``` parameter was then adjusted and allowed reading of another message at ```ID=6```  
```url
silverplatter.thm:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=6
```
![Recon - SSH found.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Recon%20-%20SSH%20found.png)  

**Impact:** The present Insecure Direct Object Reference (IDOR) allows attackers to read all messages sent between other users; including those sent only to administrators. This vulnerability also enables attackers to build automated scripts to recursively download all message content present in the application. This can lead to attackers finding highly sensitive information which may lead to further intrusion into the organisation’s infrastructure.  

**Remediation Advice:** The vendor responsible for the maintenance of Silverpeas has implemented proper access controls over the message’s functionality in version 6.3.2 and onwards. It is strongly recommended that the application be updated immediately to a non-vulnerable version.  

---

### 3. Privilege Escalation
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H  

**Summary:** System misconfigurations allowed a low-privileged user to access log files, extract passwords and use them to log in as a high-privileged user account. This can allow an attacker to fully compromise the targeted host machine.  

**Background:** A discovered low privilege user was part of the ```(adm)``` group on the target host which enables reading of all system log files. Log files often contain sensitive information entered by users into their terminals or captured from other sources. Upon reviewing the logs, clear-text password entries were discovered from a privileged user. Using the located credentials our low privilege user was able to change their logged in session to that of a high privileged user and gain root access. Such misconfigurations allow attackers, even with limited access, to elevate their privileges. This can result in full control and compromise of the entire system.    

**Technical details & Evidence:** Initial enumeration on the logged in low-privileged user indicated their membership of the ```(adm)``` group.  
```bash
tim@ip-10-201-36-198:~$ id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
```
![Flag2 - ID.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20-%20ID.png)  
Research on this group indicated that this user could read the contents of the ```/var/log``` directory on the target host.  
```bash
tim@ip-10-201-36-198:~$ cd /var/log
tim@ip-10-201-36-198:/var/log$ ls
```
![Flag2 - var.log.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20-%20var.log.png)  
Grep was then used to parse all the log files and return any line entries containing the word password.  
```
grep -ir "password"
```
This resulted in multiple returned entries with a clear text password being found.  
```bash
uth.log.2:Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=REDACTED -v postgresql-data:/var/lib/postgresql/data postgres:12.3
```
![Flag2 log output.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20log%20output.png)  

This enabled the low-privileged user to login as the user ```tyler``` which resulted in control over a user with full root access.  
``` bash
tim@ip-10-201-36-198:~$ su tyler
tyler@ip-10-201-36-198:~$ id
uid=1000(tyler) gid=1000(tyler) groups=1000(tyler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
tyler@ip-10-201-36-198:~$ sudo -l
[sudo] password for tyler: 
Matching Defaults entries for tyler on ip-10-201-36-198:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tyler may run the following commands on ip-10-201-36-198:
    (ALL : ALL) ALL
```
![Flag2 - tyler.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20-%20tyler.png)  
![Flag2 - sudo -l.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20-%20sudo%20-l.png)  

As the user ```tyler``` had full sudo privileges to the host machine root user access could be achieved.  
```bash
sudo su
```  
![Flag2 - sudo.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Silver_Platter/Images/Flag2%20-%20sudo.png)  

**Impact:** Attackers can escalate from a limited user to full system access, bypassing security boundaries. This enables complete compromise of the host; enabling long-term persistence, as attackers can create new users for monitoring or continued data extraction. This allows attackers to fully take over the target system.  

**Remediation Advice:** The principle of least privileged access should be applied to all aspects of the host system. All users should be educated regarding the risks of passing clear-text passwords on the command line. Ensure that all privileged users’ credentials are reset and rotated. Organisations should also monitor for misuse of elevated accounts.    

---
## Appendices  
[https://www.silverpeas.org/](https://www.silverpeas.org/)  
[CWE-284](https://cwe.mitre.org/data/definitions/284.html)  
[CVE-2023-47320](https://nvd.nist.gov/vuln/detail/CVE-2023-47320)  
