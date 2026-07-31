# TCP Handshake

## Definition

The **TCP Three-Way Handshake** is the process used to **establish a reliable connection** between a client and a server **before any data is exchanged**.

Think of it as two people confirming they are both ready before starting a conversation.

> **Simple Definition**
>
> "The TCP Handshake is a 3-step process that creates a reliable connection between two devices."

---

## Why it Matters

Whenever you:

- Open a website
- Call a REST API
- Connect to a database
- Login to an application

...a TCP handshake usually happens first.

Without it:

- The client doesn't know if the server is ready.
- The server doesn't know if the client is reachable.
- Reliable communication cannot begin.

---

## How it Works

TCP uses **3 steps** to establish a connection.

### Step 1 - SYN

The client sends a **SYN (Synchronize)** packet.

Meaning:

> "Hello Server, I want to connect."

```
Client
   │
   │ SYN
   │────────────►
   │
Server
```

---

### Step 2 - SYN + ACK

The server replies with **SYN + ACK**.

Meaning:

> "I received your request. I'm ready."

```
Client
   │
   │ SYN
   │────────────►
   │
   │◄────────────
   │ SYN + ACK
Server
```

---

### Step 3 - ACK

The client sends the final **ACK**.

Meaning:

> "Great! Let's start communicating."

```
Client
   │
   │ SYN
   │────────────►
   │
   │◄────────────
   │ SYN + ACK
   │
   │ ACK
   │────────────►
Server
```

Now the TCP connection is established.

Data transfer can begin.

---

## Easy Way to Remember

Imagine calling your friend.

```
You:
Can we talk?

Friend:
Yes, I'm here.

You:
Okay, let's start.
```

Exactly the same as:

```
SYN

↓

SYN + ACK

↓

ACK
```

---

## Complete Flow

```
Client                           Server

   SYN ------------------------->

       <------------------- SYN + ACK

   ACK ------------------------->

========= Connection Established =========

HTTP Request -------------------->

<------------------------- HTTP Response
```

---

## Why Three Steps?

The handshake ensures:

- Both devices are online.
- Both devices are ready.
- Initial sequence numbers are exchanged.
- Reliable communication can begin.

This is why TCP is called a **connection-oriented protocol**.

---

## What Happens After the Handshake?

Once the connection is established:

```
HTTP Request

↓

TCP

↓

Internet

↓

Server

↓

HTTP Response
```

The handshake happens **only once** when the connection starts.

After that, data flows normally until the connection is closed.

---

## Connection Termination

TCP also closes the connection safely.

It uses **4 steps**:

```
FIN

↓

ACK

↓

FIN

↓

ACK
```

Think:

> Handshake = Start communication

> FIN = End communication

---

## Real-Life Example

Ordering food online.

```
You:
Can I place an order?

Restaurant:
Yes.

You:
Great, here's my order.
```

After confirmation, the conversation continues.

---

## Common Pitfalls

### ❌ HTTP Handshake and TCP Handshake are different.

TCP creates the connection.

HTTP sends the request.

```
HTTP

runs

OVER

TCP.
```

---

### ❌ UDP does not use a handshake.

UDP sends data immediately.

No connection is established.

---

### ❌ HTTPS still uses TCP Handshake.

For HTTPS:

```
TCP Handshake

↓

TLS Handshake

↓

HTTPS Communication
```

Both handshakes happen before encrypted data is exchanged.

---

### ❌ Handshake happens before every packet.

No.

The handshake happens **once per TCP connection**, not for every packet.

---

## Spring Boot / Java Notes

Suppose your application exposes:

```java
@GetMapping("/users")
public List<User> getUsers() {
    return service.getUsers();
}
```

When a browser calls:

```
GET /users
```

The flow is:

```
Browser

↓

TCP Handshake

↓

TLS Handshake (HTTPS only)

↓

HTTP GET Request

↓

Spring Boot Controller

↓

Response

↓

Browser
```

As a Spring Boot developer, you don't write handshake code yourself.

The operating system and TCP stack handle it automatically.

However, understanding it helps when debugging:

- Connection refused
- Connection timeout
- Slow first request
- Load balancer issues
- Reverse proxy problems

---

## Quick Revision

- TCP Handshake creates a reliable connection.
- It has **3 steps**.
- Steps are:
  - SYN
  - SYN + ACK
  - ACK
- After the handshake, data transfer begins.
- TCP is **connection-oriented**.
- UDP does **not** perform a handshake.
- HTTPS uses **TCP Handshake first**, then **TLS Handshake**.

---

## Interview Questions

### 1. What is the TCP Three-Way Handshake?

A process that establishes a reliable TCP connection between a client and a server before data is exchanged.

---

### 2. What are the three steps?

1. SYN
2. SYN + ACK
3. ACK

---

### 3. Why is the handshake required?

To confirm that both devices are reachable and ready for reliable communication.

---

### 4. Which protocol uses the Three-Way Handshake?

TCP.

---

### 5. Does UDP use a handshake?

No. UDP is connectionless.

---

### 6. Does HTTPS use the TCP Handshake?

Yes. First the TCP Handshake occurs, then the TLS Handshake, and finally encrypted HTTP communication begins.

---

### 7. What happens after the handshake?

The client and server exchange application data, such as HTTP requests and responses.

---

### 8. What is a connection-oriented protocol?

A protocol that establishes a connection before sending data. TCP is connection-oriented.

---

### 9. How is a TCP connection closed?

Using a Four-Way Termination process with FIN and ACK packets.

---

### 10. Why should a Java developer understand the TCP Handshake?

Because every REST API, database connection, and web request relies on TCP. It helps diagnose connection failures, latency, and networking issues.

---

## References

- [[TCP-IP]]
- [[UDP]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[IP]]
- [[Ports]]
- [[OSI Model]]