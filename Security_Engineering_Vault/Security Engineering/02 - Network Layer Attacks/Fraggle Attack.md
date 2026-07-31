# Fraggle Attack

## Definition

A **Fraggle Attack** is a **Distributed Denial-of-Service (DDoS)** amplification attack that is similar to a **Smurf Attack**, but instead of using **ICMP (Ping)**, it uses **UDP** packets.

The attacker sends **UDP packets** to a network's **broadcast address** while **spoofing the victim's IP address**. Every device receiving the broadcast sends a reply to the victim, overwhelming it with traffic.

> **Simple Definition**
>
> "A Fraggle Attack tricks many devices into sending UDP replies to a victim, causing a DDoS attack."

---

# How it Works

### Normal UDP Communication

```
Client

↓

UDP Request

↓

Server

↓

UDP Response
```

---

### During a Fraggle Attack

The attacker sends:

```
Source IP

↓

Victim IP (Spoofed)


Destination

↓

Broadcast Address
```

Example:

```
Destination

192.168.1.255
```

All devices on the network receive the UDP packet.

```
PC 1

↓

Victim

PC 2

↓

Victim

PC 3

↓

Victim

Printer

↓

Victim

Server

↓

Victim
```

The victim receives replies from every device.

---

# Attack Flow

```
Attacker

↓

Spoofed UDP Packet

↓

Broadcast Address

↓

All Devices Respond

↓

Victim
```

One packet can trigger hundreds of responses.

This makes Fraggle an **amplification attack**.

---

# Target Layer

**OSI Layer:** **Layer 4 (Transport Layer)**

Reason:

- The attack primarily abuses the **UDP protocol**.
- It also relies on **IP Spoofing (Layer 3)**.

---

# Prerequisites

For a Fraggle Attack:

- Ability to spoof the victim's IP address.
- Network allows directed broadcasts.
- Devices respond to broadcast UDP requests.
- UDP services (such as Echo or Chargen) are enabled.

Modern systems usually disable these services, making Fraggle attacks uncommon today.

---

# Real-world Example

Victim:

```
10.0.0.5
```

Attacker sends:

```
Source IP

10.0.0.5

↓

Destination

192.168.1.255

↓

UDP Port 7 (Echo)
```

Suppose:

```
300 Devices
```

Each device sends a UDP reply.

```
300 Replies

↓

Victim
```

One spoofed packet generates hundreds of responses.

---

# Impact

Possible consequences:

- Network congestion.
- High bandwidth usage.
- Server unavailability.
- Increased latency.
- Denial of Service.
- Business interruption.

The attack mainly affects **Availability** in the **CIA Triad**.

---

# Detection

Signs of a Fraggle Attack:

- Large amount of UDP traffic.
- Many UDP replies from different devices.
- Bandwidth spikes.
- Firewall alerts.
- IDS/IPS detections.
- High CPU or network utilization.

Useful tools:

- Wireshark
- tcpdump
- NetFlow
- Firewall logs
- IDS/IPS

---

# Prevention

### 1. Disable Directed Broadcasts ✅

Routers should not forward packets sent to broadcast addresses.

---

### 2. Disable Unused UDP Services

Disable services such as:

- UDP Echo (Port 7)
- Chargen (Port 19)

if they are not needed.

---

### 3. Block IP Spoofing

Enable:

- Ingress Filtering
- Egress Filtering

---

### 4. Firewall Rules

Restrict unnecessary UDP traffic.

---

### 5. DDoS Protection

Use:

- CDN
- Cloudflare
- AWS Shield
- Azure DDoS Protection
- Google Cloud Armor

---

### 6. IDS/IPS

Monitor and block abnormal UDP amplification traffic.

---

# Spring Boot Protection

Spring Boot cannot prevent a Fraggle Attack because it happens before application traffic reaches the server.

Applications should:

- Run behind a reverse proxy.
- Use load balancers.
- Enable health checks.
- Support horizontal scaling.
- Monitor availability.

Example:

```
Internet

↓

Firewall

↓

Load Balancer

↓

Spring Boot Application
```

---

# Infrastructure Protection

Enterprise environments should implement:

- Disable directed broadcasts.
- Disable unused UDP services.
- Firewalls.
- IDS/IPS.
- Ingress/Egress filtering.
- CDN.
- DDoS protection.
- Load balancers.
- Continuous monitoring.

Example:

```
Internet

↓

CDN

↓

DDoS Protection

↓

Firewall

↓

Load Balancer

↓

Spring Boot Servers
```

---

# Best Practices

- Disable UDP Echo and Chargen services.
- Disable directed broadcasts.
- Enable ingress and egress filtering.
- Restrict unnecessary UDP traffic.
- Deploy IDS/IPS.
- Use DDoS mitigation services.
- Monitor network traffic.
- Keep routers and firewalls updated.

---

# Interview Questions

### 1. What is a Fraggle Attack?

A UDP-based amplification DDoS attack that sends spoofed UDP packets to a broadcast address, causing many devices to reply to the victim.

---

### 2. Which protocol does a Fraggle Attack use?

UDP.

---

### 3. Which OSI layer does Fraggle Attack target?

Primarily Layer 4 (Transport Layer), while also relying on Layer 3 for IP spoofing.

---

### 4. Why is Fraggle called an amplification attack?

Because one spoofed UDP request generates replies from many devices, multiplying the attack traffic.

---

### 5. What services were commonly abused in Fraggle attacks?

- UDP Echo (Port 7)
- Chargen (Port 19)

---

### 6. What is the best defense against a Fraggle Attack?

- Disable directed broadcasts.
- Disable unused UDP services.
- Enable ingress and egress filtering.

---

### 7. Can HTTPS stop a Fraggle Attack?

No.

The attack occurs at the network and transport layers before HTTP or HTTPS traffic is processed.

---

### 8. Can Spring Boot prevent a Fraggle Attack?

No.

Protection must be implemented by the network infrastructure.

---

### 9. Why are Fraggle Attacks rare today?

Modern routers disable directed broadcasts, and most systems no longer enable vulnerable UDP services like Echo and Chargen by default.

---

### 10. How is Fraggle different from Smurf Attack?

| Fraggle Attack | Smurf Attack |
|----------------|--------------|
| Uses UDP | Uses ICMP |
| Often targets UDP Echo/Chargen | Uses ICMP Echo Request (Ping) |
| Layer 4 attack (with Layer 3 spoofing) | Layer 3 attack |
| UDP amplification | ICMP amplification |

---

# References

- [[UDP]]
- [[IP]]
- [[IP Spoofing]]
- [[Smurf Attack]]
- [[OSI Model]]
- [[TCP-IP]]
- [[Firewall]]
- [[Routing]]
- [[Load Balancer]]
- [[Reverse Proxy]]
- [[DDoS]]