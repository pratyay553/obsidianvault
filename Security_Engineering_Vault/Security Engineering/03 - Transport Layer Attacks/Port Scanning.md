# Port Scanning

## Definition

**Port Scanning** is the process of checking a computer or server to find **which network ports are open, closed, or filtered**.

Attackers use port scanning to discover running services and identify possible entry points. System administrators also use it to audit and secure networks.

> **Simple Definition**
>
> "Port Scanning is checking which doors (ports) of a server are open."

Example:

```
Server

↓

Port 22   → Open (SSH)

Port 80   → Open (HTTP)

Port 443  → Open (HTTPS)

Port 3306 → Closed
```

---

# How it Works

Every service listens on a specific port.

Example:

```
Internet

↓

Server

↓

Port 80

↓

Web Server
```

A scanner sends requests to many ports.

Example:

```
Scanner

↓

Port 20

↓

Closed

↓

Port 21

↓

Closed

↓

Port 22

↓

Open

↓

Port 80

↓

Open

↓

Port 443

↓

Open
```

The scanner records the response and builds a list of available services.

---

# Common Types of Port Scanning

## 1. TCP Connect Scan

Completes the full TCP handshake.

```
SYN

↓

SYN + ACK

↓

ACK
```

Easy to detect.

---

## 2. TCP SYN Scan (Half-Open Scan)

Only sends the SYN packet.

```
SYN

↓

SYN + ACK

↓

RST
```

Faster and less noisy than a full TCP Connect Scan.

---

## 3. UDP Scan

Checks UDP ports.

Example:

```
DNS

UDP 53
```

---

## 4. Service Version Detection

Attempts to identify:

- Web server
- Database
- SSH version
- Operating system

---

# Attack Flow

```
Attacker

↓

Port Scan

↓

Open Ports Found

↓

Running Services Identified

↓

Possible Vulnerabilities
```

Port scanning is usually the **first step** before an attack.

---

# Target Layer

**Primary OSI Layer:** Layer 4 (Transport Layer)

Reason:

- Ports belong to TCP and UDP.
- TCP and UDP operate at the Transport Layer.

It also involves:

- Layer 3 → IP addressing.
- Layer 7 → Identifying application services.

---

# Prerequisites

For Port Scanning:

- Target IP address.
- Network connectivity.
- Scanning tool.

Common tools:

- Nmap
- Masscan
- Angry IP Scanner

Security professionals use these tools with proper authorization.

---

# Real-world Example

A company hosts a Spring Boot application.

Attacker scans:

```
203.0.113.10
```

Results:

```
22    SSH

80    HTTP

443   HTTPS

8080  Spring Boot
```

The attacker now knows:

- SSH is enabled.
- Web server is available.
- Spring Boot is running.

---

# Impact

Port scanning itself does **not damage** a system.

However, it helps attackers:

- Discover open services.
- Find outdated software.
- Identify attack targets.
- Plan future attacks.

Open unnecessary ports increase the attack surface.

---

# Detection

Signs of Port Scanning:

- Many connection attempts to different ports.
- Sequential port access.
- Repeated failed connections.
- Firewall alerts.

Useful tools:

- Firewall logs.
- IDS/IPS.
- SIEM.
- Network monitoring.

Example:

```
One IP

↓

1000 port requests

↓

Within 30 seconds
```

This is suspicious.

---

# Prevention

## 1. Close Unused Ports ✅

Only expose required services.

Example:

```
Open:

443 HTTPS


Closed:

22 SSH (Public Internet)

3306 MySQL
```

---

## 2. Firewall Rules

Allow only trusted traffic.

Example:

```
Internet

↓

Firewall

↓

Only Port 443 Allowed
```

---

## 3. Network Segmentation

Separate:

- Public servers.
- Internal servers.
- Databases.

---

## 4. IDS/IPS

Detect repeated scanning behavior.

---

## 5. Rate Limiting

Limit repeated connection attempts from the same IP.

---

# Spring Boot Protection

Spring Boot cannot prevent port scanning because scanning occurs before requests reach the application.

However, you should:

## Hide Unnecessary Services

Do not expose:

- Development ports.
- Debug ports.
- Management ports.

Example:

```
Internet

↓

443

↓

Spring Boot Application
```

Avoid exposing:

```
8080

9090

JMX

Debug Ports
```

---

## Secure Actuator

Protect Actuator endpoints using:

- Authentication.
- Authorization.
- IP restrictions.

---

## Use Reverse Proxy

Example:

```
Internet

↓

Nginx

↓

Spring Boot
```

Expose only the reverse proxy to the Internet.

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

Reverse Proxy

↓

Spring Boot Services

↓

Database
```

Security controls:

- Firewalls.
- IDS/IPS.
- Network segmentation.
- VPN for administration.
- Continuous monitoring.

---

# Best Practices

- Close unused ports.
- Expose only required services.
- Use firewalls.
- Monitor connection attempts.
- Disable unnecessary services.
- Hide server information.
- Use reverse proxies.
- Restrict SSH access.
- Perform regular vulnerability scans.
- Follow the principle of least privilege.

---

# Interview Questions

### 1. What is Port Scanning?

Port Scanning is the process of identifying open, closed, and filtered ports on a system.

---

### 2. Which OSI layer does Port Scanning target?

Primarily Layer 4 (Transport Layer).

---

### 3. Is Port Scanning always illegal?

No.

System administrators and penetration testers perform authorized port scanning for security assessments.

---

### 4. What is the purpose of Port Scanning?

To discover:

- Open ports.
- Running services.
- Potential vulnerabilities.

---

### 5. What is the difference between TCP Connect Scan and TCP SYN Scan?

| TCP Connect Scan | TCP SYN Scan |
|------------------|--------------|
| Completes full handshake | Stops before completing handshake |
| Easier to detect | Faster and less noisy |
| Establishes a connection | Creates a half-open connection |

---

### 6. Can Spring Boot stop Port Scanning?

No.

Port scanning happens before requests reach the application.

---

### 7. How do organizations prevent attackers from finding services?

By:

- Closing unused ports.
- Using firewalls.
- Restricting access.
- Using reverse proxies.

---

### 8. Why should databases not be exposed to the Internet?

Because they increase the attack surface and may expose sensitive data if compromised.

---

### 9. What is a filtered port?

A **filtered** port is protected by a firewall or network device, so the scanner cannot determine whether it is open or closed.

---

### 10. Which tool is most commonly used for Port Scanning?

**Nmap** is the most widely used tool for network discovery and port scanning.

---

# References

- [[Ports]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[UDP]]
- [[Firewall]]
- [[Network Reconnaissance]]
- [[Packet Sniffing]]
- [[Reverse Proxy]]
- [[Load Balancer]]
- [[Spring Boot Security]]
- [[OSI Model]]