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

Via a web browser, a registration request was submitted to `http://storage.cloudsite.thm/api/register` this request was intercepted via Caido and the value, `"subscription":"active"` added to the body. 
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
Enforce an allowlist of assignable fields server-side. Validate and normalise inputs and ensure default account attributes are set server-side only, do not accept client-controlled values for authorisation-sensitive attributes. Perform a detailed review of all API endpoints to ensure only intended fields can be modified by clients.   

---

### 2. Server-Side Request Forgery  
**CVSS v4.0 Base Score:** 5.3 (Medium)  
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:L/VI:N/VA:N/SC:L/SI:N/SA:N  

**CVSS Justification:**  
The upload-from-URL featured allowed the server to fetch arbitrary URLs, including `localhost` on the same host. Attack complexity is low, requires access to the subscription only dashboard and the impact include disclosure of limited internal resources. As such a lower-than-expected CVSS v4.0 score was modelled, in practice Server-Side Request Forgery is a high-risk vulnerability, and remediation should be prioritised.  

**Summary:**

**Background:** 

**Technical details & Evidence:** 

**Impact:** 

**Remediation Advice:** 

---

### 3. Server-Side Template Injection  
**CVSS v4.0 Base Score:** 


**Summary:** 

**Background:** 

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
