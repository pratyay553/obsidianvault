# Rogue DHCP

## Definition

A **Rogue DHCP Server** is an **unauthorized DHCP server** connected to a network that provides **fake IP configuration** to clients.

Instead of receiving network settings from the legitimate DHCP server, victims receive settings from the attacker's DHCP server.

> **Simple Definition**
>
> "A Rogue DHCP server tricks devices into using fake network settings controlled by an attacker."

---

# How it Works

### Normal DHCP Process

```
Client

↓

DHCP Discover

↓

Legitimate DHCP Server

↓

DHCP Offer

↓

DHCP Request

↓

DHCP ACK

↓

Valid Network Configuration
```

---

### During Rogue DHCP Attack

The attacker connects a fake DHCP server.

```
Client

↓

DHCP Discover

↓

Legitimate DHCP Server

      +

Rogue DHCP Server
```

If the rogue server responds first:

```
Client

↓

Accepts Fake DHCP Offer

↓

Fake IP

↓

Fake Gateway

↓

Fake DNS
```

Now all traffic follows the attacker's network settings.

---

# Target Layer

**Primary OSI Layer:** **Layer 2 (Data Link Layer)**

Reason:

- DHCP uses MAC addresses to identify clients.
- The rogue server is usually located within the same LAN.

**Uses:** UDP ports **67** and **68**.

---

# Prerequisites

For a Rogue DHCP attack:

- Attacker must connect to the same LAN.
- Clients must use DHCP.
- No DHCP Snooping enabled.
- The rogue server responds faster than the legitimate server.

Common targets:

- Office networks
- Public Wi-Fi
- Schools
- Enterprise LANs

---

# Real-world Example

Legitimate Gateway:

```
192.168.1.1
```

Attacker starts a Rogue DHCP Server.

Victim receives:

```
IP Address

192.168.1.100


Gateway

192.168.1.250   ← Attacker


DNS

192.168.1.250   ← Attacker
```

Traffic flow:

```
Victim

↓

Attacker

↓

Internet
```

The attacker can now inspect, redirect, or block traffic.

---

# Impact

Possible consequences:

- Man-in-the-Middle (MITM) attacks.
- Traffic interception.
- Fake DNS responses.
- Website redirection.
- Credential theft (if traffic is unencrypted).
- Malware distribution.
- Denial of Service.

A Rogue DHCP attack is often combined with:

- DNS Spoofing
- ARP Spoofing
- Phishing
- SSL stripping

---

# Detection

Signs of a Rogue DHCP Server:

- Users receive unexpected IP addresses.
- Wrong default gateway.
- Unknown DNS server.
- Multiple DHCP servers detected.
- Network instability.
- Security monitoring alerts.

Useful tools:

- Wireshark
- DHCP server logs
- Switch logs
- IDS/IPS
- `ipconfig /all` (Windows)
- `ip addr` and `ip route` (Linux)

Example:

```bash
ipconfig /all
```

Verify:

- DHCP Server
- Default Gateway
- DNS Server

---

# Prevention

### 1. DHCP Snooping ✅ (Best Protection)

Managed switches allow DHCP responses **only from trusted ports**.

Unauthorized DHCP servers are blocked.

---

### 2. Port Security

Restrict who can connect devices to switch ports.

---

### 3. 802.1X Authentication

Authenticate devices before granting network access.

---

### 4. Network Access Control (NAC)

Only approved devices can join the network.

---

### 5. Monitor DHCP Traffic

Alert if more than one DHCP server is detected.

---

### 6. Secure Physical Access

Prevent unauthorized devices from being connected to the network.

---

# Spring Boot Protection

Spring Boot cannot prevent Rogue DHCP attacks because they occur before application communication begins.

However, Spring Boot applications should:

- Use HTTPS/TLS.
- Validate certificates.
- Avoid trusting client-supplied IP information.
- Use secure authentication (JWT/OAuth2).
- Enable detailed security logging.

Even if network settings are manipulated, encrypted application traffic remains protected.

---

# Infrastructure Protection

Enterprise environments should implement:

- DHCP Snooping.
- Network Access Control (NAC).
- Port Security.
- VLAN segmentation.
- IDS/IPS.
- Secure switch configurations.
- Redundant DHCP servers.
- Continuous network monitoring.

Example:

```
Internet

↓

Firewall

↓

Managed Switch

↓

DHCP Snooping

↓

Authorized DHCP Server

↓

Spring Boot Servers
```

---

# Best Practices

- Enable DHCP Snooping on all managed switches.
- Use 802.1X authentication.
- Restrict physical access to network ports.
- Monitor DHCP server activity.
- Use secure DNS (DNSSEC where applicable).
- Always use HTTPS/TLS.
- Regularly audit network infrastructure.
- Train administrators to identify rogue devices.

---

# Interview Questions

### 1. What is a Rogue DHCP Server?

An unauthorized DHCP server that provides fake network configuration to clients.

---

### 2. What is the goal of a Rogue DHCP attack?

To redirect or control a victim's network traffic by supplying malicious IP configuration.

---

### 3. Which OSI layer does Rogue DHCP primarily target?

Layer 2 (Data Link Layer), while DHCP communication itself uses UDP.

---

### 4. What information can a Rogue DHCP Server provide?

- IP address
- Default gateway
- DNS server
- Subnet mask
- Lease time

---

### 5. What is the best defense against Rogue DHCP?

**DHCP Snooping** on managed switches.

---

### 6. Why is a fake DNS server dangerous?

It can redirect users to malicious websites even when they type correct domain names.

---

### 7. Can HTTPS reduce the impact of a Rogue DHCP attack?

Yes.

HTTPS encrypts traffic and verifies server identity, making credential theft and content manipulation much harder.

---

### 8. Can Spring Boot prevent Rogue DHCP attacks?

No.

The attack occurs in the network infrastructure before requests reach the application.

---

### 9. What enterprise security features help prevent Rogue DHCP?

- DHCP Snooping
- Port Security
- 802.1X
- NAC
- VLANs
- IDS/IPS

---

### 10. How is Rogue DHCP different from DHCP Starvation?

| Rogue DHCP | DHCP Starvation |
|------------|-----------------|
| Provides fake network settings | Exhausts the DHCP IP pool |
| Redirects user traffic | Prevents users from obtaining an IP |
| Used for MITM and phishing | Used for Denial of Service |
| Requires a fake DHCP server | Uses many fake DHCP requests |

---

# References

- [[DHCP]]
- [[DHCP Starvation]]
- [[ARP Spoofing]]
- [[DNS]]
- [[DNS Resolution]]
- [[DNS Spoofing]]
- [[OSI Model]]
- [[TCP-IP]]
- [[UDP]]
- [[Switching]]
- [[VLAN]]
- [[Firewall]]
- [[HTTPS]]
- [[TLS]]
- [[Man-in-the-Middle (MITM)]]