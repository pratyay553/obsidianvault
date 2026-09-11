---
title: "Computer Networks — Index"
aliases:
  - Networking Index
  - Computer Networks Index
  - Network Knowledge Base
tags:
  - index
  - computer-networks
  - networking
  - backend
  - java
  - spring-boot
  - system-design
difficulty: beginner
status: learning
category: computer-networks
---

# 🌐 Computer Networks — Index

> [!abstract] Purpose
> This is the central navigation page for my **Computer Networks & Web Communication** knowledge base.
>
> Learning direction:
>
> **Network Fundamentals → TCP/IP → DNS → HTTP/HTTPS → Security → Backend Communication → Distributed Systems → System Design**

---

# 🧭 Learning Roadmap

```text
Computer Networks
       │
       ├── 01. Fundamentals
       │      ├── Client-Server Communication
       │      ├── Network Basics
       │      ├── Protocols
       │      └── OSI Model
       │
       ├── 02. Network Addressing
       │      ├── IP Addressing
       │      ├── IPv4 / IPv6
       │      ├── Ports
       │      └── NAT
       │
       ├── 03. Transport Layer
       │      ├── TCP
       │      ├── UDP
       │      ├── TCP Handshake
       │      ├── Flow Control
       │      └── Congestion Control
       │
       ├── 04. DNS
       │      ├── Domain Names
       │      ├── DNS Resolution
       │      ├── DNS Records
       │      └── DNS Caching
       │
       ├── 05. Web Communication
       │      ├── HTTP
       │      ├── HTTPS
       │      ├── HTTP Methods
       │      ├── HTTP Headers
       │      ├── HTTP Status Codes
       │      ├── Cookies
       │      └── Sessions
       │
       ├── 06. Security
       │      ├── TLS
       │      ├── Certificates
       │      ├── Authentication
       │      ├── Authorization
       │      ├── JWT
       │      ├── CORS
       │      └── CSRF
       │
       ├── 07. Backend Communication
       │      ├── REST API
       │      ├── Spring MVC
       │      ├── Spring Security
       │      ├── API Gateway
       │      └── Reverse Proxy
       │
       ├── 08. Performance
       │      ├── HTTP Caching
       │      ├── Connection Pooling
       │      ├── Compression
       │      ├── Rate Limiting
       │      └── Timeouts & Retries
       │
       ├── 09. Distributed Systems
       │      ├── Service-to-Service Communication
       │      ├── HTTP vs gRPC
       │      ├── HTTP vs Kafka
       │      ├── WebSocket
       │      └── Message Queues
       │
       └── 10. System Design
              ├── Load Balancing
              ├── Scalability
              ├── Fault Tolerance
              ├── Observability
              └── Production Networking
```

---

# 01 — Network Fundamentals

## Core Concepts

- [[Client-Server Communication, HTTP & HTTPS]]
- [[Computer Network Fundamentals]]
- [[What is a Protocol]]
- [[Network Topologies]]
- [[OSI Model]]
- [[TCP-IP Model]]

> [!tip] Start Here
> Begin with [[Client-Server Communication, HTTP & HTTPS]] if your primary goal is backend development.
>
> Then learn the OSI and TCP/IP models to understand what happens underneath HTTP.

---

# 02 — Network Addressing

## Addressing

- [[IP Addressing]]
- [[IPv4]]
- [[IPv6]]
- [[Public vs Private IP]]
- [[MAC Address]]
- [[Subnetting]]
- [[CIDR]]
- [[NAT]]

## Ports

- [[Network Ports]]
- [[Well Known Ports]]
- [[Socket]]

### Important Ports

| Port | Technology |
|---:|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 9092 | Kafka |

---

# 03 — Transport Layer

## TCP

- [[TCP-IP]]
- [[TCP Three-Way Handshake]]
- [[TCP Connection]]
- [[TCP Reliability]]
- [[TCP Flow Control]]
- [[TCP Congestion Control]]
- [[TCP Keep Alive]]

## UDP

- [[UDP]]
- [[TCP vs UDP]]

## Important Concepts

- [[Socket Programming]]
- [[Connection Timeout]]
- [[Read Timeout]]
- [[Connection Reset]]
- [[Connection Refused]]

---

# 04 — DNS

## Core

- [[DNS]]
- [[DNS Resolution]]
- [[DNS Recursive Resolver]]
- [[Authoritative DNS Server]]
- [[DNS Cache]]
- [[DNS TTL]]

## DNS Records

- [[A Record]]
- [[AAAA Record]]
- [[CNAME Record]]
- [[MX Record]]
- [[TXT Record]]
- [[NS Record]]

### Mental Model

```text
Browser
   ↓
"What is the IP of api.example.com?"
   ↓
DNS Resolver
   ↓
DNS Servers
   ↓
IP Address
   ↓
Client
```

---

# 05 — HTTP & Web Communication

> [!important] Core Backend Topic
> This is one of the most important sections for a Java/Spring Boot developer.

## Main Note

- [[Client-Server Communication, HTTP & HTTPS]]

## HTTP Fundamentals

- [[HTTP]]
- [[HTTP Request]]
- [[HTTP Response]]
- [[HTTP Request Anatomy]]
- [[HTTP Response Anatomy]]
- [[HTTP Methods]]
- [[HTTP URL Structure]]
- [[HTTP Headers]]
- [[HTTP Body]]
- [[HTTP Query Parameters]]
- [[HTTP Path Parameters]]

## HTTP Methods

- [[GET]]
- [[POST]]
- [[PUT]]
- [[PATCH]]
- [[DELETE]]
- [[HEAD]]
- [[OPTIONS]]

## HTTP Status Codes

- [[HTTP Status Codes]]
- [[2xx Success]]
- [[3xx Redirection]]
- [[4xx Client Errors]]
- [[5xx Server Errors]]

### Must Know

```text
200 → OK
201 → Created
204 → No Content

301 → Permanent Redirect
302 → Redirect
304 → Not Modified

400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
409 → Conflict
429 → Too Many Requests

500 → Internal Server Error
502 → Bad Gateway
503 → Service Unavailable
504 → Gateway Timeout
```

---

# 06 — HTTPS & Security

## HTTPS

- [[HTTPS]]
- [[HTTP vs HTTPS]]
- [[TLS]]
- [[TLS Handshake]]
- [[SSL vs TLS]]
- [[TLS Certificate]]
- [[Certificate Authority]]
- [[Certificate Chain]]

## Cryptography

- [[Symmetric Encryption]]
- [[Asymmetric Encryption]]
- [[Public Key]]
- [[Private Key]]
- [[Digital Signature]]
- [[Hashing]]

## Web Authentication

- [[Authentication vs Authorization]]
- [[Cookies and Sessions]]
- [[Session Based Authentication]]
- [[JWT Authentication]]
- [[OAuth 2.0]]
- [[OpenID Connect]]

## Web Security

- [[CORS]]
- [[CSRF]]
- [[XSS]]
- [[SQL Injection]]
- [[Clickjacking]]
- [[Security Headers]]
- [[HTTPS Security Best Practices]]

---

# 07 — Backend Communication

## REST

- [[REST]]
- [[REST API]]
- [[REST API Design]]
- [[Resource Based API Design]]
- [[HTTP Semantics]]
- [[Idempotency]]
- [[API Versioning]]
- [[Pagination]]
- [[Filtering and Sorting]]

## Spring Boot

- [[Spring MVC]]
- [[DispatcherServlet]]
- [[Spring Boot HTTP Request Flow]]
- [[Controller-Service-Repository]]
- [[Request Mapping]]
- [[DTO]]
- [[Validation]]
- [[Global Exception Handling]]
- [[Spring Security]]

### Spring Boot Request Flow

```text
HTTP Request
     ↓
Embedded Tomcat
     ↓
DispatcherServlet
     ↓
Controller
     ↓
Service
     ↓
Repository
     ↓
Database
```

---

# 08 — Proxy, Gateway & Load Balancing

## Reverse Proxy

- [[Reverse Proxy]]
- [[Nginx]]
- [[TLS Termination]]
- [[Proxy vs Reverse Proxy]]

## Load Balancer

- [[Load Balancer]]
- [[Layer 4 Load Balancing]]
- [[Layer 7 Load Balancing]]
- [[Round Robin]]
- [[Least Connections]]
- [[Health Check]]

## API Gateway

- [[API Gateway]]
- [[Spring Cloud Gateway]]
- [[Gateway Authentication]]
- [[Gateway Rate Limiting]]
- [[Gateway Routing]]

### Production Flow

```text
Internet
   ↓
DNS
   ↓
CDN
   ↓
Load Balancer
   ↓
Reverse Proxy / API Gateway
   ↓
Spring Boot
```

---

# 09 — Performance & Reliability

## HTTP Performance

- [[HTTP Keep Alive]]
- [[Connection Pooling]]
- [[HTTP Compression]]
- [[HTTP Caching]]
- [[Cache-Control]]
- [[ETag]]
- [[Conditional Requests]]

## Reliability

- [[Timeout]]
- [[Retry]]
- [[Exponential Backoff]]
- [[Jitter]]
- [[Circuit Breaker]]
- [[Bulkhead Pattern]]
- [[Rate Limiting]]
- [[Idempotency]]

### Important Mental Model

```text
Network Call
     ↓
Timeout
     ↓
Retry?
     ↓
Is operation idempotent?
     ↓
Backoff + Jitter
     ↓
Circuit Breaker
```

---

# 10 — HTTP Versions & Modern Web

- [[HTTP 1.0]]
- [[HTTP 1.1]]
- [[HTTP 2]]
- [[HTTP 3]]
- [[HTTP 1.1 vs HTTP 2]]
- [[HTTP 2 vs HTTP 3]]
- [[QUIC]]

### Evolution

```text
HTTP/1.1
   ↓
HTTP/2
   ↓
HTTP/3
   ↓
QUIC
```

---

# 11 — Real-Time Communication

## WebSocket

- [[WebSocket]]
- [[WebSocket vs HTTP]]
- [[WebSocket Authentication]]
- [[Real Time Communication]]

Useful for:

- Live chat
- GPS tracking
- Notifications
- Live dashboards
- Multiplayer systems

---

# 12 — Service-to-Service Communication

## Synchronous

- [[HTTP Service Communication]]
- [[REST Between Microservices]]
- [[gRPC]]
- [[HTTP vs gRPC]]

```text
Service A
    |
    | HTTP/gRPC
    ↓
Service B
```

## Asynchronous

- [[Kafka]]
- [[Message Queue]]
- [[Event Driven Architecture]]
- [[Kafka vs REST]]
- [[Producer Consumer Model]]

```text
Service A
    |
    | Event
    ↓
Kafka
    |
    +---- Service B
    |
    +---- Service C
```

---

# 13 — Caching & Redis

- [[Caching Fundamentals]]
- [[HTTP Caching]]
- [[Redis]]
- [[Redis Cache]]
- [[Cache Aside Pattern]]
- [[Cache Invalidation]]
- [[Redis TTL]]
- [[Distributed Cache]]

### Request Path

```text
Client
  ↓
API
  ↓
Redis
  |
  +---- Cache Hit → Response
  |
  +---- Cache Miss
          ↓
       Database
          ↓
       Redis
          ↓
       Response
```

---

# 14 — Production Networking

## Infrastructure

- [[CDN]]
- [[Load Balancer]]
- [[Reverse Proxy]]
- [[API Gateway]]
- [[Firewall]]
- [[WAF]]
- [[NAT Gateway]]
- [[VPC]]
- [[Subnet]]
- [[Security Group]]

## Cloud

- [[AWS Networking]]
- [[AWS VPC]]
- [[AWS ALB]]
- [[AWS Route 53]]
- [[AWS CloudFront]]
- [[AWS API Gateway]]

---

# 15 — Docker Networking

Important for backend deployment.

- [[Docker Networking]]
- [[Docker Bridge Network]]
- [[Docker Host Network]]
- [[Docker Container Communication]]
- [[Docker Port Mapping]]
- [[Docker DNS]]

Example:

```text
                 Docker Network
                       |
        +--------------+--------------+
        |              |              |
        ↓              ↓              ↓
    Spring Boot     PostgreSQL      Redis
      :8080           :5432          :6379
```

---

# 16 — Debugging & Troubleshooting

## Tools

- [[curl]]
- [[Postman]]
- [[Browser Network Tab]]
- [[ping]]
- [[traceroute]]
- [[nslookup]]
- [[dig]]
- [[netstat]]
- [[ss]]
- [[telnet]]
- [[nc]]

## Debugging Flow

```text
API Not Working
      ↓
Is DNS resolving?
      ↓
Is port reachable?
      ↓
Is TCP connection working?
      ↓
Is TLS working?
      ↓
Is HTTP request correct?
      ↓
Authentication?
      ↓
Authorization?
      ↓
Application?
      ↓
Database?
      ↓
External dependency?
```

---

# 17 — Common Production Errors

| Problem | First Area to Check |
|---|---|
| Domain not resolving | DNS |
| Connection refused | Server/Port |
| Connection timeout | Network/Server |
| TLS error | Certificate/TLS |
| 400 | Request |
| 401 | Authentication |
| 403 | Authorization |
| 404 | URL/Resource |
| 409 | Resource state |
| 429 | Rate limiting |
| 500 | Application |
| 502 | Gateway/Upstream |
| 503 | Service availability |
| 504 | Upstream timeout |

---

# 18 — System Design Connections

Networking knowledge directly connects to system design.

Study these next:

- [[Scalability]]
- [[Load Balancing]]
- [[Horizontal Scaling]]
- [[Vertical Scaling]]
- [[Caching]]
- [[Database Replication]]
- [[Database Sharding]]
- [[Message Queue]]
- [[Event Driven Architecture]]
- [[Service Discovery]]
- [[Distributed Systems]]
- [[CAP Theorem]]
- [[Consistency]]
- [[Availability]]
- [[Fault Tolerance]]
- [[Observability]]

---

# 19 — Backend Engineer Mental Model

As a Java/Spring Boot developer, I should gradually be able to understand this entire chain:

```text
                    USER
                      |
                      ↓
                  Browser
                      |
                    HTTPS
                      |
                      ↓
                     DNS
                      |
                      ↓
                  IP Address
                      |
                      ↓
                TCP / QUIC
                      |
                      ↓
                     TLS
                      |
                      ↓
                    HTTP
                      |
                      ↓
                CDN / WAF
                      |
                      ↓
              Load Balancer
                      |
                      ↓
               API Gateway
                      |
                      ↓
              Spring Boot API
                      |
             +--------+--------+
             |                 |
             ↓                 ↓
        PostgreSQL           Redis
             |
             ↓
           Kafka
             |
             ↓
      Other Services
```

This is the networking picture I want to build in my mind.

---

# 20 — Interview Preparation

## Beginner

- [[What is Client Server Architecture]]
- [[What is HTTP]]
- [[What is HTTPS]]
- [[What is DNS]]
- [[What is TCP]]
- [[What is a Port]]
- [[What is an HTTP Request]]
- [[What is an HTTP Response]]

## Intermediate

- [[HTTP Methods]]
- [[HTTP Status Codes]]
- [[HTTP Headers]]
- [[Cookies vs Sessions]]
- [[Authentication vs Authorization]]
- [[HTTP vs HTTPS]]
- [[TCP vs UDP]]
- [[HTTP vs WebSocket]]
- [[REST vs gRPC]]
- [[401 vs 403]]
- [[502 vs 504]]
- [[PUT vs PATCH]]
- [[Content-Type vs Accept]]
- [[Idempotency]]

## Advanced

- [[TLS Handshake]]
- [[HTTP 2]]
- [[HTTP 3]]
- [[QUIC]]
- [[Load Balancing]]
- [[Reverse Proxy]]
- [[API Gateway]]
- [[Rate Limiting]]
- [[Circuit Breaker]]
- [[Distributed Systems Networking]]
- [[Service-to-Service Communication]]

---

# 21 — Practical Projects

To make networking knowledge practical, build these progressively.

## Project 1 — Basic HTTP API

```text
Spring Boot
    ↓
GET /hello
    ↓
JSON Response
```

---

## Project 2 — CRUD REST API

```text
Spring Boot
    ↓
REST API
    ↓
PostgreSQL
```

Learn:

- HTTP methods
- Status codes
- DTOs
- Validation
- Exception handling

---

## Project 3 — Authentication

```text
Frontend
   ↓
HTTPS
   ↓
Spring Security
   ↓
JWT / Session
```

Learn:

- Cookies
- JWT
- Authentication
- Authorization
- CORS
- CSRF

---

## Project 4 — Redis

```text
Client
  ↓
Spring Boot
  ↓
Redis
  ↓
PostgreSQL
```

Learn:

- Caching
- TTL
- Cache invalidation
- Performance

---

## Project 5 — Kafka

```text
Order Service
      ↓
     Kafka
      ↓
Notification Service
      ↓
Email/SMS
```

Learn:

- Asynchronous communication
- Events
- Producers
- Consumers
- Event-driven architecture

---

## Project 6 — Production Deployment

```text
Internet
   ↓
DNS
   ↓
HTTPS
   ↓
Nginx
   ↓
Docker
   ↓
Spring Boot
   ↓
PostgreSQL
   ↓
Redis
```

Learn:

- DNS
- TLS certificates
- Reverse proxy
- Docker networking
- Ports
- Production debugging

---

# 22 — Current Core Note

The first completed note in this knowledge base is:

> [[Client-Server Communication, HTTP & HTTPS]]

This note covers:

```text
Client
Server
Protocol
HTTP
HTTPS
DNS
TCP
TLS
Request
Response
Headers
Methods
Status Codes
Cookies
Sessions
JWT
REST
Spring Boot
CORS
Caching
Timeouts
Retries
Idempotency
Load Balancer
Reverse Proxy
System Design
Production Debugging
```

---

# 23 — Learning Progress

## Fundamentals

- [ ] Client-Server Communication
- [ ] Protocols
- [ ] OSI Model
- [ ] TCP/IP Model

## Networking

- [ ] IP Addressing
- [ ] Ports
- [ ] TCP
- [ ] UDP
- [ ] TCP Handshake
- [ ] DNS

## Web

- [ ] HTTP
- [ ] HTTP Methods
- [ ] HTTP Headers
- [ ] HTTP Status Codes
- [ ] Cookies
- [ ] Sessions
- [ ] REST API

## Security

- [ ] HTTPS
- [ ] TLS
- [ ] Certificates
- [ ] Authentication
- [ ] Authorization
- [ ] JWT
- [ ] CORS
- [ ] CSRF

## Backend

- [ ] Spring MVC
- [ ] Spring Security
- [ ] Reverse Proxy
- [ ] API Gateway
- [ ] Load Balancer

## Distributed Systems

- [ ] HTTP Service Communication
- [ ] gRPC
- [ ] WebSocket
- [ ] Kafka
- [ ] Event Driven Architecture

## Production

- [ ] Docker Networking
- [ ] AWS Networking
- [ ] CDN
- [ ] WAF
- [ ] Rate Limiting
- [ ] Timeouts
- [ ] Retries
- [ ] Circuit Breaker
- [ ] Observability

---

# 24 — The Golden Rule

> **Do not learn networking as isolated definitions. Learn it as the journey of one request.**

Always ask:

```text
Where does the request start?

        ↓

How does the client find the server?

        ↓

How does the connection get established?

        ↓

How is the connection secured?

        ↓

How does HTTP carry the request?

        ↓

How does Spring Boot receive it?

        ↓

How does the application process it?

        ↓

How does it reach PostgreSQL/Redis/Kafka?

        ↓

How does the response come back?

        ↓

What happens if something fails?
```

If I can answer these questions clearly, I am building **real backend engineering knowledge**, not just memorising interview definitions.

---

# 🔗 Related Knowledge Areas

[[Java]]

[[Spring Boot]]

[[Spring Security]]

[[PostgreSQL]]

[[Redis]]

[[Kafka]]

[[Docker]]

[[AWS]]

[[Distributed Systems]]

[[System Design]]

[[Cyber Security]]

---

# ⭐ Most Important Notes to Master First

1. [[Client-Server Communication, HTTP & HTTPS]]
2. [[OSI Model]]
3. [[TCP]]
4. [[TCP Three-Way Handshake]]
5. [[DNS]]
6. [[HTTP Methods]]
7. [[HTTP Headers]]
8. [[HTTP Status Codes]]
9. [[TLS]]
10. [[REST API]]
11. [[Spring MVC]]
12. [[Spring Security]]
13. [[Reverse Proxy]]
14. [[Load Balancer]]
15. [[Caching]]
16. [[Distributed Systems]]

---

# 🧠 Final Mental Model

```text
                    COMPUTER NETWORKS
                           |
        +------------------+------------------+
        |                  |                  |
     ADDRESSING         TRANSPORT          APPLICATION
        |                  |                  |
        IP                TCP/UDP          HTTP/HTTPS
        |                  |                  |
       DNS             Connections          REST
        |                  |                  |
        +------------------+------------------+
                           |
                       SECURITY
                           |
                     TLS / Auth
                           |
                      BACKEND
                           |
                Spring Boot / APIs
                           |
                +----------+----------+
                |          |          |
             Database     Redis      Kafka
                |
                ↓
         Distributed Systems
                |
                ↓
          System Design
```

> [!quote] Core Principle
> **A strong backend engineer should not only know how to write a REST controller. They should understand the journey of the request before it reaches the controller, what happens while it is processed, and how the response travels back to the client.**