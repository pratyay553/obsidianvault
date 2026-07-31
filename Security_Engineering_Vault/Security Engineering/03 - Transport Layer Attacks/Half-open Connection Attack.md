# Half-open Connection Attack

## Definition

A **Half-open Connection Attack** is a **Denial-of-Service (DoS)** attack where an attacker creates a large number of **incomplete TCP connections** by starting the TCP handshake but never completing it.

The server keeps these **half-open connections** in memory, eventually exhausting its connection queue and preventing legitimate users from connecting.

> **Simple Definition**
>
> "A Half-open Connection Attack fills the server with incomplete TCP connections until it cannot accept new users."

> **Note:** A **TCP SYN Flood** is the most common type of Half-open Connection Attack.

---

# How it Works

## Normal TCP Handshake

```
Client                      Server

SYN  ---------------------->

      <-------------------- SYN + ACK

ACK  ---------------------->

Connection Established
```

The connection is fully established.

---

## During Half-open Connection Attack

The attacker sends many SYN packets.

```
Attacker

↓

SYN

↓

Server
```

The server replies:

```
SYN + ACK
```

But the attacker never sends the final ACK.

The server waits for the connection to complete.

```
Connection State:

SYN_RECEIVED
```

After thousands of incomplete requests:

```
Connection Queue Full

↓

New Connections Rejected
```

---

# Attack Flow

```
Attacker

↓

Thousands of SYN Packets

↓

Server Creates Half-open Connections

↓

Connection Queue Fills

↓

Legitimate Users Blocked
```

---

# Target Layer

**OSI Layer:** Layer 4 (Transport Layer)

Reason:

- Exploits the TCP three-way handshake.
- Targets TCP connection management.

---

# Prerequisites

For a Half-open Connection Attack:

- Ability to send many TCP SYN packets.
- Network connectivity to the target.
- A server using TCP.

Large-scale attacks often use:

- Botnets.
- Distributed systems.
- Spoofed IP addresses.

Common targets:

- Web servers.
- API servers.
- VPN gateways.
- Mail servers.
- Load balancers.

---

# Real-world Example

An online shopping website normally supports:

```
5,000 Active Connections
```

An attacker sends:

```
500,000 SYN Requests
```

The server stores:

```
Half-open Connections

↓

Memory Consumption

↓

Queue Full
```

Result:

```
Customer

↓

Cannot Open Website
```

---

# Impact

Possible consequences:

- Website unavailable.
- Connection failures.
- High memory usage.
- CPU overhead.
- Service interruption.
- Poor user experience.

This attack mainly affects:

**Availability** in the CIA Triad.

---

# Detection

Common indicators:

- Large number of connections in:

```
SYN_RECEIVED
```

- Many incomplete TCP handshakes.
- High SYN traffic.
- Low connection completion rate.
- Increased server resource usage.

Useful tools:

- `netstat`
- `ss`
- Wireshark
- IDS/IPS
- Firewall logs

Example:

```bash
netstat -an | grep SYN_RECV
```

or

```bash
ss -ant state syn-recv
```

---

# Prevention

## 1. Enable SYN Cookies ✅

SYN Cookies allow the server to avoid allocating memory until the handshake is completed.

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

Allow more pending connections during traffic spikes.

---

## 3. Configure Shorter SYN Timeout

Reduce the waiting time for incomplete connections.

---

## 4. Rate Limiting

Limit the number of new TCP connections from a single IP.

---

## 5. Stateful Firewall

Drop abnormal SYN traffic before it reaches the server.

---

## 6. DDoS Protection

Use services such as:

- Cloudflare
- AWS Shield
- Azure DDoS Protection
- Google Cloud Armor

---

# Spring Boot Protection

Spring Boot cannot directly prevent a Half-open Connection Attack because it occurs **before HTTP requests reach the application**.

However, you should design the deployment securely.

---

## 1. Place Spring Boot Behind a Reverse Proxy

```
Internet

↓

Nginx

↓

Spring Boot
```

The reverse proxy handles incoming TCP connections.

---

## 2. Use a Load Balancer

```
Client

↓

Load Balancer

↓

Spring Boot Cluster
```

Traffic is distributed across multiple instances.

---

## 3. Configure Connection Timeouts

Example:

```properties
server.tomcat.connection-timeout=5000
```

This helps release idle connections more quickly.

---

## 4. Monitor Application Health

Monitor:

- Active connections.
- Response time.
- Thread usage.
- Error rate.

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

Reverse Proxy

↓

Spring Boot Services

↓

Database
```

Security controls:

- SYN Cookies.
- Stateful firewalls.
- IDS/IPS.
- Rate limiting.
- Auto scaling.
- Continuous monitoring.

---

# Best Practices

- Enable SYN Cookies.
- Keep operating systems updated.
- Close unnecessary ports.
- Use firewalls.
- Configure proper TCP timeouts.
- Deploy load balancers.
- Monitor TCP connection states.
- Use DDoS protection services.
- Perform regular load testing.
- Monitor server resource usage.

---

# Interview Questions

### 1. What is a Half-open Connection Attack?

It is a DoS attack where attackers create many incomplete TCP connections to exhaust server resources.

---

### 2. Which OSI layer does it target?

Layer 4 (Transport Layer).

---

### 3. What is a half-open TCP connection?

A connection where the server has received a SYN request but has not yet received the final ACK.

---

### 4. Is TCP SYN Flood the same as a Half-open Connection Attack?

Yes.

A **TCP SYN Flood** is the most common implementation of a Half-open Connection Attack.

---

### 5. Why does the server consume memory?

Because it stores information for each incomplete TCP connection while waiting for the final ACK.

---

### 6. What are SYN Cookies?

A security mechanism that delays allocating connection resources until the client completes the TCP handshake.

---

### 7. Can HTTPS prevent this attack?

No.

The attack happens at Layer 4 before HTTPS communication begins.

---

### 8. Can Spring Boot stop this attack?

No.

Protection must occur at the operating system, firewall, load balancer, or reverse proxy.

---

### 9. How do enterprises defend against Half-open Connection Attacks?

Using:

- SYN Cookies.
- Firewalls.
- Load balancers.
- Rate limiting.
- DDoS protection.
- IDS/IPS.

---

### 10. Which CIA principle is primarily affected?

**Availability**, because legitimate users cannot establish new connections.

---

# References

- [[TCP SYN Flood]]
- [[TCP Handshake]]
- [[TCP-IP]]
- [[TCP]]
- [[Firewall]]
- [[Reverse Proxy]]
- [[Load Balancer]]
- [[DDoS]]
- [[Ports]]
- [[OSI Model]]
- [[Spring Boot Security]]