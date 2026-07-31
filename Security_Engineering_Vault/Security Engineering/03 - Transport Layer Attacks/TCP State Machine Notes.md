# TCP State Machine, SYN Queue, Backlog Queue

## Definition

The **TCP State Machine** defines the different states a TCP connection goes through from **connection creation to connection termination**.

The **SYN Queue** stores **half-open connections** (connections waiting for the final ACK).

The **Backlog Queue (Accept Queue)** stores **fully established connections** waiting for the application (like Spring Boot/Tomcat) to accept them.

> **Simple Definition**
>
> - **TCP State Machine** = Life cycle of a TCP connection.
> - **SYN Queue** = Waiting room for incomplete connections.
> - **Backlog Queue** = Waiting room for completed connections.

---

# Why it Matters

Understanding these concepts helps explain:

- How TCP creates reliable connections.
- Why **TCP SYN Flood** attacks work.
- Why servers reject connections under heavy load.
- How Linux and web servers manage incoming connections.
- Performance tuning for high-traffic applications.

For interviews:

> "SYN Queue stores half-open connections, while the Backlog Queue stores completed connections waiting for the application."

---

# How it Works

## Step 1: Client Sends SYN

```
Client

↓

SYN

↓

Server
```

The server creates a **half-open connection**.

Connection State:

```
SYN_RECEIVED
```

The connection enters the **SYN Queue**.

---

## Step 2: Server Replies

```
Client               Server

SYN ------------->

      <----------- SYN + ACK
```

The server is waiting for the final ACK.

Still inside:

```
SYN Queue
```

---

## Step 3: Client Sends ACK

```
Client               Server

ACK ------------->
```

The TCP handshake is completed.

Connection State:

```
ESTABLISHED
```

The connection moves to the:

```
Backlog Queue
```

---

## Step 4: Application Accepts Connection

Example:

```
Linux Kernel

↓

Backlog Queue

↓

Tomcat

↓

Spring Boot
```

Tomcat accepts the connection and starts processing the HTTP request.

---

# TCP Connection States

The most important TCP states are:

| State | Meaning |
|--------|---------|
| LISTEN | Server is waiting for connections |
| SYN_SENT | Client has sent a SYN packet |
| SYN_RECEIVED | Server received SYN and is waiting for ACK |
| ESTABLISHED | Connection is active |
| FIN_WAIT_1 | Closing has started |
| FIN_WAIT_2 | Waiting for peer to close |
| CLOSE_WAIT | Waiting for application to close |
| LAST_ACK | Waiting for final ACK |
| TIME_WAIT | Waiting to ensure delayed packets expire |
| CLOSED | Connection is fully closed |

### Easy Memory Trick

```
LISTEN

↓

SYN_RECEIVED

↓

ESTABLISHED

↓

FIN_WAIT

↓

TIME_WAIT

↓

CLOSED
```

---

# SYN Queue

The **SYN Queue** contains **half-open connections**.

Example:

```
Server

↓

SYN Queue

-----------------

Connection A

Connection B

Connection C

-----------------
```

These connections are waiting for:

```
Final ACK
```

If the queue becomes full:

```
New SYN Requests

↓

Dropped
```

This is why **TCP SYN Flood** attacks are effective.

---

# Backlog Queue (Accept Queue)

After the handshake completes:

```
Connection

↓

ESTABLISHED

↓

Backlog Queue
```

Example:

```
Backlog Queue

-----------------

Client 1

Client 2

Client 3

-----------------
```

The application removes connections one by one.

```
Tomcat

↓

accept()

↓

Spring Boot
```

---

# Complete Flow

```
Client

↓

SYN

↓

SYN Queue

↓

ACK Received

↓

Backlog Queue

↓

Tomcat

↓

Spring Boot Controller
```

---

# Common Pitfalls

### 1. SYN Queue Full

Cause:

- SYN Flood attack.
- Very high traffic.

Result:

```
New Connections Rejected
```

---

### 2. Backlog Queue Full

Cause:

- Application is slow.
- Thread pool exhausted.
- CPU overload.

Result:

```
TCP Handshake Completed

↓

Application Cannot Accept Connection
```

Clients may experience:

- Slow response.
- Connection timeout.
- Connection refused.

---

### 3. Long Processing Time

Example:

```
Database Slow

↓

Tomcat Busy

↓

Backlog Queue Grows
```

---

### 4. Wrong Server Configuration

Small backlog size:

```
Backlog = 50
```

High traffic:

```
5000 Clients
```

Result:

```
Dropped Connections
```

---

# Spring Boot / Java Notes

Spring Boot itself **does not manage the TCP handshake**.

The flow is:

```
Client

↓

Linux TCP Stack

↓

SYN Queue

↓

Backlog Queue

↓

Tomcat

↓

Spring Boot

↓

Controller
```

### Important Configuration

Tomcat accepts connections from the backlog queue.

Example:

```properties
server.tomcat.accept-count=100
```

- `accept-count` defines how many completed connections can wait before being accepted by Tomcat.

Other useful settings:

```properties
server.tomcat.threads.max=200
server.tomcat.connection-timeout=5000
```

For production:

- Use a reverse proxy (Nginx/Apache).
- Use a load balancer.
- Tune backlog and thread pool based on expected traffic.
- Monitor connection states.

---

# Interview Questions

### 1. What is the TCP State Machine?

It is the collection of states that describe the life cycle of a TCP connection, from creation to termination.

---

### 2. What is the SYN Queue?

The SYN Queue stores **half-open TCP connections** waiting for the final ACK of the three-way handshake.

---

### 3. What is the Backlog Queue?

The Backlog Queue stores **fully established TCP connections** waiting for the application (such as Tomcat) to accept them.

---

### 4. What is the difference between SYN Queue and Backlog Queue?

| SYN Queue | Backlog Queue |
|-----------|---------------|
| Half-open connections | Fully established connections |
| Waiting for ACK | Waiting for application |
| State: `SYN_RECEIVED` | State: `ESTABLISHED` |

---

### 5. Why is the SYN Queue important?

It allows the server to track incomplete TCP handshakes until they are completed or timed out.

---

### 6. What happens if the SYN Queue is full?

New SYN requests may be dropped or ignored, causing connection failures.

---

### 7. What happens if the Backlog Queue is full?

The TCP handshake may complete, but the application cannot accept new connections until space becomes available.

---

### 8. Does Spring Boot manage the SYN Queue?

No.

The operating system's TCP stack manages the SYN Queue and Backlog Queue. Spring Boot receives connections only after Tomcat accepts them.

---

### 9. Which attack targets the SYN Queue?

**TCP SYN Flood**, by creating many half-open connections.

---

### 10. Easy interview explanation?

> "The SYN Queue stores incomplete TCP handshakes. After the handshake finishes, the connection moves to the Backlog Queue, where it waits for Tomcat or another application server to accept it."

---

# References

- [[TCP]]
- [[TCP Handshake]]
- [[TCP SYN Flood]]
- [[Half-open Connection Attack]]
- [[TCP Reset Attack]]
- [[Ports]]
- [[OSI Model]]
- [[TCP-IP]]
- [[Spring Boot]]
- [[Tomcat]]
- [[Reverse Proxy]]
- [[Load Balancer]]