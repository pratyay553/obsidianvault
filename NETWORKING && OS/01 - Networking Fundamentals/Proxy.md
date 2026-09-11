# Proxy

## Definition

A **Proxy Server** is an intermediate server that sits between a **client and a destination server** and forwards requests on behalf of the client.

It acts as a **middle layer** that controls, filters, and manages communication.

> **Simple Definition**
>
> "A proxy is a middle server that receives a request from a client and sends it to the destination server."

Basic flow:

```
Client

   ↓

Proxy Server

   ↓

Destination Server
```

---

# Why it Matters

Proxy servers are used in enterprise systems for:

- Security.
- Privacy.
- Access control.
- Monitoring.
- Caching.
- Traffic management.

Examples:

- Company employees accessing the Internet.
- API requests passing through gateways.
- Microservices communication.
- Hiding internal servers.

---

# Real World Example

Imagine a company employee accessing a website.

Without proxy:

```
Employee

↓

Internet

↓

Website
```

With proxy:

```
Employee

↓

Company Proxy

↓

Internet

↓

Website
```

The website sees the proxy's IP instead of the employee's IP.

---

# How it Works

A proxy receives requests and forwards them.

Example:

User wants:

```
https://example.com
```

Flow:

```
Client

Request

↓

Proxy Server

Request Forwarded

↓

Example Server

Response

↓

Proxy Server

Response

↓

Client
```

The client does not directly communicate with the destination server.

---

# Proxy Components

## Client

The user application making the request.

Examples:

- Browser.
- Mobile app.
- API client.

---

## Proxy Server

The intermediate server.

Responsibilities:

- Receive requests.
- Apply rules.
- Forward requests.
- Return responses.

---

## Destination Server

The final application or website.

---

# Types of Proxy

## 1. Forward Proxy

A forward proxy works on behalf of the client.

Architecture:

```
Client

↓

Forward Proxy

↓

Internet

↓

Server
```

Used for:

- Employee Internet access.
- Content filtering.
- Privacy.

Example:

Company blocks social media using a proxy.

---

## 2. Reverse Proxy

A reverse proxy works on behalf of the server.

Architecture:

```
Client

↓

Reverse Proxy

↓

Application Servers
```

Used for:

- Load balancing.
- SSL termination.
- Security.
- Routing.

Examples:

- Nginx.
- HAProxy.
- Cloud Load Balancers.

---

# Forward Proxy vs Reverse Proxy

| Forward Proxy | Reverse Proxy |
|-|-|
| Protects clients | Protects servers |
| Client knows proxy | Client usually does not know |
| Used for outbound traffic | Used for inbound traffic |
| User-side | Server-side |

Easy memory:

```
Forward Proxy:

"Forward my request"


Reverse Proxy:

"Protect my servers"
```

---

# Proxy Functions

## 1. Request Forwarding

Proxy receives:

```
Client Request
```

and sends:

```
Server Request
```

---

## 2. IP Hiding

Example:

Client:

```
192.168.1.20
```

Proxy:

```
45.10.20.30
```

Website sees:

```
45.10.20.30
```

---

## 3. Caching

Proxy can store frequently requested data.

Example:

First request:

```
Proxy → Server
```

Second request:

```
Proxy → Cache Response
```

Benefits:

- Faster response.
- Less server load.

---

## 4. Access Control

Example:

Block:

```
facebook.com
```

Allow:

```
company.com
```

---

## 5. Logging

Proxy can record:

- User requests.
- URLs.
- Response time.
- Errors.

---

# Reverse Proxy in Enterprise Architecture

Modern applications commonly use:

```
Internet

↓

Reverse Proxy

↓

Load Balancer

↓

Spring Boot Services

↓

Database
```

---

# Common Reverse Proxy Responsibilities

## SSL/TLS Termination

Client:

```
HTTPS
```

↓

Reverse Proxy decrypts

↓

Internal communication:

```
HTTP or HTTPS
```

---

## Load Balancing

Distributes traffic:

```
User Request

       ↓

Reverse Proxy

   ↙       ↓       ↘

Server1 Server2 Server3
```

---

## Security Layer

Blocks:

- Malicious requests.
- Invalid traffic.
- Suspicious patterns.

---

# Proxy vs Firewall

| Proxy | Firewall |
|-|-|
| Handles application requests | Controls network traffic |
| Acts as middle server | Filters packets |
| Can modify requests | Allows/blocks connections |
| Layer 7 commonly | Layer 3/4 commonly |

---

# Proxy vs VPN

| Proxy | VPN |
|-|-|
| Application-level forwarding | Network-level tunnel |
| Usually one application | Entire device traffic |
| No complete encryption by default | Encrypts traffic |
| Faster/simple | More secure |

---

# Common Pitfalls

## ❌ Proxy and Reverse Proxy are the same.

Wrong.

Forward proxy protects clients.

Reverse proxy protects servers.

---

## ❌ Proxy provides complete security.

Wrong.

Proxy is only one security layer.

You still need:

- Firewall.
- Authentication.
- Encryption.
- Monitoring.

---

## ❌ Reverse proxy replaces load balancer.

Not always.

Many reverse proxies provide load balancing, but dedicated load balancers may provide additional features.

---

## ❌ Proxy always improves performance.

Wrong.

A poorly configured proxy can introduce latency.

---

## ❌ HTTPS traffic cannot pass through proxy.

Wrong.

HTTPS can work through proxies using:

- TLS termination.
- TLS passthrough.

---

# Spring Boot / Java Notes

Spring Boot applications commonly run behind a reverse proxy.

Production architecture:

```
Client

↓

Nginx / Load Balancer

↓

Spring Boot Application

↓

Database
```

---

# Example Request Flow

User calls:

```
https://api.company.com/users
```

Flow:

```
Browser

↓

Reverse Proxy

↓

Spring Boot Controller

↓

Service Layer

↓

Database

↓

Response
```

---

# Nginx Reverse Proxy Example

Example:

```
server {

listen 443;

location /api {

proxy_pass http://springboot-server:8080;

}

}
```

Meaning:

```
Incoming:

/api

↓

Forward to:

Spring Boot port 8080
```

---

# Spring Boot Behind Proxy

Common configuration:

```properties
server.forward-headers-strategy=framework
```

Allows Spring Boot to understand:

- Original client IP.
- HTTPS information.
- Forwarded headers.

---

# Common Production Issues

## Wrong Client IP

Problem:

Application sees:

```
Proxy IP
```

instead of:

```
Real User IP
```

Solution:

Use:

```
X-Forwarded-For
```

header.

---

## Redirect Loop

Example:

```
HTTPS

↓

Proxy

↓

HTTP

↓

HTTPS again
```

Usually caused by incorrect proxy configuration.

---

## 502 Bad Gateway

Possible causes:

- Spring Boot server down.
- Wrong port.
- Network issue.

---

## 504 Gateway Timeout

Possible causes:

- Application response too slow.
- Proxy timeout too low.

---

# Quick Revision

- Proxy is an intermediate server.
- Forward proxy works for clients.
- Reverse proxy works for servers.
- Reverse proxy commonly provides:
  - Load balancing.
  - SSL termination.
  - Security.
- Proxy can provide caching and access control.
- Spring Boot applications commonly run behind reverse proxies.
- Nginx is a popular reverse proxy.

---

# Interview Questions

### 1. What is a proxy server?

A proxy server is an intermediate server that forwards requests between clients and destination servers.

---

### 2. What is the difference between forward proxy and reverse proxy?

Forward proxy represents clients.

Reverse proxy represents servers.

---

### 3. Why use a reverse proxy?

For:

- Load balancing.
- Security.
- SSL termination.
- Routing.

---

### 4. Is Nginx a proxy server?

Yes. Nginx is commonly used as a reverse proxy.

---

### 5. Can proxy improve performance?

Yes, using caching and traffic optimization.

---

### 6. Does proxy provide encryption?

Not necessarily. HTTPS/TLS provides encryption.

---

### 7. What is SSL termination?

The proxy decrypts HTTPS traffic and forwards requests internally.

---

### 8. Why do Spring Boot applications use reverse proxies?

To hide application servers, manage traffic, handle TLS, and improve scalability.

---

### 9. Difference between proxy and firewall?

Firewall controls traffic access.

Proxy forwards and manages application requests.

---

### 10. What headers help identify the original client behind a proxy?

Common headers:

```
X-Forwarded-For

X-Real-IP
```

---

# References

- [[Reverse Proxy]]
- [[Firewall]]
- [[Load Balancer]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[TCP-IP]]
- [[NAT]]
- [[VPN]]