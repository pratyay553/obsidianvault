# Ports

## Definition

A **Port** is a logical communication endpoint used by applications to send and receive network data.

Think of an **IP address as a building address** and a **port as a room number** inside that building.

> **Simple Definition**
>
> "An IP address identifies a device, while a port identifies a specific application or service running on that device."

Example:

```
IP Address

192.168.1.10

↓

Port 8080

↓

Spring Boot Application
```

---

# Why it Matters

A single computer can run multiple applications at the same time.

Example:

```
Laptop

↓

Chrome

↓

VS Code

↓

Spring Boot

↓

MySQL
```

All these applications share the same IP address but use **different ports**.

Without ports, the operating system wouldn't know which application should receive incoming data.

---

# How it Works

When a client sends a request, it includes:

- Source IP
- Source Port
- Destination IP
- Destination Port

Example:

```
Client

IP: 192.168.1.5

Port: 50010

        |

        |

Server

IP: 192.168.1.10

Port: 8080
```

The operating system delivers the request to the application listening on **port 8080**.

---

# Client Port vs Server Port

## Client Port (Ephemeral Port)

Automatically assigned by the operating system.

Example:

```
Client

192.168.1.5:50021
```

These ports are temporary.

---

## Server Port

Fixed port where an application waits for requests.

Example:

```
Spring Boot

8080
```

```
MySQL

3306
```

```
HTTPS

443
```

---

# Port Range

| Port Range | Purpose |
|------------|----------|
| 0 – 1023 | Well-known ports |
| 1024 – 49151 | Registered ports |
| 49152 – 65535 | Dynamic (Ephemeral) ports |

---

# Common Well-Known Ports

| Port | Service |
|------|---------|
| 20/21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | Spring Boot / Tomcat (Default) |
| 9092 | Apache Kafka |
| 27017 | MongoDB |

---

# Port Communication Example

Opening:

```
https://example.com
```

Flow:

```
Browser

↓

DNS

↓

Server IP

↓

TCP Connection

↓

Port 443

↓

HTTPS Server
```

---

# Multiple Applications

One computer:

```
IP

192.168.1.10

|

|-- Port 80 → Nginx

|-- Port 443 → HTTPS

|-- Port 8080 → Spring Boot

|-- Port 3306 → MySQL

|-- Port 6379 → Redis
```

Every application has its own listening port.

---

# Port States

A port can be:

### Open

Application is listening.

```
Port 8080

↓

Spring Boot Running
```

---

### Closed

No application is using the port.

---

### Filtered

A firewall blocks access.

```
Internet

↓

Firewall ❌

↓

Port 22
```

---

# TCP vs UDP Ports

Some ports use TCP, others use UDP.

Examples:

| Protocol | Common Ports |
|----------|--------------|
| TCP | 80, 443, 22, 3306 |
| UDP | 53 (DNS), 67 (DHCP), 123 (NTP) |

Some services (like DNS) can use both TCP and UDP.

---

# Ports in Enterprise Architecture

```
Internet

↓

443 (HTTPS)

↓

Load Balancer

↓

8080

↓

Spring Boot

↓

5432

↓

PostgreSQL
```

Each component communicates through different ports.

---

# Ports and Firewalls

Firewall rules often allow or block ports.

Example:

Allow:

```
443
```

Block:

```
22
```

Database:

```
5432

Only Application Server
```

---

# Common Pitfalls

## ❌ Port and IP address are the same.

Wrong.

- IP identifies the **device**.
- Port identifies the **application**.

---

## ❌ One port can be used by multiple applications.

Wrong.

Normally, only one application can listen on the same port on the same IP.

---

## ❌ Opening every port is safe.

Wrong.

Every open port increases the attack surface.

Only required ports should be open.

---

## ❌ Port 8080 is the same as HTTP.

Wrong.

HTTP is a protocol.

Port **80** is the default HTTP port.

Port **8080** is commonly used for development but can also serve HTTP.

---

## ❌ Closed ports cause application bugs.

Not always.

Sometimes:

- Firewall blocks traffic.
- Wrong IP.
- Wrong configuration.

---

# Spring Boot / Java Notes

By default, Spring Boot runs on:

```
8080
```

Change the port:

```properties
server.port=9090
```

Now application runs on:

```
http://localhost:9090
```

---

# Multiple Spring Boot Applications

Example:

```
User Service

8081

Order Service

8082

Payment Service

8083
```

Each service needs a unique port.

---

# Embedded Tomcat

Spring Boot includes an embedded Tomcat server.

Tomcat listens on the configured port.

```
Client

↓

Port 8080

↓

Tomcat

↓

Spring Boot Controller
```

---

# Checking Open Ports

Windows:

```bash
netstat -ano
```

Linux/macOS:

```bash
netstat -tuln

# or

ss -tuln
```

---

# Common Production Issues

## Port Already in Use

Error:

```
Port 8080 already in use
```

Solution:

- Stop the application using the port.
- Use another port.

---

## Connection Refused

Possible causes:

- Application not running.
- Wrong port.
- Firewall blocked.

---

## Timeout

Possible causes:

- Firewall.
- Network issue.
- Server unavailable.

---

# Quick Revision

- A port identifies an application on a device.
- IP identifies the device.
- HTTP → Port 80.
- HTTPS → Port 443.
- Spring Boot → Port 8080 (default).
- MySQL → Port 3306.
- PostgreSQL → Port 5432.
- One application listens on one port.
- Firewalls commonly allow or block ports.

---

# Interview Questions

### 1. What is a port?

A port is a logical endpoint used by applications to communicate over a network.

---

### 2. What is the difference between an IP address and a port?

IP identifies a device, while a port identifies a specific application running on that device.

---

### 3. Why do we need ports?

Ports allow multiple applications to communicate over the same IP address.

---

### 4. What is the default port for HTTP and HTTPS?

- HTTP → 80
- HTTPS → 443

---

### 5. What is the default Spring Boot port?

8080.

---

### 6. Can two applications use the same port?

No, not on the same IP address at the same time.

---

### 7. What is an ephemeral port?

A temporary client-side port automatically assigned by the operating system.

---

### 8. Why might "Connection Refused" occur?

Because the application is not running, the wrong port is used, or a firewall blocks the connection.

---

### 9. What does a firewall do with ports?

It allows or blocks network traffic based on configured rules.

---

### 10. Why should a Spring Boot developer understand ports?

Because application deployment, debugging, API communication, databases, Docker, and cloud services all depend on correct port configuration.

---

# References

- [[TCP-IP]]
- [[TCP Handshake]]
- [[UDP]]
- [[IP]]
- [[HTTP]]
- [[HTTPS]]
- [[Firewall]]
- [[NAT]]
- [[CIDR]]
- [[Routing]]