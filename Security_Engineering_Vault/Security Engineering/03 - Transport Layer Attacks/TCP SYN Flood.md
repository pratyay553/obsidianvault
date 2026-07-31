```markdown
# TCP SYN Flood

## Definition

A **TCP SYN Flood** is a **Denial-of-Service (DoS)** attack where an attacker sends a large number of **TCP SYN requests** to a server but does not complete the TCP handshake.

The server keeps waiting for connections that never finish, consuming resources.

> **Simple Definition**
>
> "TCP SYN Flood fills a server's connection queue with incomplete TCP connection requests."

---

# How it Works

## Normal TCP Three-Way Handshake

TCP establishes a reliable connection using three steps.

```

Client Server

SYN --------------------->

```
    <--------------------- SYN + ACK
```

ACK --------------------->

Connection Established

```

After this:

```

Client ↔ Server

```

communication begins.

---

# During TCP SYN Flood

The attacker sends many SYN requests.

```

Attacker

SYN

SYN

SYN

SYN

↓

Server

```

Server responds:

```

SYN + ACK

↓

Waiting for ACK

```

But the attacker never sends the final ACK.

---

# Half-Open Connection State

The server stores incomplete connections:

```

Connection Table

---

SYN Received

SYN Received

SYN Received

SYN Received

---

```

Eventually:

```

Connection Queue Full

↓

New Users Cannot Connect

```

---

# Attack Flow

```

Attacker

↓

Many SYN Requests

↓

Server Allocates Resources

↓

No Final ACK

↓

Connection Queue Exhausted

↓

Service Unavailable

```

---

# Target Layer

**OSI Layer:** Layer 4 (Transport Layer)

Reason:

- Uses TCP protocol.
- Exploits TCP connection establishment.

---

# Prerequisites

For a TCP SYN Flood:

- Ability to send large numbers of TCP SYN packets.
- Network bandwidth.
- Target service running TCP.

Common targets:

- Web servers.
- API servers.
- Load balancers.
- Database services.
- VPN gateways.

Large attacks often use:

- Botnets.
- Distributed systems.
- Spoofed IP addresses.

---

# Real-world Example

An e-commerce website normally handles:

```

10,000 TCP connections

```

During SYN Flood:

```

Millions of SYN Requests

```

Server state:

```

Waiting Connections:

999999

```

Result:

```

Real Customer

↓

Connection Refused

↓

Website Unavailable

````

---

# Impact

Possible consequences:

- Website downtime.
- Connection failures.
- High memory usage.
- CPU exhaustion.
- Slow response times.
- Service interruption.

The attack affects:

**Availability** in the CIA Triad.

---

# Detection

Signs of TCP SYN Flood:

- Large number of SYN packets.
- Very few completed connections.
- Many connections in SYN_RECEIVED state.
- Increased network traffic.
- High server resource usage.

Useful commands:

Linux:

```bash
netstat -an | grep SYN_RECV
````

or:

```bash
ss -s
```

Network tools:

- Wireshark.
    
- tcpdump.
    
- IDS/IPS.
    
- Firewall logs.
    
- NetFlow monitoring.
    

---

# Prevention

## 1. SYN Cookies ✅

SYN Cookies allow servers to handle SYN requests without storing connection information immediately.

Flow:

```
Client

↓

SYN

↓

Server Generates Cookie

↓

ACK Received

↓

Connection Created
```

---

## 2. Increase Connection Queue Size

Increase allowed pending connections.

Example:

```
More Queue Capacity

↓

More Resistance
```

---

## 3. Rate Limiting

Limit incoming SYN requests.

Example:

```
Maximum:

500 SYN/sec
```

---

## 4. Stateful Firewall

Track connection states and block abnormal traffic.

---

## 5. Load Balancer

Distribute traffic across multiple servers.

---

## 6. DDoS Protection

Use:

- Cloudflare.
    
- AWS Shield.
    
- Azure DDoS Protection.
    
- Google Cloud Armor.
    

---

# Spring Boot Protection

Spring Boot does not directly prevent TCP SYN Flood because the attack happens before HTTP requests reach the application.

However, production architecture should include:

## 1. Load Balancer

```
Client

↓

Load Balancer

↓

Spring Boot Application
```

The load balancer absorbs connection pressure.

---

## 2. Connection Management

Configure:

- Server connection limits.
    
- Timeouts.
    
- Thread pools.
    

Example:

```properties
server.tomcat.connection-timeout=5000
```

---

## 3. Application Monitoring

Monitor:

- Request latency.
    
- Connection failures.
    
- Resource usage.
    

---

## 4. Use API Gateway

Architecture:

```
Client

↓

API Gateway

↓

Spring Boot Services
```

Benefits:

- Rate limiting.
    
- Filtering.
    
- Authentication.
    

---

# Infrastructure Protection

Enterprise architecture:

```
Internet

↓

DDoS Protection

↓

Firewall

↓

Load Balancer

↓

API Gateway

↓

Spring Boot Application

↓

Database
```

Security controls:

- SYN flood protection.
    
- SYN cookies.
    
- Rate limiting.
    
- IDS/IPS.
    
- Firewall rules.
    
- Traffic monitoring.
    
- Auto scaling.
    

---

# Best Practices

- Enable SYN cookies.
    
- Configure firewall SYN protection.
    
- Use load balancers.
    
- Deploy DDoS protection.
    
- Monitor TCP connection states.
    
- Configure proper timeouts.
    
- Limit exposed services.
    
- Keep network devices updated.
    
- Perform load testing.
    
- Maintain incident response plans.
    

---

# Interview Questions

### 1. What is TCP SYN Flood?

A DoS attack where attackers send many SYN requests without completing the TCP handshake.

---

### 2. Which OSI layer does TCP SYN Flood target?

Layer 4 (Transport Layer).

---

### 3. What part of TCP does SYN Flood exploit?

The TCP three-way handshake.

---

### 4. Why does SYN Flood consume server resources?

Because the server allocates memory and connection entries for incomplete connections.

---

### 5. What is a half-open connection?

A TCP connection where SYN is received but the final ACK has not arrived.

---

### 6. What are SYN Cookies?

A defense mechanism where the server avoids storing connection state until the client proves it received the SYN response.

---

### 7. Can HTTPS prevent SYN Flood?

No.

HTTPS works at Layer 7, but SYN Flood happens at Layer 4 before HTTPS communication starts.

---

### 8. Can Spring Boot prevent TCP SYN Flood?

No.

Protection must happen at:

- Firewall.
    
- Load balancer.
    
- Network layer.
    

---

### 9. Difference between SYN Flood and HTTP Flood?

|TCP SYN Flood|HTTP Flood|
|---|---|
|Layer 4 attack|Layer 7 attack|
|Targets TCP handshake|Targets HTTP requests|
|Happens before application access|Reaches application|
|Uses SYN packets|Uses HTTP requests|

---

### 10. How do enterprises protect against SYN Flood?

Using:

- SYN Cookies.
    
- Firewalls.
    
- Load balancers.
    
- Rate limiting.
    
- DDoS protection.
    
- IDS/IPS.
    

---

# References

- [[TCP Handshake]]
    
- [[TCP-IP]]
    
- [[TCP]]
    
- [[Ports]]
    
- [[Firewall]]
    
- [[Load Balancer]]
    
- [[Reverse Proxy]]
    
- [[DDoS]]
    
- [[OSI Model]]
    
- [[HTTP]]
    
- [[HTTPS]]
    
- [[Spring Boot Security]]