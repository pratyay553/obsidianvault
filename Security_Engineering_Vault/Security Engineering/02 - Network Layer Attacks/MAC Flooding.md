# MAC Flooding

## Definition

**MAC Flooding** is a network attack where an attacker sends thousands of fake MAC addresses to a network switch, causing its **MAC Address Table (CAM Table)** to become full.

When the table is full, the switch starts behaving like a **hub** and sends packets to all ports, allowing the attacker to capture network traffic.

> **Simple Definition**
>
> "MAC Flooding fills a switch's MAC table with fake entries so the switch broadcasts traffic to every device."

---

# How it Works

### Normal Communication

```
Laptop

↓

Switch

↓

Server
```

The switch knows:

```
Laptop MAC

↓

Port 1


Server MAC

↓

Port 2
```

Traffic goes only to the correct port.

---

### During MAC Flooding

Attacker sends thousands of fake MAC addresses.

```
Fake MAC 1

↓

Switch

Fake MAC 2

↓

Switch

Fake MAC 3

↓

Switch

...
```

Eventually:

```
MAC Table

↓

FULL
```

Now the switch cannot learn new MAC addresses.

---

### Result

The switch starts forwarding unknown traffic to all ports.

```
Victim

↓

Switch

↓

All Devices
```

The attacker can capture packets using tools like Wireshark.

---

# Target Layer

**OSI Layer:** **Layer 2 (Data Link Layer)**

Reason:

- MAC Flooding targets the switch's **MAC Address Table**.
- It uses **MAC addresses**, not IP addresses.

---

# Prerequisites

For a MAC Flooding attack:

- The attacker must be connected to the **same LAN**.
- A vulnerable switch without port security.
- Ability to send many fake MAC addresses.

Common targets:

- Office networks
- College labs
- Public LANs
- Older unmanaged switches

---

# Real-world Example

Network:

```
Laptop

↓

Switch

↓

Server

↓

Printer

↓

Attacker
```

The attacker floods the switch with fake MAC addresses.

```
AA:AA:AA:AA:AA:01

AA:AA:AA:AA:AA:02

AA:AA:AA:AA:AA:03

...
```

The switch's MAC table becomes full.

Now traffic between the Laptop and Server is broadcast.

The attacker captures the packets.

---

# Impact

Possible consequences:

- Packet sniffing.
- Sensitive data exposure.
- Man-in-the-Middle (combined with ARP Spoofing).
- Network slowdown.
- Reduced switch performance.
- Confidential information leakage.

If traffic is encrypted with HTTPS/TLS, the attacker may capture packets but cannot easily read their contents.

---

# Detection

Signs of a MAC Flooding attack:

- Switch performance degradation.
- Unusual number of MAC addresses.
- High broadcast traffic.
- Frequent CAM table changes.
- Network monitoring alerts.

Common tools:

- Switch management console.
- Wireshark.
- SNMP monitoring.
- IDS/IPS.
- Network monitoring systems.

---

# Prevention

### 1. Enable Port Security

Limit how many MAC addresses can be learned on a switch port.

Example:

```
Port 1

↓

Maximum 2 MAC Addresses
```

---

### 2. Sticky MAC

The switch automatically learns and remembers valid MAC addresses.

Unknown MAC addresses are blocked.

---

### 3. Disable Unused Ports

Unused switch ports should be disabled.

---

### 4. VLAN Segmentation

Separate departments into different VLANs.

This reduces the attack's scope.

---

### 5. Managed Switches

Enterprise switches provide protection against CAM table overflow.

---

### 6. Monitor MAC Table

Continuously monitor for sudden increases in learned MAC addresses.

---

# Spring Boot Protection

Spring Boot cannot prevent MAC Flooding because the attack happens below the application layer.

However, applications should:

- Use HTTPS.
- Enable TLS.
- Encrypt sensitive data.
- Use secure authentication.
- Validate sessions.
- Log suspicious activities.

Example:

```
Client

↓

HTTPS

↓

Spring Boot
```

Even if packets are captured, encryption protects the data.

---

# Infrastructure Protection

Enterprise environments should use:

- Managed switches.
- Port Security.
- Sticky MAC.
- VLANs.
- IDS/IPS.
- Network monitoring.
- NAC (Network Access Control).

Example:

```
Internet

↓

Firewall

↓

Core Switch

↓

Port Security

↓

Spring Boot Servers
```

---

# Best Practices

- Enable Port Security on all access ports.
- Limit the number of MAC addresses per port.
- Use managed enterprise switches.
- Monitor switch CAM tables.
- Disable unused ports.
- Segment the network using VLANs.
- Always use HTTPS/TLS.
- Regularly update switch firmware.

---

# Interview Questions

### 1. What is MAC Flooding?

A Layer 2 attack that fills a switch's MAC Address Table with fake MAC addresses.

---

### 2. Which OSI layer does MAC Flooding target?

Layer 2 (Data Link Layer).

---

### 3. What is the goal of MAC Flooding?

To force a switch to broadcast traffic so the attacker can capture packets.

---

### 4. What happens when the MAC table becomes full?

The switch may start forwarding unknown traffic to all ports, behaving similarly to a hub.

---

### 5. Can MAC Flooding happen over the Internet?

No.

It requires the attacker to be on the same local network.

---

### 6. How do enterprises prevent MAC Flooding?

By using:

- Port Security
- Sticky MAC
- VLANs
- Managed switches
- IDS/IPS

---

### 7. Does HTTPS stop MAC Flooding?

No.

HTTPS does not prevent the attack, but it encrypts the captured traffic.

---

### 8. Can Spring Boot prevent MAC Flooding?

No.

It occurs at the network's Data Link layer, outside the application's control.

---

### 9. What is Port Security?

A switch feature that limits the number of MAC addresses allowed on a switch port.

---

### 10. How is MAC Flooding different from ARP Spoofing?

| MAC Flooding | ARP Spoofing |
|--------------|--------------|
| Fills the switch's MAC table | Sends fake ARP messages |
| Targets the switch | Targets end devices |
| Makes the switch flood traffic | Redirects traffic through the attacker |
| Layer 2 attack | Layer 2 attack |

---

# References

- [[Switching]]
- [[MAC Address]]
- [[ARP Spoofing]]
- [[OSI Model]]
- [[TCP-IP]]
- [[VLAN]]
- [[HTTPS]]
- [[TLS]]
- [[Firewall]]
- [[VPN]]
- [[Man-in-the-Middle (MITM)]]