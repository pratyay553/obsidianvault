```markdown
# Packet Sniffing

## Definition

**Packet Sniffing** is the process of **capturing and analyzing network packets** traveling through a network.

It can be used for:

- Network troubleshooting.
- Performance monitoring.
- Security analysis.

Attackers can misuse packet sniffing to capture sensitive information such as:

- Usernames.
- Passwords.
- Session tokens.
- Application data.

> **Simple Definition**
>
> "Packet Sniffing means capturing network traffic and analyzing the data inside packets."

---

# How it Works

## Normal Network Communication

When a client communicates with a server:

```

Client

↓

Network Packets

↓

Server

```

Packets contain:

- Source IP.
- Destination IP.
- Protocol information.
- Application data.

---

## During Packet Sniffing

The attacker captures packets traveling through the network.

```

Client

↓

Network Traffic

↓

Attacker Sniffer

↓

Server

```

The attacker copies packets without interrupting communication.

---

# Packet Sniffing Methods

## 1. Passive Sniffing

The attacker only listens to traffic.

Common in:

- Hub-based networks.
- Shared wireless networks.

Example:

```

User

↓

Network

↓

Attacker Captures Packets

```

---

## 2. Active Sniffing

The attacker manipulates the network to capture traffic.

Techniques:

- ARP Spoofing.
- MAC Flooding.
- MITM attacks.

Example:

```

User

↓

Attacker

↓

Server

```

---

# Packet Capture Tools

Common legitimate tools:

- Wireshark.
- tcpdump.
- Network analyzers.

Security teams use them for:

- Troubleshooting.
- Threat detection.
- Performance analysis.

---

# Target Layer

Packet Sniffing can affect multiple layers.

Main targets:

## Layer 2 (Data Link Layer)

Because attackers capture:

- MAC addresses.
- Ethernet frames.

---

## Layer 3 (Network Layer)

Because packets contain:

- Source IP.
- Destination IP.

---

## Layer 4 (Transport Layer)

Because packets contain:

- TCP/UDP ports.
- Connection information.

---

# Prerequisites

For Packet Sniffing:

- Network access.
- Ability to capture traffic.
- Weak or missing encryption.

Common scenarios:

- Public Wi-Fi.
- Compromised machines.
- Internal enterprise networks.
- Misconfigured switches.

---

# Real-world Example

A user connects to public Wi-Fi.

Without HTTPS:

```

User

↓

HTTP Request

(username=password)

↓

Attacker Sniffer

```

The attacker can read the data.

---

With HTTPS:

```

User

↓

Encrypted HTTPS Traffic

↓

Attacker Sniffer

```

The attacker sees:

```

Encrypted Data

Cannot Read Content

```

---

# Impact

Possible consequences:

- Credential theft.
- Session hijacking.
- Data leakage.
- Exposure of sensitive information.
- Internal network reconnaissance.

Sensitive targets:

- Banking applications.
- APIs.
- Admin panels.
- Internal services.

---

# Detection

Packet sniffing is difficult to detect because passive sniffing does not modify traffic.

Possible indicators:

- Unknown devices on the network.
- Promiscuous mode detection.
- Unexpected ARP activity.
- Network monitoring alerts.

Useful tools:

- IDS/IPS.
- Network monitoring systems.
- Switch logs.
- Wireshark analysis.

---

# Prevention

## 1. Use Encryption ✅

Most important defense.

Use:

- HTTPS/TLS.
- VPN.
- SSH.
- IPsec.

Example:

```

Plain Data

↓

Encryption

↓

Network

↓

Encrypted Packets

```

---

## 2. Secure Wi-Fi

Use:

- WPA3.
- Strong passwords.
- Enterprise authentication.

---

## 3. Network Segmentation

Separate:

- User networks.
- Application networks.
- Database networks.

---

## 4. Switch Security

Enable:

- Port security.
- VLAN security.
- DHCP Snooping.
- Dynamic ARP Inspection.

---

## 5. Monitor Network Traffic

Use:

- IDS.
- IPS.
- SIEM.

---

# Spring Boot Protection

Spring Boot applications cannot prevent packet capture at the network level.

Application protection:

## 1. Always Use HTTPS

Example:

```

Client

↓

HTTPS/TLS

↓

Spring Boot API

```

Even if packets are captured, data remains encrypted.

---

## 2. Secure Authentication

Use:

- JWT.
- OAuth2.
- OpenID Connect.
- mTLS.

Avoid sending:

- Passwords.
- Tokens.
- Sensitive data without encryption.

---

## 3. Protect Session Tokens

Use:

- Secure cookies.
- HttpOnly cookies.
- SameSite protection.
- Token expiration.

---

## 4. API Security

Implement:

- Request validation.
- Rate limiting.
- Audit logging.

---

# Infrastructure Protection

Enterprise architecture:

```

Internet

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
- VPN for remote access.
- Network segmentation.
- IDS/IPS.
- Secure switches.
- Zero Trust architecture.

---

# Best Practices

- Encrypt all sensitive communication.
- Never use plain HTTP for production.
- Avoid transmitting passwords in plain text.
- Use secure Wi-Fi protocols.
- Segment networks.
- Monitor traffic patterns.
- Regularly audit network security.
- Disable unnecessary services.
- Use strong authentication.

---

# Interview Questions

### 1. What is Packet Sniffing?

Packet Sniffing is capturing and analyzing network packets traveling through a network.

---

### 2. Is Packet Sniffing always illegal?

No.

It is commonly used by:

- Network administrators.
- Security engineers.
- Developers for debugging.

It becomes illegal when used without authorization.

---

### 3. Which OSI layers are involved in Packet Sniffing?

Mainly:

- Layer 2.
- Layer 3.
- Layer 4.

---

### 4. What tools are used for packet sniffing?

Examples:

- Wireshark.
- tcpdump.
- Network analyzers.

---

### 5. Can HTTPS prevent Packet Sniffing?

HTTPS cannot stop packet capture, but it prevents attackers from reading the encrypted data.

---

### 6. Can Spring Boot detect Packet Sniffing?

No.

Packet capture happens outside the application layer.

---

### 7. Difference between HTTP and HTTPS regarding sniffing?

| HTTP | HTTPS |
|---|---|
| Data is plain text | Data is encrypted |
| Easy to read using sniffing | Difficult to read |
| No certificate validation | Uses TLS certificates |

---

### 8. What is passive sniffing?

Capturing traffic without modifying network communication.

---

### 9. What is active sniffing?

Manipulating the network to capture traffic, often using attacks like ARP Spoofing.

---

### 10. How can enterprises prevent packet sniffing?

Using:

- Encryption.
- Network segmentation.
- Secure switches.
- IDS/IPS.
- VPN.
- Strong authentication.

---

# References

- [[OSI Model]]
- [[TCP-IP]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[VPN]]
- [[ARP Spoofing]]
- [[MAC Flooding]]
- [[Firewall]]
- [[VLAN]]
- [[Switching]]
- [[Man-in-the-Middle (MITM)]]
- [[Spring Boot Security]]
```