# IP Spoofing

## Definition

**IP Spoofing** is a network attack where an attacker **forges (fake) the source IP address** in IP packets to make them appear as if they came from another device.

The attacker hides their real identity or impersonates a trusted system.

> **Simple Definition**
>
> "IP Spoofing means sending packets with a fake source IP address."

Example:

```
Real Attacker IP

10.10.10.5

↓

Fake Source IP

192.168.1.100
```

The victim believes the packet came from `192.168.1.100`.

---

# How it Works

### Normal Communication

```
Client

192.168.1.10

↓

Server

192.168.1.20
```

The server knows the real client IP.

---

### During IP Spoofing

The attacker changes the source IP.

```
Attacker

10.10.10.5

↓

Spoofed Packet

Source IP

192.168.1.10

↓

Server
```

The server sees:

```
Request From

192.168.1.10
```

even though the packet came from the attacker.

---

### Important Note

The attacker usually **cannot receive the server's response** unless they are also performing a **Man-in-the-Middle (MITM)** attack or control the spoofed IP.

This is why IP Spoofing is commonly used for:

- DoS attacks
- DDoS attacks
- Reflection attacks

rather than normal two-way communication.

---

# Target Layer

**OSI Layer:** **Layer 3 (Network Layer)**

Reason:

- IP Spoofing manipulates the **IP header**.
- It changes the **Source IP Address** field.

---

# Prerequisites

For IP Spoofing:

- Ability to craft custom IP packets.
- Network that allows spoofed packets.
- Weak ingress/egress filtering.

Common attack scenarios:

- DDoS attacks
- Reflection attacks
- Trust-based authentication bypass
- Network scanning

---

# Real-world Example

Suppose a web server trusts requests from:

```
10.0.0.5
```

The attacker sends:

```
Source IP

10.0.0.5
```

The server believes:

```
Trusted Server

↓

Allowed
```

In reality:

```
Attacker

↓

Fake IP

↓

Server
```

---

# Impact

Possible consequences:

- Identity hiding.
- Bypassing weak IP-based trust rules.
- DoS attacks.
- DDoS attacks.
- Reflection and amplification attacks.
- Log manipulation.
- Difficult forensic investigation.

**By itself**, IP Spoofing does **not** allow an attacker to read encrypted traffic or hijack an established TCP session.

---

# Detection

Signs of IP Spoofing:

- Unexpected source IP addresses.
- Traffic from impossible geographic locations.
- Packets with invalid routing paths.
- Inconsistent TCP behavior.
- IDS/IPS alerts.
- Firewall log anomalies.

Useful tools:

- Wireshark
- tcpdump
- IDS/IPS
- Firewall logs
- NetFlow analysis

---

# Prevention

### 1. Ingress Filtering ✅

Routers reject packets with invalid source IP addresses.

---

### 2. Egress Filtering

Prevent internal devices from sending spoofed packets to the Internet.

---

### 3. Stateful Firewalls

Track active connections and reject unexpected packets.

---

### 4. Authentication

Never trust only an IP address.

Use:

- JWT
- OAuth2
- API Keys
- Mutual TLS (mTLS)

---

### 5. Anti-DDoS Protection

Use:

- CDN
- Web Application Firewall (WAF)
- Rate limiting
- DDoS protection services

---

### 6. Network Monitoring

Monitor unusual traffic patterns and spoofed packets.

---

# Spring Boot Protection

Spring Boot should **never trust the client IP address for authentication or authorization**.

Use:

- Spring Security.
- JWT/OAuth2.
- HTTPS.
- Mutual TLS (when needed).
- Rate limiting.
- Secure session management.

Bad example:

```java
if(clientIp.equals("10.0.0.5")){
    // Allow access
}
```

Good approach:

```
Authentication

↓

JWT

↓

Authorization
```

Use IP addresses only as an additional security signal, not as the primary identity check.

---

# Infrastructure Protection

Enterprise environments should implement:

- Stateful firewalls.
- Ingress/Egress filtering.
- Reverse proxies.
- Load balancers.
- IDS/IPS.
- Web Application Firewalls (WAF).
- DDoS protection.
- Network monitoring.

Example:

```
Internet

↓

CDN

↓

WAF

↓

Firewall

↓

Load Balancer

↓

Spring Boot Application
```

---

# Best Practices

- Never trust source IP alone.
- Enable ingress and egress filtering.
- Use strong authentication.
- Enable HTTPS everywhere.
- Deploy WAF and DDoS protection.
- Monitor firewall and network logs.
- Implement rate limiting.
- Validate proxy headers like `X-Forwarded-For` only from trusted proxies.

---

# Interview Questions

### 1. What is IP Spoofing?

IP Spoofing is the process of forging the source IP address in an IP packet.

---

### 2. Which OSI layer does IP Spoofing target?

Layer 3 (Network Layer).

---

### 3. Why do attackers use IP Spoofing?

To hide their identity, bypass weak IP-based trust, or launch DoS/DDoS and reflection attacks.

---

### 4. Can IP Spoofing establish a normal TCP connection?

Usually no.

The attacker normally cannot receive the server's replies unless they also control the spoofed address or perform a Man-in-the-Middle attack.

---

### 5. What is ingress filtering?

A router security feature that blocks packets with invalid or spoofed source IP addresses.

---

### 6. Does HTTPS prevent IP Spoofing?

No.

HTTPS encrypts application data but does not stop attackers from forging IP headers.

---

### 7. Can Spring Boot detect IP Spoofing?

Not reliably.

It should rely on strong authentication instead of trusting client IP addresses.

---

### 8. Is IP-based authentication secure?

No.

IP addresses can be spoofed or hidden behind proxies and NAT devices.

---

### 9. How do enterprises defend against IP Spoofing?

Using:

- Ingress/Egress filtering
- Stateful firewalls
- WAF
- IDS/IPS
- Strong authentication
- DDoS protection

---

### 10. How is IP Spoofing different from ARP Spoofing?

| IP Spoofing | ARP Spoofing |
|-------------|--------------|
| Targets Layer 3 | Targets Layer 2 |
| Fakes IP addresses | Fakes MAC-to-IP mappings |
| Can occur across networks | Limited to the local LAN |
| Common in DoS/DDoS attacks | Common in MITM attacks |

---

# References

- [[IP]]
- [[ARP Spoofing]]
- [[TCP-IP]]
- [[OSI Model]]
- [[Firewall]]
- [[HTTPS]]
- [[TLS]]
- [[Routing]]
- [[NAT]]
- [[Ports]]
- [[Man-in-the-Middle (MITM)]]
- [[DDoS]]