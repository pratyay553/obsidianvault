# Routing Attack

## Definition

A **Routing Attack** is an attack where an attacker **manipulates routing information** to redirect, intercept, or disrupt network traffic.

Instead of taking the correct path, packets follow a malicious or incorrect route controlled by the attacker.

> **Simple Definition**
>
> "A Routing Attack changes the path of network traffic to steal, monitor, or block communication."

---

# How it Works

### Normal Routing

```
Client

↓

Router A

↓

Router B

↓

Server
```

Packets follow the correct path.

---

### During a Routing Attack

The attacker advertises a fake or incorrect route.

```
Client

↓

Router A

↓

Attacker Router

↓

Server
```

or

```
Client

↓

Attacker

✖

Packets Dropped
```

The attacker can:

- Read traffic
- Modify traffic
- Delay traffic
- Drop packets completely

---

# Attack Flow

```
Attacker

↓

Fake Routing Information

↓

Routers Update Routes

↓

Traffic Redirected

↓

Attacker

↓

Server
```

---

# Target Layer

**OSI Layer:** **Layer 3 (Network Layer)**

Reason:

- Routing decisions are made using **IP addresses**.
- The attack targets routing protocols and routing tables.

---

# Prerequisites

For a Routing Attack:

- Access to the routing infrastructure or protocol.
- Weak authentication on routing protocols.
- Misconfigured routers.
- Ability to advertise fake routes.

Common targets:

- Enterprise networks
- ISP networks
- Data centers
- Cloud environments

---

# Real-world Example

Suppose the normal route is:

```
User

↓

Router A

↓

Router B

↓

Company Website
```

The attacker injects a fake route.

Now traffic becomes:

```
User

↓

Router A

↓

Attacker

↓

Company Website
```

The attacker can inspect or modify packets before forwarding them.

---

# Common Types of Routing Attacks

### 1. Route Injection

The attacker advertises fake network routes.

---

### 2. Route Hijacking

Traffic intended for one network is redirected to another.

---

### 3. BGP Hijacking

False **BGP** route advertisements redirect Internet traffic.

---

### 4. RIP Spoofing

Fake **RIP** updates alter routing tables.

---

### 5. OSPF Spoofing

Malicious **OSPF** messages change internal routing.

---

# Impact

Possible consequences:

- Man-in-the-Middle (MITM).
- Traffic interception.
- Data theft.
- Denial of Service.
- Increased latency.
- Network outages.
- Website unavailability.

Large routing attacks can affect entire Internet regions.

---

# Detection

Signs of a Routing Attack:

- Unexpected routing table changes.
- Traffic taking unusual paths.
- High latency.
- Packet loss.
- Unexpected next-hop addresses.
- IDS/IPS alerts.

Useful tools:

- `traceroute`
- `tracert`
- `ping`
- Router logs
- NetFlow
- BGP monitoring tools
- IDS/IPS

Example:

```bash
traceroute example.com
```

Unexpected hops may indicate a routing issue, though they are **not conclusive proof** of an attack.

---

# Prevention

### 1. Authenticate Routing Protocols

Use authentication for:

- OSPF
- RIP
- BGP (where supported)

---

### 2. Route Filtering

Reject unauthorized route advertisements.

---

### 3. Prefix Validation

Verify that advertised IP prefixes belong to legitimate networks.

---

### 4. Secure Router Configuration

- Disable unused routing protocols.
- Restrict management access.
- Apply firmware updates.

---

### 5. Network Monitoring

Continuously monitor routing changes and traffic patterns.

---

### 6. Infrastructure Security

Use:

- Firewalls
- IDS/IPS
- Network segmentation
- Secure management networks

---

# Spring Boot Protection

Spring Boot cannot prevent Routing Attacks because routing decisions occur before requests reach the application.

Applications should:

- Use HTTPS/TLS.
- Validate certificates.
- Use secure authentication.
- Enable logging and monitoring.
- Implement retries and health checks for service communication.

Even if traffic is redirected, TLS helps protect data confidentiality and integrity.

---

# Infrastructure Protection

Enterprise environments should implement:

- Secure routing protocol authentication.
- Route filtering.
- Firewalls.
- IDS/IPS.
- Network segmentation.
- VPNs for remote connections.
- Continuous route monitoring.
- High-availability routing.

Example:

```
Internet

↓

Edge Router

↓

Firewall

↓

Core Router

↓

Load Balancer

↓

Spring Boot Application
```

---

# Best Practices

- Authenticate routing protocol updates.
- Monitor routing tables regularly.
- Restrict router management access.
- Use encrypted management protocols (SSH instead of Telnet).
- Deploy IDS/IPS.
- Enable HTTPS/TLS for applications.
- Keep router firmware updated.
- Audit routing configurations periodically.

---

# Interview Questions

### 1. What is a Routing Attack?

A Routing Attack manipulates routing information to redirect, intercept, or disrupt network traffic.

---

### 2. Which OSI layer does a Routing Attack target?

Layer 3 (Network Layer).

---

### 3. What is Route Hijacking?

Redirecting traffic by advertising unauthorized routes.

---

### 4. What is BGP Hijacking?

An attack where false BGP route advertisements redirect Internet traffic through unauthorized networks.

---

### 5. What are common routing protocols affected?

- BGP
- OSPF
- RIP

---

### 6. Can HTTPS prevent Routing Attacks?

No.

HTTPS cannot stop route manipulation, but it encrypts traffic and helps prevent attackers from reading or modifying application data.

---

### 7. Can Spring Boot prevent Routing Attacks?

No.

The attack occurs in the network infrastructure before traffic reaches the application.

---

### 8. How can enterprises prevent Routing Attacks?

Using:

- Routing protocol authentication
- Route filtering
- Prefix validation
- IDS/IPS
- Secure router configurations
- Continuous monitoring

---

### 9. Why is routing authentication important?

It prevents unauthorized devices from advertising fake routing information.

---

### 10. How is a Routing Attack different from IP Spoofing?

| Routing Attack | IP Spoofing |
|----------------|-------------|
| Changes the path of network traffic | Forges the source IP address |
| Targets routing tables and protocols | Targets the IP packet header |
| Can redirect traffic | Hides or impersonates the sender |
| Affects network infrastructure | Affects packet identity |

---

# References

- [[Routing]]
- [[IP]]
- [[TCP-IP]]
- [[BGP]]
- [[OSPF]]
- [[RIP]]
- [[IP Spoofing]]
- [[OSI Model]]
- [[Firewall]]
- [[HTTPS]]
- [[TLS]]
- [[VPN]]
- [[Man-in-the-Middle (MITM)]]