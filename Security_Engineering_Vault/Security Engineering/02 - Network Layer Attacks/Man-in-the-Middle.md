```markdown
# Man-in-the-Middle (MITM)

## Definition

A **Man-in-the-Middle (MITM)** attack is a security attack where an attacker secretly places themselves **between two communicating parties** and intercepts or modifies their communication.

The attacker makes both sides believe they are communicating directly with each other.

> **Simple Definition**
>
> "MITM means an attacker secretly sits between client and server to read, modify, or steal communication."

Example:

Normal:

```

Client

↓

Server

```

During MITM:

```

Client

↓

Attacker

↓

Server

```

---

# How it Works

## Normal Communication

A client sends data directly to a server.

```

Browser

↓

Website Server

```

The client trusts the server.

---

## During MITM Attack

The attacker intercepts communication.

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
- Inject malicious content.
- Steal credentials.
- Replay requests.

---

# Common MITM Techniques

## 1. ARP Spoofing

The attacker sends fake ARP messages inside a LAN.

Example:

```

Victim

↓

Attacker MAC Address

↓

Router

```

Traffic passes through the attacker.

---

## 2. DNS Spoofing

The attacker provides fake DNS responses.

Example:

```

bank.com

↓

Fake Bank Server

```

---

## 3. Rogue Wi-Fi

A fake wireless network is created.

Example:

```

Free Airport WiFi

↓

Attacker Network

```

---

## 4. SSL Stripping

The attacker downgrades HTTPS communication to HTTP.

Example:

```

HTTPS

↓

HTTP

↓

Attacker Reads Data

```

Modern browsers and HSTS reduce this risk.

---

## 5. Session Hijacking

The attacker steals session cookies.

Example:

```

User Session Token

↓

Attacker Uses Same Session

```

---

# Attack Flow

```

Client

↓

Request

↓

Attacker Intercepts

↓

Forward Request

↓

Server

Server Response

↓

Attacker Intercepts

↓

Client

```

---

# Target Layer

MITM attacks can target multiple layers.

## Layer 2 (Data Link Layer)

Examples:

- ARP Spoofing.
- MAC attacks.

---

## Layer 3 (Network Layer)

Examples:

- IP Spoofing.
- Routing attacks.

---

## Layer 7 (Application Layer)

Examples:

- DNS Spoofing.
- HTTP manipulation.

---

# Prerequisites

For MITM:

- Ability to intercept communication.
- Network access.
- Weak or missing encryption.
- User trust in attacker-controlled infrastructure.

Common environments:

- Public Wi-Fi.
- Corporate networks.
- Compromised routers.
- Insecure applications.

---

# Real-world Example

A user connects to a fake Wi-Fi network:

```

User

↓

"Free Airport WiFi"

↓

Attacker

↓

Internet

```

User opens:

```

example.com

```

The attacker captures:

- Login credentials.
- Cookies.
- Personal information.

---

## Banking Example

User visits:

```

bank.com

```

MITM attacker modifies:

```

Transfer Amount:

₹1000

↓

₹100000

````

Without proper security controls.

---

# Impact

Possible consequences:

- Credential theft.
- Data modification.
- Session hijacking.
- Financial fraud.
- Privacy loss.
- Malware injection.
- Unauthorized access.

---

# Detection

MITM detection can be difficult.

Possible indicators:

- Certificate warnings.
- Unexpected SSL errors.
- Unknown Wi-Fi networks.
- Duplicate IP/MAC addresses.
- Strange network behavior.
- DNS changes.

Useful tools:

- Wireshark.
- IDS/IPS.
- Network monitoring tools.
- Certificate monitoring.

Network commands:

```bash
arp -a
````

Check ARP mappings.

```bash
traceroute example.com
```

Check network path.

---

# Prevention

## 1. Use HTTPS/TLS ✅

TLS provides:

- Encryption.
    
- Server authentication.
    
- Data integrity.
    

Example:

```
Client

↓

Encrypted TLS Channel

↓

Server
```

---

## 2. Certificate Validation

Never ignore browser certificate warnings.

Certificates verify:

```
Server Identity

+

Encryption Key
```

---

## 3. Use VPN on Untrusted Networks

VPN creates an encrypted tunnel.

```
Device

↓

VPN Tunnel

↓

Secure Network
```

---

## 4. Enable HSTS

HTTP Strict Transport Security forces HTTPS.

Example:

```
HTTP Request

↓

Blocked

↓

HTTPS Only
```

---

## 5. Strong Authentication

Use:

- MFA.
    
- OAuth2.
    
- JWT.
    
- Hardware security keys.
    

---

## 6. Network Security

Use:

- Secure Wi-Fi.
    
- VLAN segmentation.
    
- Firewalls.
    
- IDS/IPS.
    

---

# Spring Boot Protection

Spring Boot applications must assume network communication can be attacked.

## 1. Enforce HTTPS

Configuration:

```
Client

↓

TLS

↓

Spring Boot Application
```

Never expose production APIs over HTTP.

---

## 2. Use Spring Security

Implement:

- Authentication.
    
- Authorization.
    
- Role-based access control.
    

Example:

```
User

↓

JWT Token

↓

Spring Security Filter

↓

Controller
```

---

## 3. Secure Cookies

Use:

```
HttpOnly=true

Secure=true

SameSite=Strict
```

Protection against session theft.

---

## 4. Validate Requests

Implement:

- Input validation.
    
- CSRF protection.
    
- Request integrity checks.
    

---

## 5. Microservices Security

For internal communication:

Use:

- mTLS.
    
- Service mesh.
    
- Certificate rotation.
    

Example:

```
Order Service

↓

mTLS

↓

Payment Service
```

---

# Infrastructure Protection

Enterprise architecture:

```
Internet

↓

DDoS Protection

↓

Firewall

↓

WAF

↓

Load Balancer

↓

Spring Boot Application

↓

Database
```

Security controls:

- TLS everywhere.
    
- VPN access.
    
- IDS/IPS.
    
- Network segmentation.
    
- Zero Trust architecture.
    
- Secure DNS.
    
- Certificate management.
    

---

# Best Practices

- Always use HTTPS.
    
- Validate TLS certificates.
    
- Never ignore certificate warnings.
    
- Use MFA.
    
- Avoid unsecured public Wi-Fi.
    
- Use VPN for sensitive access.
    
- Protect session tokens.
    
- Implement mTLS for internal services.
    
- Monitor network traffic.
    
- Use strong authentication.
    
- Follow Zero Trust principles.
    

---

# Interview Questions

### 1. What is a Man-in-the-Middle attack?

A MITM attack occurs when an attacker intercepts communication between two parties to read or modify data.

---

### 2. Which OSI layers can MITM attack?

Multiple layers:

- Layer 2.
    
- Layer 3.
    
- Layer 7.
    

---

### 3. What are common MITM techniques?

Examples:

- ARP Spoofing.
    
- DNS Spoofing.
    
- Rogue Wi-Fi.
    
- SSL Stripping.
    
- Session Hijacking.
    

---

### 4. Can HTTPS prevent MITM attacks?

HTTPS significantly reduces MITM risk by providing encryption and certificate validation, but poor certificate handling can still create vulnerabilities.

---

### 5. Can Spring Boot prevent MITM attacks?

Spring Boot cannot control the network path, but it can protect communication using TLS, authentication, and secure session management.

---

### 6. What is SSL Stripping?

An attack where HTTPS connections are downgraded to HTTP so attackers can read traffic.

---

### 7. How does TLS protect against MITM?

TLS provides:

- Encryption.
    
- Server authentication.
    
- Message integrity.
    

---

### 8. Difference between MITM and Packet Sniffing?

|MITM|Packet Sniffing|
|---|---|
|Attacker sits between communication|Attacker captures traffic|
|Can modify data|Usually only observes|
|Often active attack|Can be passive|
|Requires traffic interception|Requires packet capture|

---

### 9. Why is public Wi-Fi risky?

Attackers can create fake networks or intercept traffic from insecure connections.

---

### 10. How do enterprises prevent MITM attacks?

Using:

- TLS.
    
- VPN.
    
- MFA.
    
- IDS/IPS.
    
- Network segmentation.
    
- Certificate management.
    
- Zero Trust security.
    

---

# References

- [[ARP Spoofing]]
    
- [[DNS Spoofing]]
    
- [[DNS Cache Poisoning]]
    
- [[Packet Sniffing]]
    
- [[IP Spoofing]]
    
- [[Routing Attack]]
    
- [[BGP Hijacking]]
    
- [[HTTPS]]
    
- [[TLS]]
    
- [[VPN]]
    
- [[Firewall]]
    
- [[Spring Security]]
    
- [[OSI Model]]
    
- [[TCP-IP]]