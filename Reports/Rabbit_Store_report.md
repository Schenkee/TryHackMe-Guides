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
When registering as a new user on the website, it was discovered that the JSON parameters could be modified and submitted for successful registration. This then allowed the user to log in and access the restricted subscription only portion of the website. This allows attackers to gain access to hidden functionality or information only intended for paying clients, attackers could publicly leak this information.   

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
User input passed to the Jinja2 template lead to remote code execution. The attack was network-accessible, low complexity, and required prior authenticated access to the subscription-only dashboard. This resulted in high confidentiality, integrity and availability impact.  

**Summary:**   
The `/api/fetch_messages_from_chatbot` POST endpoint which is still under development reflected user supplied inputs into the template. A crafted payload could be supplied and executed to run system commands; this succesfully allowed access to the target machine has a low privilge user.  

**Background:**   
Template rending logic included unsanitised user inputs. When supplying a user input to the `/api/fetch_messages_from_chatbot` endpoint via a HTTP POST request the output was reflected into the Jinja2 template. Using a sequence of special characters commonly used in template expressions, such as `${{<%[%'"}}%\` an exeption was raised in the server response providing details as to the template engine in use and the confirmation of a existing template-injection vulnerability. A specially crafted message was then sent to the endpoint to inititate a reverse shell connection which was executed succesfully and provided shell access to the target as the user `azrael`.  

**Technical details & Evidence:** 


**Impact:**   


**Remediation Advice:**   

---

### 4. Privilege Escalation via RabbitMQ  
**CVSS v4.0 Base Score:** 


**Summary:** 

**Background:** 

**Technical details & Evidence:** 

**Impact:** 

**Remediation Advice:** 

---
## Appendices  
[OWASP Top 10](https://owasp.org/www-project-top-ten/)  
[Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)  
[CVSS 4.0](https://www.first.org/cvss/calculator/4-0)  
