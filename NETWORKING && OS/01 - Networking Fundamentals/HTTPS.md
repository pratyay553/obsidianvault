# HTTPS (HyperText Transfer Protocol Secure)

## Definition

**HTTPS (HyperText Transfer Protocol Secure)** is the secure version of HTTP that uses **TLS (Transport Layer Security)** to encrypt communication between a client and a server.

It provides:

- Confidentiality (data privacy)
- Integrity (data is not modified)
- Authentication (verify server identity)

> **Simple Definition**
>
> "HTTPS is HTTP with encryption and security added using TLS."

Example:

HTTP:

```
Client  ─────────────► Server

Plain Data
```

HTTPS:

```
Client  ─────────────► Server

Encrypted Data
```

---

## Why it Matters

Today almost every application requires secure communication.

Examples:

- Banking applications
- E-commerce websites
- Payment systems
- REST APIs
- Cloud applications

Without HTTPS, attackers can perform:

### Data Theft

Example:

```
Username: admin
Password: 12345
```

An attacker on the network can read it.

---

### Man-in-the-Middle Attack (MITM)

Attacker sits between:

```
Client

   ↓

Attacker

   ↓

Server
```

HTTPS prevents this by encrypting communication.

---

### Data Modification

Without HTTPS:

```
Original:

Amount = 100


Attacker changes:


Amount = 10000
```

HTTPS protects data integrity.

---

## How it Works

HTTPS works using:

```
HTTP

+

TLS

=

HTTPS
```

The process happens in two major steps:

```
1. TCP Handshake

2. TLS Handshake

3. HTTP Communication
```

---

# HTTPS Connection Flow

Example:

User opens:

```
https://bank.com
```

Flow:

```
Client

↓

DNS Lookup

↓

Find Server IP

↓

TCP Three-Way Handshake

↓

TLS Handshake

↓

Encrypted HTTP Request

↓

Encrypted HTTP Response
```

---

# TLS Handshake

The TLS handshake creates a secure connection.

## Step 1: Client Hello

Client sends:

- Supported TLS versions
- Supported encryption algorithms
- Random number

Example:

```
Client:

Hello Server,
I support TLS 1.3
```

---

## Step 2: Server Hello

Server responds:

- Selected encryption algorithm
- Server certificate
- Public key

Example:

```
Server:

Here is my certificate.
```

---

## Step 3: Certificate Verification

Client checks:

- Is certificate valid?
- Is it expired?
- Is it issued by a trusted Certificate Authority?

Examples:

- DigiCert
- Let's Encrypt
- GlobalSign

---

## Step 4: Key Exchange

Client and server create a shared secret key.

Important:

The private key never leaves the server.

---

## Step 5: Encrypted Communication Starts

Now:

```
Client

↓

Encrypted HTTP Request

↓

Server

↓

Encrypted HTTP Response
```

---

# Encryption Concepts

HTTPS uses three important security concepts.

---

## 1. Asymmetric Encryption

Uses:

- Public Key
- Private Key

Example:

```
Public Key

Everyone can know


Private Key

Only server knows
```

Used during TLS handshake.

---

## 2. Symmetric Encryption

Uses one shared secret key.

Example:

```
Same Key

Encrypt

↓

Decrypt
```

Used for actual data transfer because it is faster.

---

## 3. Hashing

	Used for integrity checking.

Example:

```
Original Data

↓

Hash

↓

Compare
```

Detects data modification.

---

# Digital Certificate

A certificate proves that a server is genuine.

Example:

```
google.com

Certificate

Issued by

Certificate Authority
```

A certificate contains:

- Domain name
- Public key
- Issuer
- Expiry date
- Digital signature

---

# HTTP vs HTTPS

| HTTP | HTTPS |
|-|-|
| No encryption | Encrypted |
| Port 80 | Port 443 |
| Data visible | Data protected |
| No certificate | Uses certificate |
| Less secure | Secure |

---

# TLS vs SSL

## SSL

Older security protocol.

Examples:

```
SSL 2.0
SSL 3.0
```

Now deprecated.

---

## TLS

Modern replacement.

Versions:

```
TLS 1.2

TLS 1.3
```

Today HTTPS mainly uses TLS.

---

# HTTPS Ports

HTTP:

```
Port 80
```

HTTPS:

```
Port 443
```

---

# Common Pitfalls

## ❌ HTTPS means the website is always trustworthy.

Wrong.

HTTPS only proves encrypted communication.

A malicious website can also have HTTPS.

---

## ❌ HTTPS encrypts everything including IP.

Wrong.

HTTPS encrypts application data.

IP information is still visible for routing.

---

## ❌ The server sends the private key to the client.

Wrong.

Private key always stays on the server.

---

## ❌ HTTPS removes all security problems.

Wrong.

HTTPS protects communication but does not prevent:

- SQL Injection
- Broken authentication
- Application bugs
- Weak passwords

---

## ❌ SSL and TLS are the same.

Wrong.

SSL is the old protocol.

TLS is the modern secure protocol.

---

# Spring Boot / Java Notes

Spring Boot applications commonly run HTTPS in production.

Example:

```
Client

↓

HTTPS Request

↓

Load Balancer

↓

Spring Boot Application
```

---

# Enabling HTTPS in Spring Boot

Example:

`application.properties`

```properties
server.port=443

server.ssl.enabled=true

server.ssl.key-store=classpath:server.p12

server.ssl.key-store-password=password

server.ssl.key-store-type=PKCS12
```

---

# HTTPS Request Flow in Spring Boot

```
Browser

↓

TLS Encryption

↓

Tomcat Server

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Database

↓

Response
```

---

# Production Architecture

Most enterprise systems use TLS termination.

Example:

```
Client

HTTPS

↓

Load Balancer

TLS Ends Here

↓

HTTP/Internal HTTPS

↓

Spring Boot Service
```

Benefits:

- Central certificate management.
- Better performance.
- Easier scaling.

---

# Common Spring Boot HTTPS Problems

## SSLHandshakeException

Possible causes:

- Invalid certificate.
- Expired certificate.
- Wrong trust store.

---

## Certificate Expired

Solution:

Renew certificate.

---

## Mixed Content Error

Example:

HTTPS page calling:

```
http://api.example.com
```

Browser blocks it.

---

## Wrong Keystore Configuration

Check:

- File path.
- Password.
- Certificate format.

---

# Quick Revision

- HTTPS = HTTP + TLS.
- HTTPS provides:
  - Encryption
  - Integrity
  - Authentication
- HTTPS uses port 443.
- TLS handshake happens before HTTP communication.
- Certificates verify server identity.
- Public key encrypts during handshake.
- Symmetric key encrypts actual data.
- Private key never leaves the server.
- Modern HTTPS uses TLS 1.2/1.3.

---

# Interview Questions

### 1. What is HTTPS?

HTTPS is HTTP communication secured using TLS encryption.

---

### 2. Why do we need HTTPS?

To protect data from interception, modification, and impersonation attacks.

---

### 3. What happens when HTTPS connection starts?

Flow:

```
TCP Handshake

↓

TLS Handshake

↓

Encrypted HTTP Communication
```

---

### 4. What is TLS?

TLS is a security protocol that provides encryption and authentication for network communication.

---

### 5. What is a digital certificate?

A certificate proves that a server owns a specific domain and contains its public key.

---

### 6. Difference between HTTP and HTTPS?

HTTP sends data without encryption.

HTTPS encrypts communication using TLS.

---

### 7. Where is the private key stored?

Only on the server.

---

### 8. What port does HTTPS use?

Port 443.

---

### 9. Does HTTPS prevent all attacks?

No. It only protects communication. Application security is still required.

---

### 10. Why should a Spring Boot developer understand HTTPS?

Because production APIs, authentication, payment systems, and cloud applications require secure communication.

---

## References

- [[HTTP]]
- [[TLS]]
- [[SSL]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[DNS]]
- [[Firewall]]
- [[Reverse Proxy]]
- [[Load Balancer]]