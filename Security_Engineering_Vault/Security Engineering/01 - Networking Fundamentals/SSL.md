# SSL (Secure Sockets Layer)

## Definition

**SSL (Secure Sockets Layer)** is an older cryptographic security protocol used to secure communication between a client and a server by encrypting data.

SSL was designed to provide:

- Encryption
- Authentication
- Data integrity

However, SSL is now **deprecated** and has been replaced by **TLS (Transport Layer Security)**.

> **Simple Definition**
>
> "SSL was the original technology used to secure Internet communication. Modern HTTPS uses TLS instead of SSL."

Example:

Before SSL:

```
Client

↓

Plain Text Data

↓

Server
```

With SSL:

```
Client

↓

Encrypted Data

↓

Server
```

---

# Why it Matters

Understanding SSL is important because:

- Many old systems still mention SSL.
- Interviewers often ask SSL vs TLS.
- Certificates are still commonly called "SSL certificates".
- Production systems use TLS, but the terminology remains.

Examples where SSL/TLS is used:

- HTTPS websites.
- Banking applications.
- Payment systems.
- Secure APIs.
- Cloud services.

---

# Problems Without SSL/TLS

## Data Exposure

Without encryption:

```
Username: john

Password: 123456
```

An attacker can read the data.

---

## Man-in-the-Middle Attack

Communication:

```
Client

↓

Server
```

Attacker inserts themselves:

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

SSL was created to prevent this.

---

# How it Works

SSL works using:

```
Certificate

+

Public Key Encryption

+

Symmetric Encryption
```

The basic idea:

1. Server proves its identity.
2. Client and server create a secret key.
3. Data is encrypted using that key.

---

# SSL Handshake Process

## Step 1: Client Hello

Client sends:

- Supported SSL versions.
- Supported encryption algorithms.
- Random number.

Example:

```
Client:

Hello Server,
I want a secure connection.
```

---

## Step 2: Server Hello

Server responds with:

- Selected encryption method.
- SSL certificate.
- Public key.

Example:

```
Server:

Here is my certificate.
```

---

## Step 3: Certificate Verification

Client checks:

- Certificate validity.
- Certificate issuer.
- Domain name.
- Expiration date.

Example:

```
example.com

Certificate

Signed by Trusted CA
```

---

## Step 4: Key Exchange

Client and server create a shared secret key.

This key is used for encryption.

Important:

```
Server Private Key

never leaves the server.
```

---

## Step 5: Secure Communication

Now communication becomes encrypted:

```
Client

Encrypted Data

↓

Server
```

---

# SSL Encryption Concepts

## Asymmetric Encryption

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
Public Key:

Can be shared


Private Key:

Secret
```

---

## Symmetric Encryption

Uses:

```
One Shared Secret Key
```

Purpose:

Encrypt actual communication.

Reason:

It is faster than asymmetric encryption.

---

## Hashing

Used for:

- Data integrity.
- Detecting modification.

Example:

```
Original Data

↓

Hash

↓

Verification
```

---

# SSL Versions

## SSL 1.0

Never publicly released.

---

## SSL 2.0

Released in 1995.

Problems:

- Weak security.
- Vulnerable design.

Deprecated.

---

## SSL 3.0

Released in 1996.

Problems:

- Vulnerable to attacks like POODLE.

Deprecated.

---

# SSL vs TLS

| SSL | TLS |
|-|-|
| Older protocol | Modern protocol |
| Deprecated | Recommended |
| SSL 2.0/3.0 | TLS 1.2/1.3 |
| Less secure | More secure |
| Historical usage | Current Internet standard |

Modern systems use:

```
TLS

not SSL
```

---

# SSL Certificate

An SSL certificate is actually a **TLS certificate** in modern systems.

It contains:

- Domain name.
- Public key.
- Certificate authority.
- Expiry date.
- Digital signature.

Example:

```
api.company.com

Certificate

Issued by:

Let's Encrypt
```

---

# Certificate Authority (CA)

A trusted organization that verifies certificates.

Examples:

- Let's Encrypt
- DigiCert
- GlobalSign

Browsers trust certificates issued by trusted CAs.

---

# Common Pitfalls

## ❌ SSL and TLS are the same.

Wrong.

SSL is the old protocol.

TLS replaced SSL.

---

## ❌ HTTPS uses SSL today.

Usually wrong.

Modern HTTPS uses:

```
TLS
```

The term "SSL certificate" is still commonly used.

---

## ❌ SSL certificate means a website is safe.

Wrong.

A certificate only proves:

- Encrypted communication.
- Domain ownership.

It does not guarantee:

- No malware.
- No fraud.
- No vulnerabilities.

---

## ❌ SSL encrypts all network information.

Wrong.

SSL/TLS mainly protects application data.

IP addresses and network routing information are still visible.

---

## ❌ Private keys are shared.

Wrong.

Private keys always remain secret on the server.

---

# Spring Boot / Java Notes

Spring Boot applications today use:

```
TLS

not SSL
```

for secure communication.

However, Java terminology still contains SSL because the security APIs were named historically.

Example:

```
javax.net.ssl
```

This package supports:

- SSL
- TLS
- Certificates
- Secure sockets

---

# Spring Boot HTTPS Configuration

Example:

```properties
server.ssl.enabled=true

server.ssl.key-store=classpath:server.p12

server.ssl.key-store-password=password

server.ssl.key-store-type=PKCS12
```

The certificate and private key are stored in:

```
Keystore
```

---

# Java SSL Context

Java uses:

```
SSLContext
```

to manage secure communication.

Example:

```java
SSLContext context =
SSLContext.getInstance("TLS");
```

Even though the class name says SSL, modern applications use TLS.

---

# Production Architecture

Typical enterprise flow:

```
Client

HTTPS

↓

Load Balancer

TLS Termination

↓

Spring Boot Application

↓

Database
```

Certificates are usually managed at:

- Load balancer.
- API gateway.
- Reverse proxy.

---

# Common Spring Boot SSL Problems

## SSLHandshakeException

Possible causes:

- Expired certificate.
- Wrong certificate.
- Invalid trust chain.

---

## PKIX Path Building Failed

Meaning:

Java does not trust the certificate.

Solution:

Add certificate authority to trust store.

---

## Certificate Expired

Solution:

Renew certificate before expiry.

---

# Quick Revision

- SSL stands for Secure Sockets Layer.
- SSL is an old security protocol.
- SSL was replaced by TLS.
- Modern HTTPS uses TLS.
- SSL provides encryption, authentication, and integrity.
- SSL certificates are now actually TLS certificates.
- Java uses SSL-related APIs for TLS communication.
- Private keys never leave the server.

---

# Interview Questions

### 1. What is SSL?

SSL is an old cryptographic protocol used to secure communication between client and server.

---

### 2. Is SSL still used today?

No. SSL is deprecated. Modern systems use TLS.

---

### 3. Difference between SSL and TLS?

TLS is the improved and secure replacement for SSL.

---

### 4. Does HTTPS use SSL?

Modern HTTPS uses TLS, although people still commonly say "SSL certificate".

---

### 5. What does SSL provide?

SSL provides:

- Encryption.
- Authentication.
- Data integrity.

---

### 6. What is an SSL certificate?

A digital certificate that contains a public key and proves domain ownership.

---

### 7. Where is the private key stored?

Only on the server.

---

### 8. Why does Java have SSL classes if SSL is deprecated?

Java security APIs were named historically. They support modern TLS protocols.

---

### 9. What replaced SSL?

TLS (Transport Layer Security).

---

### 10. Why should a Java developer understand SSL/TLS?

Because secure APIs, HTTPS communication, certificates, and production deployments depend on TLS.

---

# References

- [[TLS]]
- [[HTTPS]]
- [[HTTP]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[Firewall]]
- [[Certificate]]
- [[Reverse Proxy]]
- [[Load Balancer]]