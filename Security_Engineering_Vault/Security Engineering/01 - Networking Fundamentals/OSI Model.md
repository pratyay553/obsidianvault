# OSI Model

## Definition

The **OSI (Open Systems Interconnection) Model** is a **7-layer networking model** that explains **how data travels from one device to another over a network**.

It is a **conceptual model**, which means it is used for **learning, designing, and troubleshooting networks**. In real-world applications, the **TCP/IP model** is used more often, but almost every interviewer expects you to know the OSI model.

> **Simple Definition**
>
> "OSI is a step-by-step guide that shows how data moves from an application on one computer to an application on another computer."

---

## Why it Matters

Imagine you send a message on WhatsApp.

```
You
 ↓
Internet
 ↓
Friend
```

A lot of things happen behind the scenes:

- The message is converted into data.
- The data is divided into smaller pieces.
- Each piece gets an address.
- It travels through routers and switches.
- The receiver rebuilds the original message.

The OSI model explains **which layer is responsible for each task**.

Without the OSI model:

- Network troubleshooting becomes difficult.
- Different vendors cannot build compatible devices.
- Protocol design becomes confusing.

---

## The 7 Layers

| Layer | Name         | Main Responsibility          | Examples                  |
| ----- | ------------ | ---------------------------- | ------------------------- |
| 7     | Application  | User-facing network services | HTTP, HTTPS, FTP, SMTP    |
| 6     | Presentation | Data format, Encryption      | TLS, SSL, JPEG, JSON      |
| 5     | Session      | Create and manage sessions   | NetBIOS, RPC              |
| 4     | Transport    | Reliable communication       | TCP, UDP                  |
| 3     | Network      | Routing and IP Address       | IP, ICMP                  |
| 2     | Data Link    | MAC Address, Frame delivery  | Ethernet, Switch          |
| 1     | Physical     | Transfer Bits                | Cable, Fiber, WiFi Signal |

---

## Easy Memory Trick

From Top to Bottom

```
All
People
Seem
To
Need
Data
Processing
```

Application

Presentation

Session

Transport

Network

Data Link

Physical

---

From Bottom to Top

```
Please
Do
Not
Throw
Sausage
Pizza
Away
```

Physical

Data Link

Network

Transport

Session

Presentation

Application

Choose whichever is easier for you.

---

## How Data Travels

Suppose you open Amazon in Chrome.

```
Application
↓
Presentation
↓
Session
↓
Transport
↓
Network
↓
Data Link
↓
Physical
~~~~~~~~~~~~ Internet ~~~~~~~~~~~~
Physical
↑
Data Link
↑
Network
↑
Transport
↑
Session
↑
Presentation
↑
Application
```

Sender goes **Top → Bottom**

Receiver goes **Bottom → Top**

This process is called:

- **Encapsulation** (sending)
- **Decapsulation** (receiving)

---

## What Each Layer Does (Simple Way)

### Layer 7 – Application

Closest to the user.

Provides network services to applications.

Examples:

- Browser
- Gmail
- WhatsApp
- Chrome

Protocols:

- HTTP
- HTTPS
- FTP
- SMTP
- DNS

Think:

> "User starts here."

---

### Layer 6 – Presentation

Makes data understandable.

Responsibilities:

- Encryption
- Decryption
- Compression
- Data Formatting

Example:

```
JSON

↓

Encrypted

↓

Compressed
```

Think:

> "Prepare the data."

---

### Layer 5 – Session

Starts, manages, and ends communication.

Example:

```
Login

↓

Use Website

↓

Logout
```

Think:

> "Maintain the conversation."

---

### Layer 4 – Transport

Breaks data into smaller pieces.

Ensures delivery.

Protocols:

- TCP
- UDP

Think:

> "Deliver safely."

---

### Layer 3 – Network

Adds IP Address.

Finds the destination.

Uses Routers.

Protocol:

- IP

Think:

> "Find the road."

---

### Layer 2 – Data Link

Uses MAC Address.

Transfers data inside the local network.

Uses Switch.

Think:

> "Deliver inside the building."

---

### Layer 1 – Physical

Converts bits into electrical, optical, or radio signals.

Examples:

- Ethernet Cable
- Fiber Cable
- WiFi Signal

Think:

> "Move the bits."

---

## Real Life Example

Sending a courier.

| OSI Layer | Real Life |
|-----------|-----------|
| Application | Write the letter |
| Presentation | Put it in an envelope |
| Session | Start communication |
| Transport | Courier company |
| Network | Choose delivery route |
| Data Link | Local delivery office |
| Physical | Truck, Bike, Plane |

---

## Device Mapping

| Device | OSI Layer |
|----------|-----------|
| Hub | Layer 1 |
| Repeater | Layer 1 |
| Switch | Layer 2 |
| Bridge | Layer 2 |
| Router | Layer 3 |
| Firewall | Layer 3–7 |
| Load Balancer | Layer 4/7 |
| Browser | Layer 7 |

---

## Data Unit at Each Layer

| Layer | Data Unit |
|---------|-----------|
| Application | Data |
| Presentation | Data |
| Session | Data |
| Transport | Segment (TCP) / Datagram (UDP) |
| Network | Packet |
| Data Link | Frame |
| Physical | Bits |

Interviewers love this table.

---

## Common Pitfalls

### ❌ OSI is not used directly.

It is only a reference model.

Real networking mainly follows the **TCP/IP model**.

---

### ❌ HTTP is not a Transport protocol.

Wrong.

HTTP works at **Application Layer**.

TCP works at **Transport Layer**.

---

### ❌ IP does not guarantee delivery.

Reliable delivery is handled by **TCP**, not IP.

---

### ❌ Switches don't use IP.

Normal Layer-2 switches use **MAC addresses**.

Routers use **IP addresses**.

---

### ❌ Encryption is not Transport Layer.

Encryption is mainly handled at the **Presentation Layer** in the OSI model (though in practice, protocols like TLS span multiple layers).

---

## Spring Boot / Java Notes

As a Spring Boot developer, you usually work with the upper layers of the networking stack.

Common mapping:

| Technology | OSI Layer |
|------------|-----------|
| REST API | Application |
| HTTPS | Application + Presentation |
| TLS | Presentation |
| HTTP Session | Session |
| TCP | Transport |
| IP | Network |

Example:

```
Browser

↓

HTTPS

↓

Spring Boot REST API

↓

Tomcat

↓

TCP

↓

IP

↓

Internet
```

You rarely write code for Layers 1–3 directly, but understanding them helps when debugging connectivity issues.

---

## Quick Revision

- OSI has **7 layers**.
- Sender: **Top → Bottom**.
- Receiver: **Bottom → Top**.
- Transport Layer uses **TCP/UDP**.
- Network Layer uses **IP**.
- Data Link Layer uses **MAC Address**.
- Physical Layer transfers **Bits**.
- Application Layer is closest to the user.
- OSI is mainly for learning and troubleshooting.
- TCP/IP is the model used in real systems.

---

## Interview Questions

### 1. What is the OSI Model?

A conceptual 7-layer model that explains how data moves between devices over a network.

---

### 2. How many layers are in the OSI Model?

7 layers.

---

### 3. Which layer uses IP Address?

Network Layer (Layer 3).

---

### 4. Which layer uses MAC Address?

Data Link Layer (Layer 2).

---

### 5. Which layer uses TCP and UDP?

Transport Layer (Layer 4).

---

### 6. Which layer does HTTP belong to?

Application Layer (Layer 7).

---

### 7. Which device works at Layer 2?

Switch.

---

### 8. Which device works at Layer 3?

Router.

---

### 9. What is encapsulation?

Adding protocol information as data moves **down** the OSI layers before transmission.

---

### 10. Why should a Java developer learn the OSI Model?

To understand how web requests travel, debug networking issues, configure servers, and communicate effectively with DevOps and infrastructure teams.

---

## References

- [[TCP-IP]]
- [[TCP Handshake]]
- [[UDP]]
- [[IP]]
- [[DNS]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[Firewall]]
- [[Reverse Proxy]]

