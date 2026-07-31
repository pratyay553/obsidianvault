```markdown
# TCP Reset Attack

## Definition

A **TCP Reset Attack (TCP RST Attack)** is a network attack where an attacker sends **fake TCP RST (Reset) packets** to terminate an active TCP connection between two systems.

The attacker forces both sides to believe that the connection has failed.

> **Simple Definition**
>
> "TCP Reset Attack suddenly closes a TCP connection by sending fake reset packets."

Example:

Normal:

```

Client

<======== TCP Connection ========>

Server

```

During attack:

```

Client

<======== Connection ========>

Server

```
    ↑

Fake TCP RST

    ↑

Attacker
```

```

Result:

```

Connection Closed

```

---

# How it Works

## Normal TCP Communication

TCP maintains a connection using:

- Sequence numbers.
- Acknowledgment numbers.
- Connection states.

Example:

```

Client Server

DATA ----------------------->

```
    <-------------------- ACK
```

DATA ----------------------->

```

Connection continues.

---

# During TCP Reset Attack

The attacker observes an existing TCP connection.

Example:

```

Client

↓

Communication

↓

Server

```

The attacker creates a fake TCP RST packet:

```

Fake RST Packet

Client/Server

↓

Connection Terminated

```

Both systems believe the other side closed the connection.

---

# Attack Flow

```

Client

↓

TCP Connection

↓

Server

Attacker

↓

Fake TCP RST Packet

↓

Client + Server

↓

Connection Closed

```

---

# TCP RST Packet

A TCP Reset packet contains:

- Source IP.
- Destination IP.
- Source port.
- Destination port.
- TCP sequence number.
- RST flag.

Example:

```

TCP Flags:

[RST]

```

The RST flag tells the system:

```

"Immediately terminate this connection."

```

---

# Target Layer

**OSI Layer:** Layer 4 (Transport Layer)

Reason:

- The attack manipulates TCP communication.
- TCP operates at the Transport Layer.

---

# Prerequisites

For a TCP Reset Attack:

- Ability to observe network traffic.
- Ability to send forged TCP packets.
- Knowledge of connection details.

Required information:

- Source IP.
- Destination IP.
- Source port.
- Destination port.
- Sequence number.

Common environments:

- Public networks.
- Unsecured Wi-Fi.
- Compromised routers.
- ISP-level attacks.

---

# Real-world Example

A user is downloading a large file:

```

Client

↓

Large File Transfer

↓

Server

```

An attacker sends:

```

Fake TCP RST

```

The connection closes:

```

Download Failed

```

---

## Enterprise Example

An attacker wants to stop communication between:

```

Application Server

↓

Database Server

```

The attacker sends reset packets.

Result:

```

Database Connection Lost

```

The application experiences failures.

---

# Impact

Possible consequences:

- Forced disconnections.
- Service interruption.
- Failed transactions.
- Dropped downloads.
- Application instability.
- Network disruption.

For financial systems:

- Failed payments.
- Incomplete transactions.
- Customer experience issues.

---

# Detection

Signs of TCP Reset Attack:

- Unexpected connection termination.
- Many RST packets.
- Connections closing without normal FIN handshake.
- Repeated TCP resets from unknown sources.

Example:

Normal close:

```

FIN → ACK → FIN → ACK

```

Attack:

```

RST

````

Tools:

- Wireshark.
- tcpdump.
- IDS/IPS.
- Firewall logs.

Example:

```bash
tcpdump tcp[tcpflags] & tcp-rst != 0
````

---

# Prevention

## 1. Use Encryption (TLS)

TLS does not stop RST packets, but it protects data from modification.

Example:

```
Client

↓

TLS Connection

↓

Server
```

---

## 2. Network Security

Use:

- Firewalls.
    
- IDS/IPS.
    
- Traffic filtering.
    

---

## 3. Validate TCP Packets

Network devices can check:

- Sequence numbers.
    
- Connection state.
    
- Packet legitimacy.
    

---

## 4. Use Reliable Protocols

Applications should handle unexpected disconnections.

Example:

- Automatic retry.
    
- Connection recovery.
    
- Health checks.
    

---

## 5. Secure Networks

Avoid untrusted networks without protection.

Use:

- VPN.
    
- Secure Wi-Fi.
    

---

# Spring Boot Protection

Spring Boot cannot prevent TCP Reset Attacks because the attack happens at the TCP layer before HTTP reaches the application.

However, applications should handle connection failures properly.

---

## 1. Implement Retry Mechanisms

Example:

```
Request Failed

↓

Retry

↓

Recover Connection
```

Use:

- Spring Retry.
    
- Resilience4j.
    

---

## 2. Configure Connection Timeouts

Avoid hanging connections.

Example:

```properties
server.connection-timeout=5000
```

---

## 3. Handle Exceptions

Handle:

- Connection reset.
    
- Socket exceptions.
    
- Timeout errors.
    

Example:

```
java.net.SocketException:
Connection reset
```

---

## 4. Microservices Protection

For service communication:

Use:

- Service discovery.
    
- Circuit breakers.
    
- Retry policies.
    
- mTLS.
    

Example:

```
Order Service

↓

Circuit Breaker

↓

Payment Service
```

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

Spring Boot Services
```

Security controls:

- Stateful firewalls.
    
- IDS/IPS.
    
- Packet filtering.
    
- Network monitoring.
    
- VPN.
    
- Zero Trust networking.
    

---

# Best Practices

- Use TLS for communication.
    
- Monitor abnormal TCP resets.
    
- Use firewalls with state tracking.
    
- Implement application retries.
    
- Use circuit breakers.
    
- Avoid insecure public networks.
    
- Maintain updated network devices.
    
- Monitor connection health.
    
- Use redundant services.
    
- Perform network security testing.
    

---

# Interview Questions

### 1. What is a TCP Reset Attack?

A TCP Reset Attack is an attack where fake TCP RST packets are sent to terminate active TCP connections.

---

### 2. Which OSI layer does TCP Reset Attack target?

Layer 4 (Transport Layer).

---

### 3. Which TCP flag is used in this attack?

The RST (Reset) flag.

---

### 4. What is the difference between TCP FIN and TCP RST?

|TCP FIN|TCP RST|
|---|---|
|Graceful connection close|Immediate connection termination|
|Normal shutdown|Error or forced termination|
|Both sides complete closing process|Connection stops immediately|

---

### 5. Can HTTPS prevent TCP Reset Attack?

No.

HTTPS protects application data but cannot prevent TCP-level connection termination.

---

### 6. Can Spring Boot prevent TCP Reset Attack?

No.

The attack happens before requests reach the application, but Spring Boot can handle connection failures gracefully.

---

### 7. How does TCP know a packet belongs to a connection?

Using:

- Source IP.
    
- Destination IP.
    
- Source port.
    
- Destination port.
    
- Sequence numbers.
    

---

### 8. How can TCP Reset Attacks be detected?

Using:

- Packet analysis.
    
- IDS/IPS.
    
- Firewall logs.
    
- Network monitoring.
    

---

### 9. Difference between TCP SYN Flood and TCP Reset Attack?

|TCP SYN Flood|TCP Reset Attack|
|---|---|
|Creates many fake connections|Terminates existing connections|
|Targets availability|Targets active sessions|
|Uses SYN packets|Uses RST packets|
|Layer 4 attack|Layer 4 attack|

---

### 10. How do applications handle TCP connection failures?

Using:

- Retry mechanisms.
    
- Timeout handling.
    
- Circuit breakers.
    
- Error handling.
    

---

# References

- [[TCP-IP]]
    
- [[TCP Handshake]]
    
- [[TCP]]
    
- [[Ports]]
    
- [[Packet Sniffing]]
    
- [[Man-in-the-Middle]]
    
- [[Firewall]]
    
- [[VPN]]
    
- [[TLS]]
    
- [[HTTPS]]
    
- [[OSI Model]]
    
- [[Spring Boot Security]]
    
- [[Resilience4j]]