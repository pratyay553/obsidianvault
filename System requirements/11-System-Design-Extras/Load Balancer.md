---
title: "Load Balancer"
aliases:
  - Load Balancing
  - Load Balancer
tags:
  - system-design
  - computer-networks
  - backend
  - distributed-systems
  - spring-boot
  - devops
difficulty: beginner
status: learning
---

# Load Balancer

> [!abstract] One-Line Definition
> A **Load Balancer** is a component that receives incoming client requests and distributes them across multiple servers so that no single server becomes overloaded.

---

## 1. Big Picture

### Simple Explanation

Think of a restaurant with many counters. Instead of every customer going to the same counter, a person at the entrance directs customers to different counters so no single counter gets overwhelmed.

Mapped to software:

```text
Users
  |
  v
Load Balancer
  |
  +--------> Server 1
  |
  +--------> Server 2
  |
  +--------> Server 3
```

We need multiple servers because one machine has limited CPU, memory, and network capacity — beyond a point it simply cannot keep up with traffic.

---

## 2. Why It Matters

### Problems with a single server

```text
Users
  |
  v
Server
```

- Too much traffic
- Server overload
- Slow response
- Single point of failure
- Difficult maintenance
- Limited scalability

### With a Load Balancer

```text
                 +--> Server 1
                 |
Users --> Load Balancer --> Server 2
                 |
                 +--> Server 3
```

Traffic is spread out, one server going down doesn't take the whole system with it, and maintenance can happen on one node at a time.

---

## 3. How It Works

```text
Client
  |
  | HTTP/HTTPS Request
  v
Load Balancer
  |
  | Select healthy server
  v
Application Server
  |
  v
Response
  |
  v
Client
```

**Step 1 — Client Sends Request** — same request it would send to any server.
**Step 2 — Load Balancer Receives Request** — it becomes the single entry point.
**Step 3 — Load Balancer Selects a Server** — based on an algorithm (see below).
**Step 4 — Request Is Forwarded** — to the chosen backend instance.
**Step 5 — Server Processes Request** — normal application logic runs.
**Step 6 — Response Returns to Client** — usually routed back through the load balancer.

---

## 4. Why Multiple Servers? (Scalability)

```text
100 users
   |
   v
Server 1
```

vs.

```text
100,000 users
      |
      v
Load Balancer
   /    |    \
  v     v     v
 S1    S2    S3
```

This is **horizontal scaling** — adding more machines rather than making one machine bigger. See [[16 - Spring Core - DI, IoC, Beans]] for how stateless Spring beans make this easier, and [[27 - Why You Still Need to Learn the Internals]] for why understanding this matters beyond "just add servers."

---

## 5. Load Balancing Algorithms

### Round Robin
```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```
Works well when all servers have similar capacity.

### Weighted Round Robin
```text
Server 1 → Weight 3
Server 2 → Weight 1
```
Server 1 gets proportionally more traffic — useful when servers have different capacities.

### Least Connections
Send the request to the server with the fewest active connections. Good for long-lived connections.

### IP Hash
Requests from the same client IP are generally routed to the same backend based on a hash — a simple way to get session affinity.

### Random
Select a server randomly — simple, low-overhead distribution.

| Algorithm | Basic Idea | Good For |
|---|---|---|
| Round Robin | Rotate servers | Similar servers |
| Weighted Round Robin | Consider server capacity | Different server capacities |
| Least Connections | Choose least busy server | Long-lived connections |
| IP Hash | Hash client IP | Some session-affinity use cases |
| Random | Random selection | Simple distribution |

---

## 6. Health Checks

> How does a Load Balancer know a server is actually working?

```text
Load Balancer
      |
      | Health Check
      v
   Server 1
      |
      | 200 OK
      v
   Healthy
```

If a server fails to respond:

```text
Server 2
   |
   | Connection failed
   v
Unhealthy
```

The load balancer then stops routing traffic to it:

```text
Load Balancer
      |
      +----> Server 1 ✓
      |
      +----> Server 2 ✗
      |
      +----> Server 3 ✓
```

Key vocabulary: **health check**, **liveness** (is the process alive?), **readiness** (is it ready to accept traffic?), **failure detection**.

Related: [[Spring Boot Actuator]], [[Health Check]]

---

## 7. Types of Load Balancers

### Layer 4 (Transport layer)
Operates on **TCP/UDP** — uses IP address, port, and connection info only. Fast, but "blind" to HTTP content.

### Layer 7 (Application layer)
Operates on **HTTP/HTTPS** — can read URL, HTTP method, headers, cookies, hostname, and route accordingly.

| Feature | Layer 4 | Layer 7 |
|---|---|---|
| Layer | Transport | Application |
| Protocol | TCP/UDP | HTTP/HTTPS |
| Understands URL | No | Yes |
| Content-based routing | Limited | Yes |
| Typical use | High-performance network balancing | HTTP/API routing |

---

## 8. Reverse Proxy vs Load Balancer

```text
Client
  |
  v
Reverse Proxy
  |
  +--> Server 1
  +--> Server 2
```

A **Load Balancer** distributes traffic. A **reverse proxy** is a broader concept that can also provide routing, TLS termination, caching, compression, and security controls. In practice, one component (like NGINX) often performs both roles.

Related: [[Reverse Proxy]], [[NGINX]], [[API Gateway]]

---

## 9. Load Balancer vs API Gateway

| Load Balancer | API Gateway |
|---|---|
| Mainly distributes traffic | Manages API traffic |
| Focuses on availability/scaling | Focuses on API management |
| Health checks | Authentication |
| Routing | Rate limiting |
| Traffic distribution | Request transformation |

Real products often overlap in features.

---

## 10. Sessions and Sticky Sessions

Problem:

```text
Request 1 → Server 1
Request 2 → Server 2
```

What if the user's session only exists on Server 1?

### Sticky Session
```text
User A
  |
  v
Load Balancer
  |
  v
Server 1
```
Future requests from User A keep going to Server 1 — but this reduces the benefit of load balancing and complicates failover.

### The modern, preferred approach — shared session store
```text
Server 1
Server 2
Server 3
      |
      v
Shared Redis Session Store
```

Related: [[Redis]], [[Distributed Session]], [[Stateless Application]]

---

## 11. Stateless Applications

```text
Request
   |
   v
Load Balancer
  / \
 v   v
S1  S2
```

Either server can process the request when the server doesn't depend on local memory for important user state — this is why stateless services work so well behind a load balancer.

---

## 12. HTTPS and TLS Termination

```text
Client
  |
  | HTTPS
  v
Load Balancer
  |
  | HTTP / HTTPS
  v
Spring Boot
```

**TLS termination**: the load balancer handles the encrypted TLS connection from the client, then talks to internal servers over plain HTTP (or re-encrypted HTTPS) inside a trusted network. Discuss: certificates, encryption, internal communication, security considerations.

Related: [[HTTP]], [[HTTPS]], [[TLS]]

---

## 13. Load Balancer in a Production Architecture

```text
                    Internet
                       |
                       v
                 DNS / Route
                       |
                       v
                Load Balancer
                       |
          +------------+------------+
          |            |            |
          v            v            v
     Spring Boot   Spring Boot   Spring Boot
       Server 1      Server 2      Server 3
          |            |            |
          +------------+------------+
                       |
              +--------+--------+
              |                 |
              v                 v
            Redis           PostgreSQL
```

---

## 14. Load Balancer and the Database

> A Load Balancer usually balances application-server traffic. It does **not** automatically solve database scaling.

```text
Load Balancer
      |
  +---+---+
  |       |
 App1    App2
  |       |
  +---+---+
      |
   Database
```

That's a separate problem, solved with [[Database Replication]], [[Read Replica]], [[Database Sharding]].

---

## 15. High Availability

```text
Single Server

Client
  |
  v
Server
  X
```

vs.

```text
High Availability

             +--> Server 1 ✓
Client --> LB|
             +--> Server 2 ✓
```

Multiple instances reduce the impact of any single server failure. Note: a single Load Balancer can itself become a single point of failure, so production architectures often use redundant/managed load-balancing infrastructure.

---

## 16. Scaling

### Vertical Scaling
```text
Server
  ↓
More CPU
More RAM
```

### Horizontal Scaling
```text
Server 1
Server 2
Server 3
```

Load Balancers are strongly associated with horizontal scaling — you can't spread traffic across servers that don't exist yet.

---

## 17. Common Pitfalls

For each: **Problem → Why it happens → Impact → Solution**

1. **Single Point of Failure** — one load balancer instance, no redundancy.
2. **Stateful Application Servers** — local session state breaks failover.
3. **Poor Health Checks** — traffic sent to dead/unready instances.
4. **Wrong Load-Balancing Algorithm** — mismatch between traffic pattern and algorithm.
5. **Connection Limits** — LB or backend runs out of connection slots.
6. **Long-Running Requests** — tie up connections, starve others.
7. **Timeout Configuration** — too aggressive or too lax timeouts cause cascading issues.
8. **Retry Storms** — naive retries amplify load during partial outages.
9. **Ignoring WebSocket Connections** — long-lived connections need special LB handling.
10. **Assuming the Load Balancer Solves Everything** — it doesn't fix database bottlenecks or bad code.

---

## 18. Timeout and Retry

```text
Client
  |
  v
Load Balancer
  |
  v
Server
```

What happens when the server takes too long? Concepts: connection timeout, request timeout, response timeout, retry, exponential backoff.

> Retrying a request is not always safe, especially for non-idempotent operations such as some POST requests.

Related: [[Timeout]], [[Retry]], [[Idempotency]], [[Circuit Breaker]]

---

## 19. WebSockets

```text
HTTP Request/Response      vs.      WebSocket
                                     Client <=================> Server
```

WebSockets are long-lived connections, so load balancing, sticky sessions, and connection management all need special handling compared to normal stateless HTTP requests.

Related: [[WebSocket]]

---

# Spring Boot / Java Notes

## 20. Spring Boot Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public List<User> getUsers() {
        return userService.getUsers();
    }
}
```

```text
Client
 ↓
Load Balancer
 ↓
Spring Boot Instance
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Database
```

This is the exact same `@RestController` / `@GetMapping` pattern from [[22 - Controllers and @RestController]] and [[23 - Creating Our First Endpoint - @GetMapping]] — a load balancer just puts several copies of that whole Spring Boot process behind one address.

## 21. Running Multiple Spring Boot Instances

```text
Instance 1 → Port 8081
Instance 2 → Port 8082
Instance 3 → Port 8083
```

```properties
server.port=8081
```

The Load Balancer distributes traffic among these instances — same idea as [[25 - Changing the Server Port]], just repeated across many machines/processes.

## 22. Spring Boot Actuator

Actuator can expose health information at an endpoint like:

```text
/actuator/health
```

Concepts: liveness, readiness, health checks.

Related: [[Spring Boot Actuator]], [[Health Check]]

---

# 23. Real-World Example

```text
100,000 users
      |
      v
Load Balancer
      |
 +----+----+----+
 |    |    |    |
 v    v    v    v
App1 App2 App3 App4
      |
      v
    Redis
      |
      v
 PostgreSQL
```

---

# 24. Interview Questions

## Beginner
1. What is a Load Balancer?
2. Why do we need a Load Balancer?
3. What is horizontal scaling?
4. What is Round Robin?
5. What is a health check?

## Intermediate
6. Layer 4 vs Layer 7 Load Balancer?
7. Load Balancer vs Reverse Proxy?
8. What is a sticky session?
9. Why are stateless services preferred?
10. How does health checking work?
11. What happens if one server goes down?
12. What is TLS termination?

## Advanced
13. How would you design a highly available Load Balancer architecture?
14. How would you handle WebSocket connections?
15. How would you avoid retry storms?
16. How does Load Balancing affect database architecture?
17. How would you scale an application from 1,000 to 1 million users?
18. What happens if the Load Balancer itself fails?

---

# 25. Scenario-Based Questions

1. Your application has one Spring Boot server and traffic suddenly increases 10x. What would you do?
2. One application instance is down but users are still able to use the application. Explain why.
3. Users randomly lose their login session. What could be wrong?
4. Your Load Balancer reports many 504 errors. How would you investigate?
5. Your API works directly but fails through the Load Balancer. What would you check?
6. One server receives much more traffic than others. What could be the reason?

---

# 26. Debugging Checklist

```text
DNS
 ↓
Load Balancer
 ↓
Health Check
 ↓
Routing
 ↓
Security Group / Firewall
 ↓
TLS
 ↓
Application
 ↓
Database
```

---

# 27. Common Interview Mistakes

❌ "Load Balancer makes the application faster."
✅ "A Load Balancer can improve availability and scalability by distributing traffic across multiple instances."

❌ "Load Balancer is only used for multiple servers."
✅ It can also do TLS termination, health checking, and content-based routing.

❌ "Sticky sessions are always better."
✅ They trade away even distribution and complicate failover — prefer stateless services + shared session store where possible.

❌ "Load Balancer solves database scaling."
✅ Database scaling is a separate concern (replication, sharding, read replicas).

---

# 28. System Design Connections

[[Scalability]] · [[High Availability]] · [[Fault Tolerance]] · [[Caching]] · [[CDN]] · [[Reverse Proxy]] · [[API Gateway]] · [[Microservices]] · [[Distributed Systems]] · [[Database Replication]] · [[Kubernetes]]

```text
                    Users
                      |
                      v
                     CDN
                      |
                      v
                Load Balancer
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
        App 1       App 2       App 3
          |           |           |
          +-----------+-----------+
                      |
                  Redis Cache
                      |
                  PostgreSQL
```

Requests hit a CDN first (for static/cacheable content), fall through to the Load Balancer for dynamic requests, get distributed across app instances, which share a cache and a database.

---

# 29. Quick Revision — 10 Things to Remember

1. A Load Balancer distributes incoming traffic across multiple servers.
2. It exists to fix overload, downtime, and scalability limits of a single server.
3. Common algorithms: Round Robin, Weighted Round Robin, Least Connections, IP Hash, Random.
4. Health checks stop traffic from going to dead servers.
5. Layer 4 = TCP/UDP based; Layer 7 = HTTP-aware, content-based routing.
6. A reverse proxy is a broader concept; a load balancer is one of its jobs.
7. Sticky sessions trade simplicity for reduced load distribution — prefer stateless apps + shared session store.
8. TLS termination lets the load balancer handle HTTPS decryption.
9. A Load Balancer does NOT solve database scaling on its own.
10. A single Load Balancer instance can itself be a single point of failure — plan for redundancy.

---

# 30. Final Mental Model

```text
Client
  |
  v
Load Balancer
  |
  +------> Application 1
  |
  +------> Application 2
  |
  +------> Application 3
              |
              v
           Database
```

> **Load Balancer = distribute incoming traffic across available application instances.**

---

## Related Obsidian Notes
- [[01 - Client, Server & The Request-Response Cycle]]
- [[02 - IP Address vs Port Number]]
- [[14 - Servlet Containers and Tomcat]]
- [[16 - Spring Core - DI, IoC, Beans]]
- [[25 - Changing the Server Port]]
- [[27 - Why You Still Need to Learn the Internals]]
