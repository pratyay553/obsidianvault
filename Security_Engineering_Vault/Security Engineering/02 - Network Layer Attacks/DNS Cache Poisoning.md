```markdown
# DNS Cache Poisoning

## Definition

**DNS Cache Poisoning** is a DNS attack where an attacker inserts **fake DNS records into a DNS resolver's cache**.

After poisoning, users who query that DNS resolver receive the attacker's fake IP address instead of the legitimate server IP.

> **Simple Definition**
>
> "DNS Cache Poisoning corrupts stored DNS information so users are redirected to a fake server."

Example:

Before attack:

```

amazon.com

↓

203.0.113.10

```

After poisoning:

```

amazon.com

↓

198.51.100.50 (Attacker Server)

```

---

# How it Works

## Normal DNS Caching

DNS caching improves performance.

Flow:

```

User

↓

DNS Resolver

↓

Authoritative DNS Server

↓

Correct IP

↓

DNS Cache Stores Result

↓

Future Users Get Faster Response

```

Example cache:

```

google.com

↓

142.250.x.x

TTL: 3600 seconds

```

---

## During DNS Cache Poisoning

The attacker sends a fake DNS response before the real response arrives.

Flow:

```

User

↓

DNS Query

↓

DNS Resolver

↓

Attacker Fake Response

↓

DNS Cache Updated

↓

Wrong IP Stored

```

Now:

```

User

↓

DNS Resolver

↓

Fake IP

↓

Malicious Website

```

---

# Attack Flow

```

Attacker

↓

Fake DNS Response

↓

DNS Resolver

↓

Poisoned Cache

↓

Users

↓

Fake Website

```

The attack can affect many users because the poisoned record remains cached until the TTL expires.

---

# Target Layer

**Primary OSI Layer:** Layer 7 (Application Layer)

Reason:

- DNS is an application-layer protocol.
- The attack manipulates domain name resolution.

Supporting layers:

- Layer 4 → DNS uses UDP/TCP.
- Layer 3 → IP addresses are changed.

---

# Prerequisites

For DNS Cache Poisoning:

- Access to influence DNS responses.
- Vulnerable DNS resolver.
- Weak DNS security configuration.
- Ability to guess or manipulate DNS query information.

Common targets:

- Public DNS resolvers.
- Enterprise DNS servers.
- Home routers with insecure DNS forwarding.
- Old DNS software.

---

# Real-world Example

A user wants to visit:

```

[www.bank.com](http://www.bank.com)

```

Correct DNS:

```

[www.bank.com](http://www.bank.com)

↓

Bank Server IP

```

After cache poisoning:

```

[www.bank.com](http://www.bank.com)

↓

Attacker Server IP

````

User sees a fake banking website.

The attacker may collect:

- Username.
- Password.
- Payment information.

---

# Impact

Possible consequences:

- Phishing attacks.
- Credential theft.
- Malware installation.
- Traffic redirection.
- Data interception.
- Customer trust damage.
- Large-scale user compromise.

Because DNS is a trusted service, successful poisoning can affect many users.

---

# Detection

Signs of DNS Cache Poisoning:

- Domain resolves to an unexpected IP.
- SSL certificate warnings.
- DNS records change unexpectedly.
- Different users receive different IP addresses.
- Suspicious DNS responses.

Useful tools:

## Check DNS Records

```bash
nslookup example.com
````

or

```bash
dig example.com
```

Example:

```
Expected:

example.com → 10.10.10.10


Received:

example.com → 192.168.1.50
```

---

## Network Monitoring

Tools:

- Wireshark
    
- IDS/IPS
    
- SIEM systems
    
- DNS monitoring platforms
    

---

# Prevention

## 1. DNSSEC ✅

DNSSEC adds digital signatures to DNS records.

It verifies:

```
DNS Response

↓

Authentic or Modified
```

---

## 2. Random DNS Query Parameters

Modern DNS resolvers use:

- Random source ports.
    
- Random transaction IDs.
    

This makes spoofing harder.

---

## 3. Secure DNS Software

Keep DNS servers:

- Updated.
    
- Patched.
    
- Properly configured.
    

---

## 4. Use Trusted DNS Resolvers

Avoid unknown DNS servers.

---

## 5. Limit DNS Cache Access

Only authorized systems should manage DNS infrastructure.

---

## 6. Monitor DNS Changes

Detect:

- Unexpected record changes.
    
- Abnormal DNS responses.
    
- Suspicious domains.
    

---

# Spring Boot Protection

Spring Boot cannot directly prevent DNS Cache Poisoning because DNS resolution happens before application communication.

However, Spring Boot applications should:

## Use HTTPS/TLS

Example:

```
Client

↓

DNS Resolution

↓

TLS Certificate Validation

↓

Spring Boot Application
```

A fake server without the correct certificate will trigger warnings.

---

## Use Strong Authentication

Do not trust:

- Domain names alone.
    
- IP addresses alone.
    

Use:

- JWT.
    
- OAuth2.
    
- mTLS.
    

---

## Microservices Protection

Example:

```
order-service

↓

payment-service.company.com
```

Protection:

- Service mesh.
    
- mTLS.
    
- Internal DNS security.
    
- Certificate validation.
    

---

# Infrastructure Protection

Enterprise environments should implement:

- DNSSEC.
    
- Secure DNS resolvers.
    
- DNS monitoring.
    
- Firewall rules.
    
- IDS/IPS.
    
- Access control.
    
- Network segmentation.
    

Example:

```
Internet

↓

Firewall

↓

Secure DNS Resolver

↓

Load Balancer

↓

Spring Boot Services
```

---

# Best Practices

- Enable DNSSEC where possible.
    
- Use trusted DNS resolvers.
    
- Keep DNS software updated.
    
- Monitor DNS traffic.
    
- Protect DNS servers.
    
- Use HTTPS everywhere.
    
- Validate TLS certificates.
    
- Do not use IP-based trust as authentication.
    
- Implement Zero Trust security.
    
- Regularly audit DNS records.
    

---

# Interview Questions

### 1. What is DNS Cache Poisoning?

DNS Cache Poisoning is an attack where fake DNS records are inserted into a DNS resolver cache.

---

### 2. Why is DNS Cache Poisoning dangerous?

Because users are redirected to malicious servers even when they enter the correct domain name.

---

### 3. Which OSI layer does DNS Cache Poisoning target?

Primarily Layer 7 (Application Layer).

---

### 4. What is the difference between DNS Spoofing and DNS Cache Poisoning?

|DNS Spoofing|DNS Cache Poisoning|
|---|---|
|Fake DNS responses are provided|Fake records are stored in DNS cache|
|Can affect individual queries|Can affect many users|
|Often temporary|Remains until cache expires|
|Broader attack category|Specific DNS cache attack|

---

### 5. What is DNSSEC?

DNSSEC is a security extension that digitally signs DNS records to verify authenticity.

---

### 6. Can HTTPS prevent DNS Cache Poisoning?

No.

HTTPS cannot stop DNS poisoning, but certificate validation can protect users from fake websites.

---

### 7. Can Spring Boot prevent DNS Cache Poisoning?

No.

DNS security belongs to infrastructure, but applications should use TLS and strong authentication.

---

### 8. How can DNS Cache Poisoning be detected?

Using:

- DNS monitoring.
    
- `dig`
    
- `nslookup`
    
- Wireshark.
    
- IDS/IPS.
    

---

### 9. Why does DNS Cache Poisoning have a large impact?

Because one poisoned DNS resolver can affect many users using that resolver.

---

### 10. How do enterprises protect against DNS Cache Poisoning?

Using:

- DNSSEC.
    
- Secure DNS resolvers.
    
- Monitoring.
    
- Access control.
    
- Network security controls.
    

---

# References

- [[DNS]]
    
- [[DNS Resolution]]
    
- [[DNS Spoofing]]
    
- [[DNSSEC]]
    
- [[HTTPS]]
    
- [[TLS]]
    
- [[IP Spoofing]]
    
- [[ARP Spoofing]]
    
- [[Firewall]]
    
- [[VPN]]
    
- [[OSI Model]]
    
- [[TCP-IP]]