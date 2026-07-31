# Firewall

## Definition

A **Firewall** is a security system that monitors and controls **incoming and outgoing network traffic** based on predefined security rules.

It acts as a **security gate** between trusted and untrusted networks.

> **Simple Definition**
>
> "A firewall decides which network traffic is allowed and which traffic should be blocked."

Example:

```
Internet

   ↓

Firewall

   ↓

Application Server

   ↓

Database
```

---

## Why it Matters

Every application connected to a network faces security risks.

A firewall helps protect against:

- Unauthorized access.
- Port scanning.
- Malicious traffic.
- Data exposure.
- Network attacks.

Example:

Without firewall:

```
Attacker

↓

Database Server
```

With firewall:

```
Attacker

↓

Firewall ❌ Block

↓

Database Server
```

---

## How it Works

A firewall checks network traffic using rules.

Each connection contains information like:

```
Source IP

Destination IP

Port Number

Protocol

Traffic Type
```

The firewall compares this information with security rules.

---

## Firewall Decision Process

Example request:

```
Client:

IP: 10.10.1.20

Port: 443

Protocol: HTTPS
```

Firewall checks:

```
Is this IP allowed?

Is this port open?

Is this protocol allowed?
```

Decision:

```
ALLOW ✅

or

DENY ❌
```

---

# Firewall Rules Example

Allow HTTPS:

```
Allow

Port: 443

Protocol: TCP
```

Block Telnet:

```
Deny

Port: 23
```

Allow database only from application server:

```
Allow

Source:

Application Server IP


Destination:

Database Port 5432
```

---

# Types of Firewalls

## 1. Packet Filtering Firewall

Works mainly at:

```
Layer 3 and Layer 4
```

Checks:

- IP address
- Port
- Protocol

Example:

```
Allow TCP 443

Block TCP 23
```

Fast but basic.

---

## 2. Stateful Firewall

Tracks active connections.

Example:

Client starts:

```
Connection Request
```

Firewall remembers:

```
This connection is allowed.
```

Response traffic is automatically permitted.

More secure than packet filtering.

---

## 3. Application Firewall

Works at:

```
Layer 7
```

Understands application traffic.

Examples:

- HTTP requests
- API calls
- SQL patterns

Can block:

```
SQL Injection

XSS attacks

Malicious API requests
```

Example:

```
GET /users?id=' OR 1=1
```

Blocked.

---

## 4. Next Generation Firewall (NGFW)

Advanced firewall with:

- Deep packet inspection.
- Intrusion prevention.
- Malware detection.
- Application awareness.

---

# Firewall Placement in Enterprise Architecture

Common production design:

```
Internet

↓

DDoS Protection

↓

Load Balancer

↓

Web Application Firewall

↓

Application Firewall

↓

Spring Boot Services

↓

Database Firewall

↓

Database
```

---

# Firewall vs WAF

| Firewall | WAF |
|-|-|
| Network security | Web application security |
| Layer 3/4 mainly | Layer 7 |
| Controls IP/Port | Controls HTTP traffic |
| Protects network | Protects web applications |

Example:

Firewall blocks:

```
Port 22 from unknown IP
```

WAF blocks:

```
SQL Injection request
```

---

# Firewall Rules Best Practices

## 1. Default Deny

Allow only required traffic.

Example:

Bad:

```
Allow Everything
```

Good:

```
Allow HTTPS only

Block everything else
```

---

## 2. Least Privilege

Give minimum required access.

Example:

Database:

```
Only Application Server can connect.
```

Not:

```
Entire Internet
```

---

## 3. Regular Rule Review

Remove:

- Old rules.
- Unused ports.
- Temporary access.

---

## 4. Logging

Record:

- Allowed traffic.
- Blocked traffic.
- Suspicious activity.

---

# Common Ports

| Service | Port |
|-|-|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| FTP | 21 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| Redis | 6379 |

---

# Common Firewall Attacks

## Port Scanning

Attacker checks open ports.

Example:

```
Which ports are open?
```

Firewall blocks unnecessary ports.

---

## Unauthorized Access

Example:

```
Internet

↓

Database Port
```

Firewall blocks direct access.

---

## Brute Force

Example:

```
10000 login attempts
```

Firewall + security systems can detect and block it.

---

## Common Pitfalls

### ❌ Firewall alone provides complete security.

Wrong.

Firewall is only one security layer.

You also need:

- Authentication.
- Encryption.
- Application security.
- Monitoring.

---

### ❌ Opening all ports is okay.

Wrong.

Every open port increases attack surface.

---

### ❌ Internal networks don't need firewalls.

Wrong.

Modern systems use:

```
Zero Trust Architecture
```

Even internal traffic should be controlled.

---

### ❌ Firewall blocks all attacks.

Wrong.

A firewall cannot detect every application vulnerability.

Example:

SQL Injection inside allowed HTTPS traffic may bypass a normal firewall.

---

### ❌ Firewall and WAF are the same.

Wrong.

Firewall protects network traffic.

WAF protects web application traffic.

---

# Spring Boot / Java Notes

In production, Spring Boot applications are usually protected by multiple firewalls.

Example:

```
Internet

↓

Cloud Firewall

↓

Load Balancer

↓

Application Firewall

↓

Spring Boot Server

↓

Database Firewall

↓

Database
```

---

# Example: Allow Spring Boot API Traffic

Spring Boot runs:

```
Port 8080
```

Firewall rule:

```
Allow

Source:

Load Balancer


Destination:

Spring Boot Server

Port:

8080
```

Direct Internet access:

```
Internet → 8080

Blocked ❌
```

---

# Database Security Example

Bad architecture:

```
Internet

↓

Database
```

Firewall should block this.

---

Good architecture:

```
Internet

↓

Spring Boot Application

↓

Database

Port 5432
```

Firewall rule:

```
Allow only:

Spring Boot Server → Database
```

---

# Cloud Firewall Examples

AWS:

- Security Groups
- Network ACL

Azure:

- Network Security Group

Google Cloud:

- VPC Firewall Rules

---

# Debugging Firewall Issues

## Connection Refused

Check:

- Application running?
- Port open?
- Firewall rule?

---

## Connection Timeout

Usually:

- Firewall blocking.
- Network route issue.

---

## Cannot Connect Database

Check:

```
Application IP

↓

Firewall Rule

↓

Database Port

↓

Database Status
```

---

# Quick Revision

- Firewall controls network traffic.
- Works mainly at Layer 3/4.
- Advanced firewalls work at Layer 7.
- Uses rules based on IP, port, and protocol.
- Default deny is the best practice.
- Do not expose databases directly.
- Firewall is one layer of security, not complete protection.
- WAF protects HTTP applications.
- Cloud environments use virtual firewalls.

---

# Interview Questions

### 1. What is a firewall?

A security system that filters network traffic based on predefined rules.

---

### 2. Why do we need a firewall?

To prevent unauthorized access and protect systems from malicious network traffic.

---

### 3. At which OSI layer does a firewall work?

Traditional firewalls work at Layer 3 and Layer 4. Modern firewalls can work at Layer 7.

---

### 4. Difference between firewall and WAF?

Firewall protects network traffic.

WAF protects web applications and HTTP requests.

---

### 5. What is a firewall rule?

A rule that defines whether specific traffic should be allowed or blocked.

---

### 6. Why should we avoid opening all ports?

Because every open port increases the attack surface.

---

### 7. How do you secure a database using a firewall?

Allow database access only from trusted application servers and block public access.

---

### 8. What is default deny?

Block all traffic by default and allow only required traffic.

---

### 9. Can firewall prevent SQL Injection?

A normal firewall cannot. A WAF or application-level security can detect SQL Injection patterns.

---

### 10. Why should Spring Boot developers understand firewalls?

Because deployment, API access, database connectivity, cloud security, and troubleshooting depend on firewall configuration.

---

## References

- [[OSI Model]]
- [[TCP-IP]]
- [[IP]]
- [[Ports]]
- [[NAT]]
- [[Proxy]]
- [[Reverse Proxy]]
- [[HTTPS]]
- [[TLS]]
- [[WAF]]