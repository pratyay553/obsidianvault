---
title: "Cryptography Basics for Backend Security"
chapter: 05
series: "Spring Security Handbook"
topic: "Cryptography Basics"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x

level: "Intermediate to Advanced"

tags:
  - spring-security
  - cryptography
  - hashing
  - encryption
  - password-security
  - jwt
  - backend-security
  - java

created: 2026-06-26
status: learning

previous:
  - "[[04-OWASP-Threats]]"

next:
  - "[[06-HTTP-HTTPS-TLS]]"

concepts:
  - Hashing
  - Encryption
  - Encoding
  - Password Storage
  - BCrypt
  - AES
  - RSA
  - JWT
  - Digital Signature
---

# Chapter 05 - Cryptography Basics


# 1. Learning Objectives

After completing this chapter, I should understand:

- Why cryptography exists in backend systems
- Difference between hashing, encryption, and encoding
- How passwords should be stored securely
- Why passwords should never be decrypted
- How Spring Security handles password protection
- How JWT uses cryptographic signatures
- Difference between symmetric and asymmetric encryption
- Common cryptography mistakes in production
- How to answer security interview questions


Technology:

- Java 21
- Spring Boot 3.x
- Spring Security 6.x


---

# 2. Why This Concept Exists


Backend applications handle sensitive data:
Passwords

Payment Information

Personal Data

Authentication Tokens

Business Secrets

```
If this data is leaked:
```

Database Leak

```
    |    v
```

Attacker gets data

```
    |    v
```

User information exposed

```
Cryptography protects information even when systems are compromised.---# 3. Simple DefinitionCryptography is:
```

The practice of protecting information  
using mathematical techniques.

```
Main goals:## Confidentiality
```

Only authorized people can read data

```
## Integrity
```

Data is not modified secretly

```
## Authentication
```

Verify who created the data

```
---# 4. Mental ModelThink of cryptography as different security tools.
```

```
          Data Protection                |    +-----------+-----------+    |           |           | Hashing    Encryption   Signature    |           |           |Password     Secrets      TrustStorage    Protection   Verification
```

```
Different problems require different solutions.---# 5. Hashing## What is Hashing?Hashing converts data into a fixed value.Example:
```

password123

```
    |    v
```

BCrypt

```
    |    v
```

$2a$10$8s8d9....

```
Important:
```

Hashing is one-way.

```
You cannot get:
```

hash

|

v

original password

```
---# Password StorageNever:
```

Database

username:  
john

password:  
password123

```
Correct:
```

Database

username:  
john

password:  
$2a$10$9d83ks....

```
---# Spring Security Example```java@BeanPasswordEncoder passwordEncoder(){    return new BCryptPasswordEncoder();}
```

During registration:

```
Raw Password        |        vPasswordEncoder        |        vStored Hash
```

During login:

```
User Password        |        vBCrypt Compare        |        vMatch / Reject
```

---

# 6. Encryption

## What is Encryption?

Encryption converts readable data into unreadable data.

Example:

```
Original:credit-card-number        |        vEncryption        |        v8HD82HD92HD
```

Unlike hashing:

```
Encryption can be reversedusing a key.
```

---

# Symmetric Encryption

Same key for encryption and decryption.

```
          Secret Key              |              vData  ---->  Encrypted              |              vData  <----  Decrypted
```

Example:

```
AES
```

Used for:

- File encryption
- Database encryption
- Internal communication

---

# Asymmetric Encryption

Uses two keys:

```
Public KeyPrivate Key
```

Flow:

```
Sender   |   |encrypt using public key   vEncrypted Data   |   |decrypt using private key   vReceiver
```

Examples:

```
RSAECC
```

Used for:

- TLS
- JWT signatures
- Digital certificates

---

# 7. Encoding vs Encryption

Common mistake:

Thinking Base64 is encryption.

Example:

```
hello | vaGVsbG8=
```

This is encoding.

Anyone can decode it.

Encoding:

```
Format conversion
```

Encryption:

```
Secret protection
```

---

# 8. Digital Signatures

Purpose:

```
Verify data authenticity
```

Example JWT:

```
Header+Payload+Signature
```

Signature proves:

```
Token was created by trusted serverand was not modified.
```

---

# 9. JWT Cryptography Mental Model

JWT flow:

```
Client   |   vLogin   |   vServer creates JWT   |   vSigns token using secret/private key   |   vClient stores token
```

Request:

```
GET /api/ordersAuthorization:Bearer JWT
```

Server:

```
Verify Signature        |        vRead Claims        |        vCreate Authentication        |        vAllow Request
```

---

# 10. Internal Spring Security Classes

## PasswordEncoder

Interface:

```
org.springframework.security.crypto.password.PasswordEncoder
```

Purpose:

- Encode passwords
- Compare passwords

Methods:

```
encode()matches()
```

Example:

```
passwordEncoder.matches(    rawPassword,    storedHash);
```

---

# BCryptPasswordEncoder

Implementation:

```
new BCryptPasswordEncoder()
```

Why BCrypt?

- Salt included
- Slow by design
- Resistant against brute force

---

# AuthenticationProvider

Uses password verification.

Flow:

```
AuthenticationProvider        |        vPasswordEncoder        |        vUserDetailsService
```

---

# UserDetailsService

Loads user data.

Example:

```
username    |    vDatabase    |    vUserDetails    |    vAuthentication
```

---

# 11. Request Lifecycle

Example:

```
POST /loginusername + password        |        vAuthenticationFilter        |        vAuthenticationManager        |        vAuthenticationProvider        |        vUserDetailsService        |        vPasswordEncoder.matches()        |        vAuthentication Object        |        vSecurityContextHolder
```

---

# 12. Production Examples

## Example 1: User Registration

Request:

```
{ "username":"john", "password":"secret123"}
```

Backend:

```
Password    |    vBCrypt    |    vDatabase
```

Database stores:

```
$2a$10$....
```

---

## Example 2: JWT Authentication

Login:

```
username/password
```

Server:

```
Validate passwordCreate JWTSign JWT
```

Client:

```
Authorization: Bearer token
```

---

## Example 3: Payment Data

Sensitive:

```
Credit Card Number
```

Use:

```
Encryption
```

because system may need original value later.

---

# 13. Common Mistakes

## Mistake 1

Storing plain passwords.

Wrong:

```
password123
```

Correct:

```
BCrypt Hash
```

---

## Mistake 2

Using encryption for passwords.

Wrong:

```
Encrypt password
```

Reason:

Passwords should not be recovered.

Use:

```
Hashing
```

---

## Mistake 3

Using weak algorithms.

Avoid:

```
MD5SHA-1
```

---

## Mistake 4

Logging secrets.

Never:

```
JWT TokenPasswordAPI Key
```

---

## Mistake 5

Using Base64 as security.

Base64:

```
Encoding only
```

Not protection.

---

# 14. Interview Questions

## Q1. Difference between hashing and encryption?

Hashing:

```
One-wayUsed for passwords
```

Encryption:

```
Two-wayUsed when data recovery is required
```

---

## Q2. Why use BCrypt for passwords?

Answer:

Because BCrypt:

- Uses salt
- Is slow
- Reduces brute force attacks

---

## Q3. Can hashed passwords be decrypted?

Answer:

No.

Hashing is one-way.

---

## Q4. Why not store plain passwords?

Answer:

Database leaks expose all user accounts.

---

## Q5. What is JWT signature?

Answer:

A cryptographic mechanism that verifies token integrity and authenticity.

---

# 15. Scenario Based Questions

## Scenario 1

Database leaked.

Passwords are readable.

Problem:

```
Cryptographic Failure
```

Solution:

```
Password hashing
```

---

## Scenario 2

Developer stores:

```
Base64(password)
```

Is it secure?

No.

Base64 is encoding, not encryption.

---

## Scenario 3

Company stores credit card data.

Should they hash it?

No.

Reason:

They may need original data.

Use encryption.

---

# 16. Revision Notes

Remember:

```
Hashing=Protect passwords
```

```
Encryption=Protect data that must be recovered
```

```
Encoding=Change format only
```

Spring Security:

```
User Password        |        vPasswordEncoder        |        vHash Stored        |        vAuthentication
```

JWT:

```
Header+Payload+Signature
```

Security mindset:

```
Never store secrets directlyNever trust encoded dataChoose correct cryptographic tool
```

---

# Previous Chapter

[[04-OWASP-Threats]]

# Next Chapter

[[06-HTTP-HTTPS-TLS]]