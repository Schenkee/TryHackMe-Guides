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
When registering as a new user on the website, it was discovered that the JSON parameters could be modified and submitted for successful registration. This then allowed the user to login and access the restricted subscription only portion of the website.  

**Background:** 

**Technical details & Evidence:**  

**Impact:** 

**Remediation Advice:** 

---

### 2. Server-Side Request Forgery  
**CVSS v4.0 Base Score:**


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
