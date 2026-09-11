# TCP-IP

## Definition

**TCP/IP (Transmission Control Protocol / Internet Protocol)** is the **standard networking model used by the Internet**. It defines how data is **packaged, addressed, transmitted, routed, and received** between devices.

Unlike the **OSI Model** (which is mainly for learning), **TCP/IP is the model actually used in real-world networking**.

> **Simple Definition**
>
> "TCP/IP is the set of rules that allows computers to communicate over the Internet."

---

## Why it Matters

Every time you:

- Open Google
- Watch YouTube
- Use WhatsApp
- Call an API
- Connect to a database

...your data travels using the **TCP/IP protocol suite**.

As a Java/Spring Boot developer, every REST API request follows TCP/IP.

Without TCP/IP:

- Browsers cannot access websites.
- Servers cannot communicate.
- APIs cannot exchange data.
- The Internet would not work.

---

## TCP/IP Layers

TCP/IP has **4 layers**.

| TCP/IP Layer | Responsibility | OSI Equivalent |
|--------------|----------------|----------------|
| Application | User applications and protocols | Layer 5–7 |
| Transport | Reliable communication | Layer 4 |
| Internet | Routing using IP | Layer 3 |
| Network Access | Physical transmission | Layer 1–2 |

---

## OSI vs TCP/IP

| OSI | TCP/IP |
|------|---------|
| 7 Layers | 4 Layers |
| Reference Model | Real-world Model |
| Used for Learning | Used by the Internet |
| More Detailed | Simpler |

Easy mapping:

```
OSI                    TCP/IP

Application ─┐
Presentation ├──► Application
Session ─────┘

Transport ─────► Transport

Network ───────► Internet

Data Link ─┐
Physical ──┘──► Network Access
```

---

## How it Works

Suppose you open:

```
https://amazon.com
```

The request goes through these layers:

```
Application Layer
    │
HTTP Request
    │
Transport Layer
    │
TCP adds Port Numbers
    │
Internet Layer
    │
IP adds Source & Destination IP
    │
Network Access Layer
    │
Ethernet/WiFi sends Bits
    │
Internet
    │
Amazon Server
```

The server removes the headers in the reverse order and processes your request.

---

## Responsibilities of Each Layer

### 1. Application Layer

Closest to the user.

Provides network services.

Protocols:

- HTTP
- HTTPS
- FTP
- SMTP
- DNS
- SSH

Example:

```
Chrome

↓

HTTP Request
```

Think:

> "This is where applications communicate."

---

### 2. Transport Layer

Responsible for:

- Reliable communication
- Segmentation
- Port Numbers
- Error Recovery
- Flow Control

Protocols:

- TCP
- UDP

Think:

> "Deliver the data."

---

### 3. Internet Layer

Responsible for:

- IP Address
- Routing
- Packet forwarding

Protocols:

- IPv4
- IPv6
- ICMP

Device:

- Router

Think:

> "Find the destination."

---

### 4. Network Access Layer

Responsible for:

- MAC Address
- Frames
- Physical transmission

Examples:

- Ethernet
- WiFi
- Fiber
- Switch

Think:

> "Move the data over the network."

---

## TCP vs IP

### TCP

Responsible for:

- Reliable delivery
- Error checking
- Ordering
- Retransmission
- Flow control

Example:

```
Data

↓

1
2
3
4

Receiver gets

1
2
3
4
```

No data is lost without detection.

---

### IP

Responsible for:

- Addressing
- Routing

Example:

```
Source IP

192.168.1.10

↓

Destination IP

142.250.182.206
```

IP does **not** guarantee delivery.

---

## Real-Life Example

Imagine sending a courier.

### TCP

Checks:

- Is the parcel complete?
- Did every box arrive?
- Should any missing box be resent?

### IP

Finds:

- The destination address
- The best route

Together:

```
TCP

Makes sure parcel is complete.

+

IP

Finds the address.
```

---

## Encapsulation

When sending data:

```
Application Data

↓

TCP Header

↓

IP Header

↓

Ethernet Header

↓

Bits
```

Each layer adds its own information.

At the receiver:

```
Bits

↓

Ethernet Removed

↓

IP Removed

↓

TCP Removed

↓

Original Data
```

---

## Common Protocols

| Layer | Protocols |
|---------|-----------|
| Application | HTTP, HTTPS, FTP, SMTP, DNS |
| Transport | TCP, UDP |
| Internet | IPv4, IPv6, ICMP |
| Network Access | Ethernet, WiFi, ARP |

---

## Common Pitfalls

### ❌ TCP/IP and TCP are not the same.

TCP is **one protocol**.

TCP/IP is the **entire networking model (protocol suite)**.

---

### ❌ IP does not guarantee delivery.

Many beginners think IP ensures packets arrive.

Actually:

**TCP** provides reliable delivery.

IP only routes packets.

---

### ❌ HTTP is not part of TCP.

HTTP runs **on top of TCP**.

```
HTTP

↓

TCP

↓

IP
```

---

### ❌ UDP is part of TCP/IP.

The Transport layer supports **both TCP and UDP**.

---

### ❌ TCP/IP has 4 layers, not 7.

The **OSI model has 7 layers**.

The **TCP/IP model has 4 layers**.

---

## Spring Boot / Java Notes

Whenever a Spring Boot application exposes a REST API:

```
Client

↓

HTTP

↓

TCP

↓

IP

↓

Internet

↓

Spring Boot Server
```

Example:

```
@GetMapping("/users")
public List<User> getUsers() {
    return service.getUsers();
}
```

Flow:

```
Browser

↓

HTTP Request

↓

TCP Connection

↓

IP Routing

↓

Spring Boot

↓

Response

↓

Browser
```

You usually write code at the **Application Layer**, but understanding the lower layers helps when troubleshooting:

- Connection refused
- Timeout
- SSL issues
- DNS failures
- Firewall blocks
- Load balancer problems

---

## Quick Revision

- TCP/IP is the networking model used by the Internet.
- It has **4 layers**.
- TCP provides **reliable communication**.
- IP provides **addressing and routing**.
- HTTP runs on top of TCP.
- Every REST API request uses TCP/IP.
- OSI is mainly for learning; TCP/IP is used in practice.

---

## Interview Questions

### 1. What is TCP/IP?

A protocol suite (networking model) that enables communication over the Internet.

---

### 2. How many layers does TCP/IP have?

4 layers.

---

### 3. What is the difference between OSI and TCP/IP?

OSI is a conceptual 7-layer reference model, while TCP/IP is the practical 4-layer model used on the Internet.

---

### 4. What is the responsibility of TCP?

Reliable data delivery, ordering, retransmission, and flow control.

---

### 5. What is the responsibility of IP?

Addressing and routing packets between networks.

---

### 6. Does IP guarantee delivery?

No. IP is a best-effort protocol. Reliability is provided by TCP.

---

### 7. Which layer does HTTP belong to?

Application Layer.

---

### 8. Which layer does TCP belong to?

Transport Layer.

---

### 9. Why is TCP/IP important for Spring Boot developers?

Because every HTTP request and REST API communication uses the TCP/IP stack.

---

### 10. Which is used in the real world: OSI or TCP/IP?

TCP/IP is used in real-world networking. OSI is mainly used for understanding and troubleshooting.

---

## References

- [[OSI Model]]
- [[TCP Handshake]]
- [[UDP]]
- [[IP]]
- [[DNS]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[Ports]]
- [[Routing]]
```