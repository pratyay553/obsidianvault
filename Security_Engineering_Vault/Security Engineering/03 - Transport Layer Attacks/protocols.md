If you're learning **Computer Networks for Java/Spring Boot + System Design interviews**, don't try to memorize hundreds of protocols. Focus on the important **Transport Layer protocols** and understand what each one does.

## Transport Layer Protocols

### 1. TCP — Transmission Control Protocol ⭐⭐⭐

The most important one for backend developers.

```text
Application
    ↓
HTTP / HTTPS / WebSocket
    ↓
TCP
    ↓
IP
```

TCP provides:

- Connection-oriented communication
    
- Reliable delivery
    
- Ordered delivery
    
- Error detection
    
- Retransmission
    
- Flow control
    
- Congestion control
    

Example:

```text
Browser
   |
   | HTTP
   ↓
TCP Connection
   ↓
Tomcat :8080
```

Commonly associated with:

- HTTP/1.1
    
- HTTP/2
    
- HTTPS
    
- WebSocket
    
- SSH
    
- FTP
    

---

### 2. UDP — User Datagram Protocol ⭐⭐⭐

UDP is much simpler than TCP.

```text
Application
    ↓
DNS / RTP / QUIC
    ↓
UDP
    ↓
IP
```

UDP provides:

- Connectionless communication
    
- No guaranteed delivery
    
- No guaranteed ordering
    
- No retransmission by UDP itself
    
- Low protocol overhead
    

Example:

```text
Application
    |
    ↓
UDP
    |
    ↓
IP
```

Commonly used for:

- DNS
    
- DHCP
    
- VoIP
    
- Streaming
    
- Online gaming
    
- QUIC
    

---

### 3. QUIC ⭐⭐⭐

QUIC is very important for modern web development.

QUIC is a modern transport protocol that runs **over UDP**.

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
IP
```

QUIC provides features such as:

- Reliable delivery
    
- Encryption via TLS 1.3 integration
    
- Stream multiplexing
    
- Faster connection establishment
    
- Connection migration
    

This is why:

```text
HTTP/3 ≠ TCP
```

Instead:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
```

---

# Other Transport-Layer / Transport-Related Protocols

These are worth knowing conceptually, although they're much less important for typical Java backend interviews.

### 4. SCTP — Stream Control Transmission Protocol

SCTP provides reliable, message-oriented communication.

Features include:

- Reliable delivery
    
- Multiple streams
    
- Multi-homing
    
- Message-oriented communication
    

Used in some telecommunications and specialized networking systems.

```text
Application
    ↓
SCTP
    ↓
IP
```

---

### 5. DCCP — Datagram Congestion Control Protocol

DCCP provides:

- Datagram-style communication
    
- Congestion control
    

Unlike TCP, it does not provide TCP-style reliable byte-stream delivery.

It is relatively uncommon in everyday application development.

```text
Application
    ↓
DCCP
    ↓
IP
```

---

### 6. UDP-Lite

UDP-Lite is a variant of UDP that allows partially damaged datagrams to be delivered to the application.

It is useful for specialized applications where some corruption may be acceptable.

```text
Application
    ↓
UDP-Lite
    ↓
IP
```

It is not commonly used in normal Spring Boot development.

---

# What About ICMP?

This is a common interview trap.

**ICMP is NOT a Transport Layer protocol.**

It is generally considered part of the **Internet Layer / Network Layer**.

```text
Network Layer
    ↓
ICMP
```

For example:

```text
ping
 ↓
ICMP
 ↓
IP
```

So don't put ICMP in your Transport Layer list.

---

# What About IP?

**IP is NOT a Transport Layer protocol.**

IP belongs to the **Internet/Network Layer**.

```text
Application
    ↓
Transport
    ↓
TCP / UDP / QUIC
    ↓
IP
```

---

# What Should You Actually Memorize?

For your Java + Spring Boot + System Design journey, I'd prioritize them like this:

|Protocol|Importance|Remember|
|---|--:|---|
|**TCP**|⭐⭐⭐⭐⭐|Reliable, ordered, connection-oriented|
|**UDP**|⭐⭐⭐⭐⭐|Fast/simple, connectionless, no delivery guarantee|
|**QUIC**|⭐⭐⭐⭐|Modern transport over UDP; used by HTTP/3|
|**SCTP**|⭐⭐|Reliable, message-oriented, multi-stream|
|**DCCP**|⭐|Datagram + congestion control|
|**UDP-Lite**|⭐|Specialized UDP variant|

### Your core mental model

```text
                    APPLICATION
                         |
             +-----------+-----------+
             |           |           |
            HTTP       DNS       WebSocket
             |           |           |
             ↓           ↓           ↓
          +-----------------------------+
          |       TRANSPORT LAYER       |
          |                             |
          |   TCP    UDP    QUIC    SCTP|
          +-----------------------------+
                         |
                         ↓
                    IP / IPv6
                         |
                         ↓
                 Data Link / Network
```

**For interviews, master TCP, UDP, and QUIC first.** Once you understand those three, most of the networking questions you'll encounter in Java/Spring backend and system-design interviews become much easier.