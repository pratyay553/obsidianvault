---
title: "HTTP HTTPS TLS for Secure Backend Communication"
chapter: 06
series: "Spring Security Handbook"
topic: "HTTP HTTPS TLS"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: "Intermediate to Advanced"
tags:
  - spring-security
  - http
  - https
  - tls
  - ssl
  - networking
  - backend-security
  - api-security
created: 2026-06-26
status: learning

previous:
  - "[[05-Cryptography-Basics]]"

next:
  - "[[07-Spring-Security-Architecture]]"

concepts:
  - HTTP
  - HTTPS
  - TLS
  - Certificates
  - Encryption
  - Secure Communication
  - HSTS
---

# 06 - HTTP HTTPS TLS


# 1. Learning Objectives


After completing this chapter, I should understand:


- How HTTP communication works
- Why HTTPS exists
- How TLS protects backend communication
- Difference between HTTP and HTTPS
- How certificates establish trust
- How Spring Boot applications enable HTTPS
- How Spring Security depends on secure transport
- Common HTTPS mistakes in production
- How to explain TLS in interviews


Technology:


- Java 21
- Spring Boot 3.x
- Spring Security 6.x


---

# 2. Why This Concept Exists


Backend applications constantly exchange sensitive information:


```

Username

Password

JWT Token

Payment Data

Personal Information

```


Without protection:


```

Client

```
 |

 |
```

Internet

```
 |

 |
```

Server

```


Anyone between client and server could read or modify data.


This is called:


```

Man In The Middle Attack (MITM)

```


HTTPS exists to prevent this.


---

# 3. Simple Definition


## HTTP


HTTP means:


```

HyperText Transfer Protocol

```


It defines how clients and servers communicate.


Example:


```

GET /users/10

```


---

## HTTPS


HTTPS means:


```

HTTP + TLS

```


TLS provides:


- Encryption
- Integrity
- Authentication


Simple:


```

HTTP

=  
Communication

HTTPS

=  
Secure Communication

```


---

# 4. Mental Model


Think about sending a letter.


HTTP:


```

Normal Letter

Anyone can read it

```


HTTPS:


```

Locked Envelope

Verified Receiver

Tamper Detection

```


---

# 5. HTTP Request Response Model


Normal HTTP:


```

Client

|

|

HTTP Request

|

|

Server

|

|

HTTP Response

````


Example:


Request:


```http
GET /api/users

Host: example.com
````

Response:

```json
{
 "name":"John"
}
```

---

# 6. Problem With HTTP

HTTP sends data as plain text.

Example:

```
username=john

password=secret
```

Attacker:

```
Network Traffic

        |

        v

Reads Password
```

---

# 7. HTTPS Architecture

HTTPS adds TLS layer.

```
Client


 |

 |

HTTPS Request


 |

 |

TLS Encryption Layer


 |

 |

Spring Boot Server


 |

 |

Application
```

Data becomes:

```
Plain Data

      |

      v

Encryption

      |

      v

Encrypted Bytes
```

---

# 8. TLS Mental Model

TLS provides three important things.

## 1. Encryption

Protects confidentiality.

Example:

```
Password

   |

   v

Encrypted Data
```

---

## 2. Integrity

Ensures data is not modified.

Example:

```
Original Message


        |

        v


Hash Verification


        |

        v


Same Message
```

---

## 3. Authentication

Confirms server identity.

Example:

```
Am I really talking to my-bank.com?

```

Certificates solve this.

---

# 9. TLS Handshake

Before communication:

```
Client

   |

   |

Hello

   |

   v

Server


   |

   |

Certificate


   |

   |

Verify Certificate


   |

   |

Create Session Key


   |

   |

Encrypted Communication
```

---

# 10. Certificate Concept

Certificate proves:

```
This public key belongs to this server
```

Example:

```
example.com


        |

        |

Certificate


        |

        |

Public Key
```

Issued by:

```
Certificate Authority (CA)
```

Examples:

- Let's Encrypt
    
- DigiCert
    
- GlobalSign
    

---

# 11. Encryption Flow

HTTPS communication:

```
Client


 |

 |  Encrypt using session key

 v


Encrypted Request


 |

 |

Server


 |

 |  Decrypt

 v


Application
```

---

# 12. HTTP Methods and Security

Common methods:

GET:

```
Read data
```

POST:

```
Create data
```

PUT:

```
Update data
```

DELETE:

```
Remove data
```

HTTPS protects all of them.

---

# 13. Internal Spring Boot Classes

## Embedded Web Server

Spring Boot runs using:

```
Tomcat
```

Default:

```
HTTP Connector
```

HTTPS:

```
HTTPS Connector
```

---

# ServerHttpSecurity

Used in reactive applications.

Example:

```java
@Bean
SecurityWebFilterChain security(
ServerHttpSecurity http
){

}
```

---

# HttpSecurity

Main Spring Security configuration API.

Example:

```java
@Bean
SecurityFilterChain security(
HttpSecurity http
){

}
```

Used for:

- Authentication
    
- Authorization
    
- Security rules
    

---

# SecurityFilterChain

Controls incoming requests.

Flow:

```
Request

 |

 v

SecurityFilterChain

 |

 v

Authentication

 |

 v

Authorization

 |

 v

Controller
```

---

# SSL Configuration

Spring Boot:

application.properties:

```properties
server.port=8443

server.ssl.enabled=true

server.ssl.key-store=classpath:keystore.p12

server.ssl.key-store-password=password
```

---

# 14. Request Lifecycle

HTTPS request:

```
Client


 |

 |

TLS Handshake


 |

 |

Encrypted HTTP Request


 |

 |

Embedded Server


 |

 |

Spring Security FilterChain


 |

 |

Authentication


 |

 |

Authorization


 |

 |

Controller


 |

 |

Service


 |

 |

Database
```

Security happens after TLS decrypts the request.

---

# 15. Production Examples

# Example 1: Login API

Without HTTPS:

```
POST /login


username=john

password=secret
```

Attacker can capture password.

---

With HTTPS:

```
POST /login


Encrypted Payload
```

Only client and server understand it.

---

# Example 2: JWT API

Request:

```http
GET /orders

Authorization:
Bearer eyJhbGci...
```

HTTPS protects:

- JWT token
    
- User data
    
- Response data
    

---

# Example 3: Payment System

Sensitive:

```
Credit Card Number
```

Needs:

```
HTTPS

+

Encryption

+

Access Control
```

---

# 16. Common Mistakes

## Mistake 1

Using HTTP in production.

Wrong:

```
http://api.company.com
```

Correct:

```
https://api.company.com
```

---

## Mistake 2

Ignoring certificate validation.

Bad:

```
Accept every certificate
```

Creates MITM risk.

---

## Mistake 3

Thinking HTTPS replaces authorization.

Wrong:

```
HTTPS = Security
```

Correct:

```
HTTPS

+

Authentication

+

Authorization
```

---

## Mistake 4

Logging sensitive HTTPS data.

Never log:

```
Password

JWT Token

Session ID
```

---

## Mistake 5

Disabling TLS verification.

Example:

```
Trust all certificates
```

Dangerous in production.

---

# 17. Interview Questions

## Q1. Difference between HTTP and HTTPS?

HTTP:

```
Unencrypted communication
```

HTTPS:

```
HTTP over TLS
```

---

## Q2. What does TLS provide?

Answer:

- Encryption
    
- Integrity
    
- Authentication
    

---

## Q3. What happens during TLS handshake?

Answer:

Client and server:

- Exchange information
    
- Verify certificate
    
- Create encryption keys
    

---

## Q4. Why do we need certificates?

Answer:

To prove server identity.

---

## Q5. Does HTTPS replace authentication?

Answer:

No.

HTTPS protects communication.

Authentication verifies users.

---

# 18. Scenario Based Questions

## Scenario 1

Developer creates login API:

```
http://company.com/login
```

Problem:

Credentials can be intercepted.

Solution:

Use HTTPS.

---

## Scenario 2

Application uses HTTPS but users access admin APIs.

Problem:

Missing authorization.

Fix:

Add Spring Security authorization rules.

---

## Scenario 3

Certificate expired.

Users see:

```
Connection Not Secure
```

Problem:

TLS trust failure.

Fix:

Renew certificate.

---

# 19. Revision Notes

Remember:

```
HTTP

=

Communication
```

```
HTTPS

=

HTTP + TLS
```

TLS provides:

```
Encryption

+

Integrity

+

Authentication
```

Security layers:

```
HTTPS

      |

      v

Spring Security

      |

      v

Authentication

      |

      v

Authorization

      |

      v

Business Logic
```

Backend mindset:

```
HTTPS protects the connection.

Spring Security protects access.
```

---

# Previous Chapter

[[05-Cryptography-Basics]]

# Next Chapter

[[07-Spring-Security-Architecture]]