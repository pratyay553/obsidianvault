# UDP

## Definition

**UDP (User Datagram Protocol)** is a **connectionless Transport Layer protocol** used to send data **quickly** without checking whether it reaches the destination.

Unlike TCP, UDP **does not establish a connection** and **does not guarantee delivery**.

> **Simple Definition**
>
> "UDP sends data as fast as possible without waiting for confirmation."

---

## Why it Matters

Some applications care more about **speed** than **100% reliability**.

Examples:

- Live video streaming
- Online gaming
- Voice calls (VoIP)
- Video calls
- DNS lookups
- Live sports streaming

Imagine you're on a video call.

If one video frame is lost, it's better to continue with the next frame than wait for the missing one.

That's why these applications use UDP.

---

## How it Works

UDP is very simple.

The sender sends data directly to the receiver.

```
Client
   │
   │ Datagram
   │────────────►
   │
Server
```

There is:

- ❌ No Handshake
- ❌ No ACK
- ❌ No Retransmission
- ❌ No Ordering
- ❌ No Flow Control

The sender simply sends the data and moves on.

---

## UDP Flow

```
Application

↓

UDP

↓

IP

↓

Internet

↓

Receiver
```

Unlike TCP, there is **no connection setup** before sending data.

---

## UDP vs TCP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | ✅ Yes | ❌ No |
| Reliable | ✅ Yes | ❌ No |
| Ordered Delivery | ✅ Yes | ❌ No |
| Error Recovery | ✅ Yes | ❌ No |
| Speed | Slower | Faster |
| Handshake | 3-Way Handshake | None |
| Best For | APIs, Websites, Databases | Streaming, Gaming, DNS |

---

## Real-Life Example

### TCP

Sending important documents by courier.

If one page is missing, the courier sends it again.

Reliable but slower.

---

### UDP

Listening to live radio.

If one second of audio is lost, you don't stop the broadcast.

You continue listening.

Fast but less reliable.

---

## Common Uses of UDP

- DNS
- DHCP
- Online Games
- Video Streaming
- Live Audio
- Voice Calls (VoIP)
- Video Calls
- IPTV

---

## Packet Flow

```
Game Server

↓

UDP

↓

Internet

↓

Player
```

If one packet is lost:

```
Packet 1 ✅

Packet 2 ❌ Lost

Packet 3 ✅
```

The game continues.

It does **not** wait for Packet 2.

---

## Why is UDP Faster?

UDP doesn't spend time on:

- Connection setup
- Waiting for ACK
- Retransmissions
- Packet ordering
- Flow control

This makes it lightweight and fast.

---

## Common Pitfalls

### ❌ UDP is always bad.

No.

UDP is the best choice when **speed is more important than perfect reliability**.

---

### ❌ UDP guarantees delivery.

No.

Packets can:

- Be lost
- Arrive out of order
- Be duplicated

The application must handle these cases if needed.

---

### ❌ HTTP uses UDP.

Normally, **HTTP and HTTPS use TCP**.

(Modern **HTTP/3** uses **QUIC**, which is built on UDP, but QUIC adds its own reliability and security.)

---

### ❌ UDP is only for games.

No.

Many important protocols use UDP:

- DNS
- DHCP
- NTP
- SNMP
- VoIP

---

## Spring Boot / Java Notes

Most Spring Boot applications use:

```
HTTP

↓

TCP

↓

IP
```

You usually don't use UDP for REST APIs.

However, Java supports UDP programming using:

```java
DatagramSocket
DatagramPacket
```

Typical use cases:

- IoT devices
- Multiplayer games
- Video streaming
- Device discovery
- Real-time telemetry

Most enterprise Java applications rely on TCP because they need reliable communication.

---

## Quick Revision

- UDP stands for **User Datagram Protocol**.
- It is a **Transport Layer** protocol.
- It is **connectionless**.
- No handshake.
- No guaranteed delivery.
- No packet ordering.
- Faster than TCP.
- Best for real-time applications.
- DNS commonly uses UDP.

---

## Interview Questions

### 1. What is UDP?

UDP (User Datagram Protocol) is a connectionless Transport Layer protocol that sends data without guaranteeing delivery.

---

### 2. Does UDP use a handshake?

No. UDP sends data immediately without establishing a connection.

---

### 3. Which is faster: TCP or UDP?

UDP is faster because it has less overhead.

---

### 4. Does UDP guarantee delivery?

No. Packets may be lost, duplicated, or arrive out of order.

---

### 5. Why do video calls use UDP?

Because real-time communication values low latency over perfect reliability. Losing a few packets is usually better than introducing delays.

---

### 6. Which applications commonly use UDP?

- DNS
- Online gaming
- Video streaming
- VoIP
- DHCP

---

### 7. Is UDP reliable?

No. Reliability must be handled by the application if required.

---

### 8. Does Spring Boot use UDP?

Not for REST APIs. Spring Boot applications typically use HTTP over TCP. UDP is used only for specific real-time or networking scenarios.

---

### 9. What is the main advantage of UDP?

Very low latency and high speed.

---

### 10. When should you choose UDP instead of TCP?

Choose UDP when speed and low latency are more important than guaranteed delivery, such as in streaming, gaming, or voice/video communication.

---

## References

- [[TCP-IP]]
- [[TCP Handshake]]
- [[IP]]
- [[HTTP]]
- [[HTTPS]]
- [[Ports]]
- [[OSI Model]]