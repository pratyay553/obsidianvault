```markdown
# Network Reconnaissance

## Definition

Network Reconnaissance is the process of collecting information about a target network, systems, and services before launching an attack.

Attackers use reconnaissance to understand:

- Network structure.
- Active devices.
- Open ports.
- Running services.
- Operating systems.
- Possible vulnerabilities.

Security teams also use reconnaissance for:

- Network auditing.
- Penetration testing.
- Security assessment.

> **Simple Definition**
>
> "Network Reconnaissance is the information-gathering phase where someone studies a network to find possible entry points."

---

# How it Works

Network reconnaissance usually happens in phases.

---

# Phase 1: Passive Reconnaissance

The attacker collects information without directly interacting with the target.

Sources:

- Public websites.
- DNS records.
- Search engines.
- Social media.
- Certificate information.
- Public IP information.

Example:

```

Company Domain

↓

Public Information

↓

Network Details

```

The target may not know this activity is happening.

---

# Phase 2: Active Reconnaissance

The attacker directly communicates with the target network.

Activities:

- Ping scanning.
- Port scanning.
- Service discovery.
- OS detection.

Example:

```

Attacker

↓

Network Scan

↓

Target Server

```

---

# Common Reconnaissance Techniques

## 1. Host Discovery

Finding active systems.

Example:

```

Which machines are alive?

```

---

## 2. Port Scanning

Finding open ports.

Example:

```

80 HTTP

443 HTTPS

22 SSH

```

---

## 3. Service Enumeration

Identifying running services.

Example:

```

Port 8080

↓

Apache Tomcat

```

---

## 4. DNS Enumeration

Finding DNS information:

- Subdomains.
- Mail servers.
- IP addresses.

---

## 5. Network Mapping

Creating a picture of:

```

Internet

↓

Firewall

↓

Servers

↓

Applications

```

---

# Attack Flow

```

Attacker

↓

Information Gathering

↓

Network Mapping

↓

Find Weak Points

↓

Launch Attack

```

Reconnaissance is usually the first step in the cyber attack lifecycle.

---

# Target Layer

Network Reconnaissance can involve multiple layers.

## Layer 2 (Data Link)

Information:

- MAC addresses.
- Local network devices.

---

## Layer 3 (Network)

Information:

- IP addresses.
- Routing paths.

---

## Layer 4 (Transport)

Information:

- Open TCP/UDP ports.

---

## Layer 7 (Application)

Information:

- Web servers.
- APIs.
- Application technologies.

---

# Prerequisites

For attackers:

- Network visibility.
- Internet access.
- Scanning tools.
- Knowledge of networking.

Common tools:

- Nmap.
- Wireshark.
- DNS tools.
- Network scanners.

For authorized security testing:

- Written permission.
- Defined scope.
- Testing agreement.

---

# Real-world Example

An attacker targets an e-commerce company.

Reconnaissance finds:

```

Domain:

shop.example.com

Open Ports:

443 HTTPS

8080 Application Server

Technology:

Spring Boot API

```


The attacker now knows:

- Where the application exists.
- Which technologies are used.
- Possible attack areas.

---

# Impact

Reconnaissance itself may not damage systems, but it helps attackers prepare attacks.

Possible risks:

- Identifying vulnerable services.
- Discovering exposed databases.
- Finding weak configurations.
- Targeted phishing.
- Future exploitation.

---

# Detection

Reconnaissance detection is difficult because many activities look like normal traffic.

Possible indicators:

- Large number of connection attempts.
- Port scanning patterns.
- Repeated DNS queries.
- Unusual traffic from unknown sources.

Tools:

- IDS/IPS.
- SIEM.
- Firewall logs.
- Network monitoring.

Examples:

```

1000 ports scanned

from one IP

within minutes

```

---

# Prevention

## 1. Reduce Information Exposure

Avoid exposing:

- Server versions.
- Internal IPs.
- Debug endpoints.

---

## 2. Firewall Rules

Allow only required ports.

Example:

```

Open:

443 HTTPS

Blocked:

22 SSH from Internet

```

---

## 3. Network Segmentation

Separate:

- Public servers.
- Application servers.
- Databases.

Example:

```

Internet

↓

DMZ

↓

Application Network

↓

Database Network

```

---

## 4. Disable Unnecessary Services

Every open service increases attack surface.

---

## 5. Monitoring

Use:

- IDS.
- IPS.
- SIEM.
- Log analysis.

---

# Spring Boot Protection

Spring Boot applications should reduce information leakage.

## 1. Hide Application Details

Avoid exposing:

- Framework versions.
- Stack traces.
- Debug information.

Example:

Disable:

```

/actuator/env

/actuator/heapdump

```

in production.

---

## 2. Secure Actuator Endpoints

Use:

- Authentication.
- Authorization.
- Network restrictions.

---

## 3. Secure APIs

Implement:

- Spring Security.
- Authentication.
- Authorization.
- Rate limiting.

---

## 4. API Gateway Protection

Architecture:

```

Client

↓

API Gateway

↓

Spring Boot Services

```

The gateway hides internal services.

---

## 5. Logging and Monitoring

Monitor:

- Failed requests.
- Suspicious scans.
- Abnormal API usage.

---

# Infrastructure Protection

Enterprise security architecture:

```

Internet

↓

CDN

↓

WAF

↓

Firewall

↓

API Gateway

↓

Load Balancer

↓

Spring Boot Services

↓

Database

```

Security controls:

- Firewalls.
- IDS/IPS.
- Network segmentation.
- Vulnerability scanning.
- SIEM monitoring.
- Zero Trust architecture.

---

# Best Practices

- Minimize exposed services.
- Keep systems patched.
- Use firewalls.
- Monitor network traffic.
- Disable unnecessary ports.
- Protect DNS information.
- Hide server technology details.
- Use API gateways.
- Perform regular security assessments.
- Maintain asset inventory.
- Follow least privilege access.

---

# Interview Questions

### 1. What is Network Reconnaissance?

Network Reconnaissance is the process of gathering information about networks, systems, and services before an attack.

---

### 2. Why do attackers perform reconnaissance?

To identify:

- Targets.
- Open ports.
- Vulnerable services.
- Network architecture.

---

### 3. What are the two types of reconnaissance?

1. Passive Reconnaissance.
2. Active Reconnaissance.

---

### 4. What is passive reconnaissance?

Collecting information without directly interacting with the target.

Example:

- DNS lookup.
- Public information gathering.

---

### 5. What is active reconnaissance?

Directly interacting with the target.

Example:

- Port scanning.
- Ping scanning.

---

### 6. What tools are commonly used for reconnaissance?

Examples:

- Nmap.
- Wireshark.
- DNS tools.
- Network scanners.

---

### 7. Is reconnaissance always illegal?

No.

Authorized security professionals perform reconnaissance during penetration testing with permission.

---

### 8. How can organizations reduce reconnaissance risk?

Using:

- Firewalls.
- Network segmentation.
- Monitoring.
- Minimal information exposure.
- Secure configurations.

---

### 9. Can Spring Boot stop network reconnaissance?

No.

But it can reduce information leakage through secure configuration and authentication.

---

### 10. Why is reconnaissance important in cybersecurity?

Because attackers usually gather information before exploitation, and defenders can detect and reduce attack surfaces during this stage.

---

# References

- [[OSI Model]]
- [[TCP-IP]]
- [[IP]]
- [[DNS]]
- [[DNS Resolution]]
- [[Ports]]
- [[Routing]]
- [[Firewall]]
- [[NAT]]
- [[VLAN]]
- [[Network Scanning]]
- [[Nmap]]
- [[IDS]]
- [[IPS]]
- [[Spring Boot Security]]
- [[API Gateway]]
- [[Zero Trust Security]]
```