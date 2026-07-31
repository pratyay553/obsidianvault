# ARP Spoofing

## Definition

**ARP Spoofing (ARP Poisoning)** is a network attack where an attacker sends **fake ARP messages** to trick devices into believing the attacker's MAC address belongs to another device (usually the router).

As a result, network traffic is sent to the attacker instead of the real destination.

> **Simple Definition**
>
> "ARP Spoofing tricks devices into sending their network traffic to the attacker."

Example:

```
Victim

↓

Attacker

↓

Router
```

Instead of:

```
Victim

↓

Router
```

---

# How it Works

### Normal Communication

```
Victim

↓

"Who has Router IP?"

↓

Router replies

↓

"My MAC is AA:BB:CC"
```

The victim stores:

```
Router IP

↓

AA:BB:CC
```

---

### During ARP Spoofing

Attacker sends fake ARP replies:

```
Router IP

↓

Attacker MAC
```

Victim updates its ARP cache:

```
Router IP

↓

Attacker MAC
```

Now all traffic goes to the attacker.

```
Victim

↓

Attacker

↓

Router
```

The attacker can forward the traffic to avoid suspicion.

---

# Target Layer

**OSI Layer:** **Layer 2 (Data Link Layer)**

Reason:

- ARP works using **MAC addresses**.
- It is only used inside a **Local Area Network (LAN)**.

---

# Prerequisites

For ARP Spoofing to work:

- Attacker must be on the **same LAN**.
- ARP protocol must be in use (IPv4).
- Victim accepts fake ARP replies.

Common targets:

- Public Wi-Fi
- Office LAN
- College networks
- Internal enterprise networks

---

# Real-world Example

Suppose:

```
Victim

192.168.1.10
```

Router:

```
192.168.1.1
```

Attacker:

```
192.168.1.50
```

The attacker sends:

```
192.168.1.1

↓

MAC = Attacker
```

Victim believes:

```
Router

↓

Attacker MAC
```

Now:

```
Victim

↓

Attacker

↓

Router

↓

Internet
```

The attacker can:

- Read traffic.
- Modify traffic.
- Drop packets.

---

# Impact

Possible consequences:

- Man-in-the-Middle (MITM) attacks.
- Password theft (if traffic is unencrypted).
- Session hijacking.
- Data modification.
- Network disruption.
- Denial of Service (DoS).

If HTTPS is **not** used:

```
Username

↓

Attacker reads it
```

If HTTPS **is** used:

Traffic is encrypted, making it much harder to read.

---

# Detection

Signs of ARP Spoofing:

- Sudden network slowdown.
- Frequent disconnections.
- Duplicate MAC address warnings.
- Unexpected ARP table changes.
- Certificate warnings (during MITM attempts).

Tools commonly used:

- `arp -a`
- Wireshark
- arpwatch
- Intrusion Detection Systems (IDS)

Example:

```bash
arp -a
```

Shows the current ARP cache.

---

# Prevention

### 1. Use HTTPS/TLS

Even if traffic is intercepted, encryption protects the data.

---

### 2. Static ARP Entries

Critical devices can use fixed ARP mappings.

Suitable for:

- Servers
- Routers
- Network appliances

---

### 3. Dynamic ARP Inspection (DAI)

Enterprise switches can verify ARP packets before forwarding them.

---

### 4. VLAN Segmentation

Separate users into different VLANs.

This limits the attacker's reach.

---

### 5. Secure Wi-Fi

Use:

- WPA2
- WPA3

Avoid open public networks.

---

### 6. VPN

A VPN encrypts traffic between the device and VPN server.

Even if ARP Spoofing occurs, the attacker cannot easily read encrypted traffic.

---

# Spring Boot Protection

Spring Boot cannot prevent ARP Spoofing because it occurs below the application layer.

However, Spring Boot applications should:

- Use HTTPS only.
- Enable TLS 1.2/1.3.
- Validate certificates.
- Use secure authentication (JWT/OAuth2).
- Encrypt sensitive data.
- Enable security logging.

Example:

```
Browser

↓

HTTPS

↓

Spring Boot
```

Even if the attacker captures packets, TLS protects the application data.

---

# Infrastructure Protection

Enterprise networks should implement:

- Managed switches with Dynamic ARP Inspection (DAI).
- VLAN segmentation.
- Network Access Control (NAC).
- Firewalls.
- IDS/IPS monitoring.
- Secure switch configurations.
- Private management networks.

Example:

```
Internet

↓

Firewall

↓

Core Switch (DAI)

↓

VLAN 100

↓

Spring Boot Servers
```

---

# Best Practices

- Always use HTTPS.
- Never send passwords over HTTP.
- Keep switches and routers updated.
- Use VLANs to isolate users.
- Monitor ARP tables for unusual changes.
- Enable Dynamic ARP Inspection where supported.
- Use VPN on untrusted networks.
- Implement Zero Trust principles in enterprise networks.

---

# Interview Questions

### 1. What is ARP Spoofing?

An attack where fake ARP messages redirect network traffic to the attacker.

---

### 2. Which OSI layer does ARP Spoofing target?

Layer 2 (Data Link Layer).

---

### 3. Why is ARP Spoofing dangerous?

Because it enables Man-in-the-Middle attacks, allowing attackers to intercept or modify traffic.

---

### 4. Can HTTPS prevent ARP Spoofing?

No. HTTPS does not stop the attack itself, but it encrypts the traffic, greatly reducing what the attacker can read or modify.

---

### 5. How can ARP Spoofing be detected?

Using tools like:

- `arp -a`
- Wireshark
- arpwatch
- IDS/IPS solutions

---

### 6. What is Dynamic ARP Inspection (DAI)?

A switch security feature that validates ARP packets and blocks malicious ARP messages.

---

### 7. Can ARP Spoofing happen over the Internet?

No.

It requires the attacker to be on the same local network because ARP works only within a LAN.

---

### 8. Can Spring Boot stop ARP Spoofing?

No.

Spring Boot runs at the application layer, while ARP Spoofing happens at the network's Data Link layer.

---

### 9. How does a VPN help against ARP Spoofing?

A VPN encrypts network traffic, making intercepted data much harder to read.

---

### 10. How do enterprises protect against ARP Spoofing?

Using:

- HTTPS/TLS
- Dynamic ARP Inspection
- VLANs
- IDS/IPS
- VPNs
- Secure switch configurations
- Network monitoring

---

# References

- [[ARP]]
- [[MAC Address]]
- [[OSI Model]]
- [[TCP-IP]]
- [[Switching]]
- [[VLAN]]
- [[HTTPS]]
- [[TLS]]
- [[Firewall]]
- [[VPN]]
- [[Man-in-the-Middle (MITM)]]