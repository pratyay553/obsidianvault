# TLS (Transport Layer Security)

## Definition

**TLS (Transport Layer Security)** is a cryptographic security protocol used to provide **secure communication over a network**.

It protects data by providing:

- Encryption
- Data integrity
- Authentication

TLS is mainly used with:

- HTTPS
- Secure APIs
- Email communication
- Database connections
- Cloud services

> **Simple Definition**
>
> "TLS creates a secure tunnel between client and server so attackers cannot read or modify the communication."

Example:

Without TLS:

```
Client

username=admin
password=12345

        ↓

Attacker can read data
```

With TLS:

```
Client

Encrypted Data

        ↓

Server decrypts it
```

---

# Why it Matters

Modern applications constantly exchange sensitive information.

Examples:

- Login credentials
- Payment details
- Personal information
- API tokens
- Business data

Without TLS, attackers can perform:

---

## Man-in-the-Middle Attack (MITM)

Normal communication:

```
Client

↓

Server
```

Attack:

```
Client

↓

Attacker

↓

Server
```

The attacker can:

- Read data.
- Modify data.
- Steal credentials.

TLS prevents this by encrypting communication.

---

## Data Privacy

TLS ensures:

```
Only sender and receiver

can understand the data.
```

---

## Data Integrity

TLS ensures:

```
Data sent

=

Data received
```

If someone modifies data during transmission, TLS detects it.

---

## Authentication

TLS verifies:

```
Am I really talking to the correct server?
```

Example:

```
bank.com

Certificate

↓

Trusted Certificate Authority
```

---

# How it Works

TLS works using a process called:

```
TLS Handshake
```

The handshake happens before actual encrypted communication.

Complete HTTPS flow:

```
Client

↓

DNS Lookup

↓

TCP Handshake

↓

TLS Handshake

↓

Encrypted HTTP Communication
```

---

# TLS Handshake Steps

## Step 1: Client Hello

Client sends:

- Supported TLS versions.
- Supported encryption algorithms.
- Random value.

Example:

```
Client:

Hello Server,

I support TLS 1.3
```

---

## Step 2: Server Hello

Server responds:

- Selected TLS version.
- Selected cipher suite.
- Digital certificate.
- Public key information.

Example:

```
Server:

Here is my certificate.
```

---

## Step 3: Certificate Verification

Client verifies:

- Certificate is valid.
- Certificate is not expired.
- Domain matches.
- Certificate authority is trusted.

Example:

```
google.com

signed by

Trusted CA
```

---

## Step 4: Key Exchange

Client and server create a shared secret key.

Important:

```
Private key never leaves the server.
```

---

## Step 5: Secure Communication Starts

Now:

```
Client

Encrypted Request

        ↓

Server

Encrypted Response
```

---

# Encryption in TLS

TLS uses a combination of encryption methods.

---

# 1. Asymmetric Encryption

Uses:

```
Public Key

+

Private Key
```

Purpose:

- Authentication.
- Secure key exchange.

Example:

```
Public Key

Anyone can access


Private Key

Only server owns
```

---

# 2. Symmetric Encryption

Uses:

```
One shared secret key
```

Purpose:

Encrypt actual application data.

Why?

Because symmetric encryption is much faster.

---

# 3. Hashing

Used for:

- Integrity checking.
- Detecting modifications.

Example:

```
Message

↓

Hash

↓

Verification
```

---

# TLS Versions

## SSL

Old protocol.

Examples:

```
SSL 2.0

SSL 3.0
```

Deprecated.

---

## TLS 1.0

Old.

No longer recommended.

---

## TLS 1.2

Widely used.

Provides strong security.

---

## TLS 1.3

Modern standard.

Advantages:

- Faster handshake.
- Stronger encryption.
- Removes insecure algorithms.

---

# TLS Certificate

A TLS certificate proves server identity.

Example:

```
Domain:

api.company.com


Public Key:

xxxxx


Issued By:

Certificate Authority
```

Contains:

- Domain name.
- Public key.
- Issuer.
- Expiry date.
- Digital signature.

---

# Certificate Authority (CA)

A trusted organization that issues certificates.

Examples:

- Let's Encrypt
- DigiCert
- GlobalSign

The browser trusts certificates signed by trusted CAs.

---

# TLS vs SSL

| SSL | TLS |
|-|-|
| Older protocol | Modern protocol |
| Less secure | More secure |
| Deprecated | Recommended |
| SSL 3.0 | TLS 1.3 |

Today:

```
HTTPS uses TLS

Not SSL
```

---

# Common TLS Errors

## Certificate Expired

Example:

```
Certificate expired yesterday
```

Solution:

Renew certificate.

---

## Certificate Mismatch

Example:

Website:

```
api.company.com
```

Certificate:

```
test.company.com
```

Browser rejects it.

---

## Self-Signed Certificate

Certificate created internally.

Used for:

- Development.
- Testing.

Usually not trusted publicly.

---

## Wrong Trust Store

Application does not trust the certificate authority.

Common in Java applications.

---

# Common Pitfalls

## ❌ TLS encrypts everything.

Wrong.

TLS encrypts application data.

Network metadata like IP addresses can still be visible.

---

## ❌ HTTPS means no security issues.

Wrong.

TLS protects communication only.

It does not prevent:

- SQL Injection.
- Weak passwords.
- Application bugs.

---

## ❌ Private key is shared with the client.

Wrong.

Private key always remains on the server.

---

## ❌ TLS and HTTPS are the same.

Wrong.

TLS is the security protocol.

HTTPS is:

```
HTTP + TLS
```

---

## ❌ TLS happens before TCP.

Wrong.

The order is:

```
TCP Handshake

↓

TLS Handshake

↓

HTTP Communication
```

---

# Spring Boot / Java Notes

Spring Boot applications commonly use TLS for secure APIs.

Production flow:

```
Client

HTTPS

↓

Load Balancer

↓

Spring Boot Application

↓

Database
```

---

# Enabling TLS in Spring Boot

Example:

`application.properties`

```properties
server.ssl.enabled=true

server.ssl.key-store=classpath:server.p12

server.ssl.key-store-password=password

server.ssl.key-store-type=PKCS12
```

---

# Java Keystore

Java stores certificates and private keys inside:

```
Keystore
```

Common formats:

```
JKS

PKCS12
```

Example:

```
server.p12

contains:

- Certificate
- Private Key
```

---

# TLS in Microservices

Enterprise architecture:

```
Service A

HTTPS + TLS

↓

Service B
```

Benefits:

- Encrypt internal communication.
- Prevent service impersonation.
- Support zero-trust architecture.

Often implemented using:

- mTLS (Mutual TLS)
- Service Mesh

---

# Mutual TLS (mTLS)

Normal TLS:

```
Client verifies Server
```

mTLS:

```
Client verifies Server

+

Server verifies Client
```

Used in:

- Banking systems.
- Enterprise APIs.
- Microservices.

---

# Common Spring Boot TLS Problems

## SSLHandshakeException

Causes:

- Invalid certificate.
- Expired certificate.
- Wrong trust store.

---

## PKIX Path Building Failed

Meaning:

Java does not trust the certificate.

Solution:

Add certificate to trust store.

---

## HTTPS Works in Browser but Not Java Client

Possible reasons:

- Missing CA certificate.
- Different trust store.
- TLS version mismatch.

---

# Quick Revision

- TLS provides secure communication.
- TLS provides:
  - Encryption.
  - Integrity.
  - Authentication.
- HTTPS uses TLS.
- TLS handshake happens after TCP handshake.
- Public key is used during handshake.
- Symmetric key encrypts actual data.
- Private key stays on the server.
- TLS 1.3 is the modern standard.
- mTLS provides two-way authentication.

---

# Interview Questions

### 1. What is TLS?

TLS is a cryptographic protocol that secures communication between client and server.

---

### 2. Why do we need TLS?

To protect data from interception, modification, and impersonation.

---

### 3. Difference between TLS and HTTPS?

TLS is the security protocol.

HTTPS is HTTP communication secured using TLS.

---

### 4. Explain TLS handshake.

Client Hello → Server Hello → Certificate Verification → Key Exchange → Encrypted Communication.

---

### 5. Why does TLS use both asymmetric and symmetric encryption?

Asymmetric encryption securely exchanges keys.

Symmetric encryption provides faster data encryption.

---

### 6. Where is the private key stored?

Only on the server.

---

### 7. What happens before TLS handshake?

TCP three-way handshake happens first.

---

### 8. What is a certificate?

A digital document that proves server identity and contains a public key.

---

### 9. What is mTLS?

Mutual TLS where both client and server authenticate each other.

---

### 10. Why should Java developers understand TLS?

Because production APIs, authentication systems, cloud applications, and microservices depend on secure communication.

---

# References

- [[HTTP]]
- [[HTTPS]]
- [[SSL]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[DNS]]
- [[Firewall]]
- [[Reverse Proxy]]
- [[Load Balancer]]
- [[Certificate]]