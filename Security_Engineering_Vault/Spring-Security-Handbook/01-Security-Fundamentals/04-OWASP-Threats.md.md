---
title: "OWASP Threats and Secure Backend Development"
chapter: 04
series: "Spring Security Handbook"
topic: "OWASP Security Threats"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: "Intermediate to Advanced"
tags:
  - spring-security
  - owasp
  - backend-security
  - api-security
  - authentication
  - authorization
  - secure-coding
  - java
created: 2026-06-26
status: learning

previous:
  - "[[Chapter-03-Identity-Principal-Credentials]]"

next:
  - "[[Chapter-05-Spring-Security-Authentication-Flow]]"

concepts:
  - Broken Access Control
  - Authentication Failures
  - SQL Injection
  - XSS
  - CSRF
  - SSRF
  - Cryptographic Failures
  - Security Logging
  - Secure Design
---

# 04 - OWASP Threats


# 1. Learning Objectives


After completing this chapter, I should understand:


- Why backend applications are attacked
- How attackers think about vulnerabilities
- OWASP security risks relevant to Spring Boot
- How Spring Security helps protect applications
- Why authentication and authorization mistakes create vulnerabilities
- How to design secure REST APIs
- Common production security mistakes
- Security questions asked in interviews


Technology:

- Java 21
- Spring Boot 3.x
- Spring Security 6.x


---

# 2. Why This Concept Exists


Modern backend systems expose:

Mobile Applications

```
    |    |
```

REST APIs

```
    |    |
```

Spring Boot Application

```
    |    |
```

Database

```
Every exposed layer is an attack surface.Security problems happen because:- Input is trusted- Authorization is missing- Sensitive data is exposed- Authentication is weak- Security is ignored during designOWASP helps engineers understand common application risks.---# 3. Simple Definition## OWASP
```

Open Worldwide Application Security Project

```
OWASP provides security guidance about common application vulnerabilities.Backend engineer mindset:
```

What can go wrong in my API?

How can I prevent abuse?

```
---# 4. Mental ModelThink of an API like a bank.Security layers:
```

```
         API          |          |   Authentication          |          |   Authorization          |          |   Business Rules          |          |      Database
```

```
Each layer protects something different.---# 5. Major OWASP Threats# 1. Broken Access Control## ProblemUsers access resources they should not.Example:API:
```

GET /users/100/profile

```
Attacker changes:
```

GET /users/101/profile

```
Backend checks:
```

Is user logged in?

```
But should check:
```

Does this user own user 101?

```
---## Spring Security ProtectionExample:```java@PreAuthorize("hasRole('ADMIN')")public void deleteUser(Long id){}
```

Authorization must happen before business action.

---

# 2. Authentication Failures

## Problem

Weak authentication allows account takeover.

Examples:

- Weak passwords
- Missing MFA
- Poor session handling
- Exposed credentials

Bad:

```
password123
```

Better:

```
BCrypt(password)
```

Spring:

```
PasswordEncoder encoder =        new BCryptPasswordEncoder();
```

---

# 3. SQL Injection

## Problem

User input becomes SQL.

Bad:

```
String sql ="SELECT * FROM users WHERE name='"+ username+ "'";
```

Attacker:

```
' OR 1=1
```

Database query becomes unsafe.

---

## Prevention

Use:

- JPA
- Prepared statements
- Parameter binding

Example:

```
repository.findByUsername(username);
```

---

# 4. Cross Site Scripting (XSS)

## Problem

Attacker injects scripts.

Example:

```
<script>alert('hack')</script>
```

If stored and displayed:

```
Browser executes script
```

Protection:

- Escape output
- Validate input
- Secure frontend rendering

---

# 5. CSRF

## Cross Site Request Forgery

Scenario:

User logged in:

```
Cookie contains session
```

Attacker creates:

```
POST /transfer-money
```

Browser automatically sends cookies.

---

## Protection

Spring Security provides CSRF protection.

For stateless JWT APIs:

```
http.csrf(    csrf -> csrf.disable());
```

Only disable when:

- API is stateless
- Authentication is through Authorization header
- No browser cookie authentication

---

# 6. Security Misconfiguration

Example:

Bad:

```
.requestMatchers("/**").permitAll()
```

Everything is public.

---

Better:

```
.requestMatchers("/admin/**").hasRole("ADMIN")
```

---

# 7. Sensitive Data Exposure

Bad response:

```
{ "username":"john", "password":"secret"}
```

Never expose:

- Passwords
- JWT tokens
- Internal fields
- Secrets

Use DTOs:

```
UserResponseDTO
```

---

# 8. Security Logging and Monitoring Failures

## Why It Exists

Attack can happen without detection.

Example:

```
Attacker   |Failed login attempts   |Application   |No Logs   |Attack Invisible
```

---

## Definition

Security logging failure means:

```
Application does not properly record,detect, or alert security events.
```

---

## Important Events To Log

```
Login successLogin failurePassword changePermission deniedAdmin operationsToken failures
```

Do not log:

```
PasswordJWTAPI KeysSecrets
```

---

# 9. Server Side Request Forgery (SSRF)

## Problem

Backend makes requests using user-controlled input.

Example:

API:

```
GET /fetch-image?url=
```

Attacker:

```
http://internal-server/admin
```

Backend:

```
User | vSpring Boot API | vInternal Service
```

The backend becomes the attacker's proxy.

---

## Protection

Use:

- URL validation
- Domain allowlist
- Block internal IPs
- Network restrictions

Never trust:

```
String url
```

from users.

---

# 10. Cryptographic Failures

## Problem

Sensitive information is not protected correctly.

Sensitive data:

```
PasswordsCredit CardsPersonal DataTokens
```

---

## Password Storage

Bad:

```
password123
```

Correct:

```
password123      |      vBCrypt      |      v$2a$10$....
```

Spring:

```
new BCryptPasswordEncoder();
```

---

Avoid:

```
MD5SHA-1
```

Prefer:

```
BCryptArgon2PBKDF2
```

---

# 11. Insecure Design

## Problem

Security is ignored during system design.

Example:

Requirement:

```
Password reset using email only
```

Problem:

Anyone knowing email can reset account.

---

## Backend Engineer Questions

Before coding:

```
What can go wrong?Who should access this?What happens with malicious input?What happens if permission is missing?
```

---

# Secure Design Example

Bad:

```
POST /transferamount=1000000
```

Missing:

- Limits
- Verification
- Approval

---

Better:

```
Transfer Request       |       vValidation       |       vRisk Check       |       vApproval       |       vTransaction
```

---

# 12. Internal Spring Security Classes

## SecurityFilterChain

Defines security rules.

Example:

```
@BeanSecurityFilterChain security(        HttpSecurity http)
```

---

## AuthenticationFilter

Reads credentials:

- Username/password
- JWT token

---

## AuthenticationManager

Coordinates authentication.

Flow:

```
AuthenticationManager        |        |AuthenticationProvider        |        |UserDetailsService
```

---

## SecurityContextHolder

Stores authenticated user.

Example:

```
SecurityContextHolder.getContext().getAuthentication();
```

---

## AuthorizationManager

Modern Spring Security authorization component.

Checks:

```
Can this user perform this action?
```

---

# 13. Request Lifecycle

Example:

```
HTTP Request      
|     
v
SecurityFilterChain      
|      
v
Authentication Filter      
|      
v
Authentication Object     
| 
v
SecurityContextHolder 
     |      vAuthorizationManager      |      vController      |      vService      |      vDatabase
```

Security happens before business logic.

---

# 14. Production Examples

## E-commerce

API:

```
DELETE /products/10
```

Authentication:

```
User logged in
```

Authorization:

Required:

```
PRODUCT_DELETE
```

Without permission:

```
403 Forbidden
```

---

## Banking

Request:

```
GET /accounts/5001
```

Correct check:

```
Does this account belong to this user?
```

Not only:

```
Is user authenticated?
```

---

# 15. Common Mistakes

## Mistake 1

Authentication only:

```
User logged in = full access
```

Wrong.

Need:

```
Authentication + Authorization
```

---

## Mistake 2

Returning Entity directly.

Problem:

May expose:

- Password hash
- Internal fields

Use DTO.

---

## Mistake 3

Hardcoding admin:

Bad:

```
if(username=="admin")
```

Use:

```
RolesAuthorities
```

---

## Mistake 4

Logging secrets.

Never log:

```
JWTPasswordAPI Keys
```

---

# 16. Interview Questions

## Q1. What is OWASP?

Answer:

Security project providing guidance about application vulnerabilities.

---

## Q2. Authentication vs Authorization?

Authentication:

```
Who are you?
```

Authorization:

```
What can you do?
```

---

## Q3. What is Broken Access Control?

Answer:

Users can access resources they should not.

---

## Q4. What is SSRF?

Answer:

Attacker tricks server into making unauthorized requests.

---

## Q5. How should passwords be stored?

Answer:

Use hashing:

```
BCryptArgon2PBKDF2
```

---

# 17. Scenario Based Questions

## Scenario 1

User changes:

```
/orders/100
```

to:

```
/orders/101
```

and sees another user's order.

Problem:

```
Broken Access Control
```

---

## Scenario 2

Database leaked and passwords are readable.

Problem:

```
Cryptographic Failure
```

Fix:

```
Password Hashing
```

---

## Scenario 3

Backend downloads URL provided by user.

Risk:

```
SSRF
```

Fix:

```
Validate destination
```

---

# 18. Revision Notes

Remember:

```
Authentication=Who are you?
```

```
Authorization=What can you do?
```

Security mindset:

```
Never trust inputNever expose secretsAlways verify permissionsAlways monitor actions
```

Spring Security flow:

```
Request | vAuthentication | vSecurityContextHolder | vAuthorization | vBusiness Logic
```

---

# Previous Chapter

[[Chapter-03-Identity-Principal-Credentials]]

# Next Chapter

[[Chapter-05-Spring-Security-Authentication-Flow]]