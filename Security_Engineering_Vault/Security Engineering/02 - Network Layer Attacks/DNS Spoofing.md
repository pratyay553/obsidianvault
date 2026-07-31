# DNS Spoofing

## Definition

**DNS Spoofing** (also called **DNS Cache Poisoning**) is an attack where an attacker provides **fake DNS responses** to redirect users from a legitimate domain to a malicious server.

The user enters the correct website name, but DNS returns the attacker's IP address.

> **Simple Definition**
>
> "DNS Spoofing tricks users by replacing a real domain-to-IP mapping with a fake one."

Example:

Normal:

```
bank.com

↓

203.0.113.10
```

During attack:

```
bank.com

↓

192.168.1.50 (Attacker Server)
```

---

# How it Works

## Normal DNS Resolution

```
User

↓

DNS Resolver

↓

Authoritative DNS Server

↓

Correct IP Address

↓

Website
```

---

## During DNS Spoofing

The attacker injects a fake DNS response.

```
User

↓

DNS Query

↓

Fake DNS Response

↓

Malicious IP

↓

Fake Website
```

The victim believes they are visiting the real website.

---

# DNS Cache Poisoning Flow

Example:

User asks:

```
www.bank.com
```

Attacker sends fake response:

```
www.bank.com

↓

Attacker IP
```

DNS resolver stores the wrong record:

```
bank.com

↓

Attacker Server
```

Now other users receive the malicious IP until the cache expires.

---

# Attack Flow

```
Victim

↓

DNS Request

↓

DNS Resolver

↓

Fake DNS Reply

↓

Wrong IP Address

↓

Attacker Website
```

---

# Target Layer

**Primary OSI Layer:** **Layer 7 (Application Layer)**

Reason:

- DNS is an application-layer protocol.
- The attack manipulates domain name resolution.

It also involves:

- Layer 3 → IP communication
- Layer 4 → UDP/TCP DNS traffic

---

# Prerequisites

For DNS Spoofing:

- Ability to modify DNS responses.
- Access to the victim's network.
- Vulnerable DNS resolver.
- Weak DNS security configuration.

Common attack locations:

- Public Wi-Fi.
- Compromised routers.
- Malware-infected systems.
- Poorly configured DNS servers.

---

# Real-world Example

A user types:

```
www.mybank.com
```

Expected:

```
www.mybank.com

↓

Real Bank Server
```

After DNS Spoofing:

```
www.mybank.com

↓

Fake Bank Website
```

The fake website asks:

```
Username:

Password:
```

The attacker collects credentials.

---

# Impact

Possible consequences:

- Phishing attacks.
- Credential theft.
- Malware distribution.
- Traffic redirection.
- Data interception.
- Loss of customer trust.
- Business reputation damage.

---

# Detection

Signs of DNS Spoofing:

- Website redirects unexpectedly.
- SSL certificate warnings.
- Wrong IP address resolution.
- DNS records changing unexpectedly.
- Suspicious DNS traffic.

Useful tools:

### Check DNS Resolution

```bash
nslookup example.com
```

or

```bash
dig example.com
```

### Network Analysis

- Wireshark
- DNS monitoring tools
- IDS/IPS
- SIEM systems

---

# Prevention

## 1. DNSSEC ✅

**DNS Security Extensions** digitally sign DNS records.

It verifies:

```
DNS Response

↓

Authentic or Fake
```

---

## 2. Use HTTPS/TLS

HTTPS protects users even if DNS is manipulated.

Example:

```
Fake Server

↓

Certificate Check

↓

Browser Warning
```

---

## 3. Secure DNS Resolvers

Use trusted DNS services.

Examples:

- Enterprise DNS servers.
- Secure public DNS providers.

---

## 4. DNS Monitoring

Monitor:

- DNS changes.
- Unusual queries.
- Suspicious domains.

---

## 5. Secure Network Devices

Protect:

- Routers.
- DNS servers.
- Wi-Fi infrastructure.

---

# Spring Boot Protection

Spring Boot cannot directly prevent DNS Spoofing because DNS resolution happens before the HTTP request reaches the application.

However, Spring Boot applications should:

- Use HTTPS/TLS.
- Validate SSL certificates.
- Use secure service communication.
- Avoid trusting DNS names alone for authentication.
- Use mutual TLS (mTLS) for critical services.

Example:

```
Client

↓

DNS Resolution

↓

HTTPS Validation

↓

Spring Boot Application
```

---

# Microservices Protection

In microservices:

```
order-service

↓

payment-service.company.com

↓

DNS Resolution
```

Protection:

- Service mesh.
- mTLS.
- Internal DNS security.
- Certificate validation.

Examples:

- Kubernetes DNS.
- Istio.
- Consul.

---

# Infrastructure Protection

Enterprise environments should implement:

- DNSSEC.
- Secure DNS resolvers.
- DNS filtering.
- Firewall rules.
- IDS/IPS.
- Network monitoring.
- Secure router configuration.

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

Spring Boot Application
```

---

# Best Practices

- Enable DNSSEC where possible.
- Always use HTTPS.
- Validate TLS certificates.
- Do not trust DNS alone for authentication.
- Monitor DNS queries.
- Use secure DNS resolvers.
- Protect internal DNS servers.
- Keep network devices updated.
- Implement Zero Trust principles.

---

# Interview Questions

### 1. What is DNS Spoofing?

DNS Spoofing is an attack where fake DNS responses redirect users to malicious servers.

---

### 2. What is another name for DNS Spoofing?

DNS Cache Poisoning.

---

### 3. Which layer does DNS Spoofing target?

Primarily Layer 7 (Application Layer), because DNS is an application-layer protocol.

---

### 4. How does DNS Spoofing work?

An attacker provides false DNS information so a domain resolves to a malicious IP address.

---

### 5. What is DNSSEC?

DNSSEC is a security extension that digitally signs DNS records to verify authenticity.

---

### 6. Can HTTPS prevent DNS Spoofing?

No.

HTTPS does not stop DNS manipulation, but it protects users by validating certificates and encrypting communication.

---

### 7. Can Spring Boot prevent DNS Spoofing?

No.

DNS security must be handled by network infrastructure, but Spring Boot should use TLS and certificate validation.

---

### 8. How can DNS Spoofing be detected?

Using:

- DNS monitoring.
- `nslookup`
- `dig`
- Wireshark.
- IDS/IPS.

---

### 9. What happens if DNS Spoofing redirects users to a fake website?

Users may unknowingly submit credentials or sensitive information to attackers.

---

### 10. Difference between DNS Spoofing and DNS Hijacking?

| DNS Spoofing | DNS Hijacking |
|--------------|---------------|
| Provides fake DNS responses | Takes control of DNS settings |
| Often temporary | Usually longer-term |
| Can affect individual users | Can affect entire domains |
| Uses manipulation | Uses unauthorized control |

---

# References

- [[DNS]]
- [[DNS Resolution]]
- [[DNSSEC]]
- [[HTTPS]]
- [[TLS]]
- [[IP Spoofing]]
- [[ARP Spoofing]]
- [[Man-in-the-Middle (MITM)]]
- [[Firewall]]
- [[VPN]]
- [[OSI Model]]
- [[TCP-IP]]