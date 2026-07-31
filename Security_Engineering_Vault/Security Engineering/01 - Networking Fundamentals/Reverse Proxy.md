# Reverse Proxy

## Definition

A **Reverse Proxy** is a server that sits between **clients and backend servers** and forwards client requests to the appropriate application server.

It acts as a **gateway for incoming traffic** and hides the internal server architecture.

> **Simple Definition**
>
> "A reverse proxy receives user requests, decides where to send them, and returns the response back to the user."

Basic architecture:

```
Client

   ↓

Reverse Proxy

   ↓

Application Servers
```

Examples:

- Nginx
- HAProxy
- Apache HTTP Server
- Cloud Load Balancers

---

# Why it Matters

Modern applications need to handle:

- Millions of users.
- Multiple backend services.
- High availability.
- Security requirements.

A reverse proxy provides:

- Load balancing.
- SSL/TLS termination.
- Security filtering.
- Routing.
- Caching.
- Compression.
- High availability.

---

# Real Enterprise Architecture

A typical MNC production architecture:

```
                 Users

                   |

                   |

             Internet

                   |

                   |

            Reverse Proxy

                   |

          Load Balancer Layer

          /        |        \

      Service1   Service2   Service3

                   |

                Database
```

Users never directly access backend servers.

---

# How it Works

The reverse proxy works as an intermediate layer.

Example request:

```
User requests:

https://api.shop.com/orders
```

Flow:

```
Client

↓

Reverse Proxy

↓

Order Service

↓

Database

↓

Response

↓

Reverse Proxy

↓

Client
```

The client only knows:

```
api.shop.com
```

It does not know:

```
10.20.1.15

10.20.1.16

10.20.1.17
```

---

# Reverse Proxy Request Flow

## Step 1: Client Sends Request

Example:

```
GET /users
```

---

## Step 2: Reverse Proxy Receives Request

It checks:

- URL path.
- Headers.
- Authentication.
- Rules.

---

## Step 3: Routing Decision

Example:

```
/users

↓

User Service


/orders

↓

Order Service


/payment

↓

Payment Service
```

---

## Step 4: Forward Request

Proxy sends request to backend.

Example:

```
Reverse Proxy

↓

Spring Boot Service

Port 8080
```

---

## Step 5: Return Response

Response travels back:

```
Spring Boot

↓

Reverse Proxy

↓

Client
```

---

# Main Features of Reverse Proxy

## 1. Load Balancing

Distributes traffic across multiple servers.

Example:

```
User Request

        ↓

Reverse Proxy


     ↙    ↓    ↘

Server1 Server2 Server3
```

Benefits:

- Better performance.
- No single server overload.
- High availability.

---

## 2. SSL/TLS Termination

Without reverse proxy:

```
Client

HTTPS

↓

Every Server manages certificates
```

With reverse proxy:

```
Client

HTTPS

↓

Reverse Proxy

TLS handled here

↓

Backend Servers
```

Benefits:

- Central certificate management.
- Easier maintenance.

---

## 3. Security Layer

Reverse proxy can block:

- Malicious requests.
- Invalid traffic.
- Suspicious patterns.

Example:

Block:

```
/admin

from unknown IP
```

---

## 4. Routing

Routes requests based on:

- URL.
- Domain.
- Headers.

Example:

```
api.company.com

↓

Backend API


admin.company.com

↓

Admin Service
```

---

## 5. Caching

Stores frequently requested responses.

Example:

First request:

```
Client

↓

Proxy

↓

Server
```

Second request:

```
Client

↓

Proxy Cache
```

Benefits:

- Faster response.
- Reduced backend load.

---

## 6. Compression

Reverse proxy can compress responses.

Example:

```
Large JSON

↓

Compressed Response

↓

Client
```

---

# Reverse Proxy vs Load Balancer

They are related but not exactly the same.

| Reverse Proxy | Load Balancer |
|-|-|
| Receives and forwards requests | Distributes traffic |
| Can provide many features | Mainly traffic distribution |
| Works as gateway | Focuses on availability |
| Often includes load balancing | Usually sits behind proxy |

Modern tools often provide both.

Example:

```
Nginx

= Reverse Proxy + Load Balancer
```

---

# Reverse Proxy vs Forward Proxy

| Reverse Proxy | Forward Proxy |
|-|-|
| Protects servers | Protects clients |
| Incoming traffic | Outgoing traffic |
| User does not know it | Client configures it |
| Enterprise application use | Internet access control |

Memory trick:

```
Forward Proxy:

"Forward my request"


Reverse Proxy:

"Protect my servers"
```

---

# Common Reverse Proxy Technologies

## Nginx

Popular for:

- Reverse proxy.
- Load balancing.
- Static content serving.

---

## HAProxy

Popular for:

- High-performance load balancing.
- Large-scale systems.

---

## Apache HTTP Server

Supports:

- Reverse proxy modules.
- Web serving.

---

## Cloud Solutions

Examples:

- AWS Application Load Balancer.
- Azure Application Gateway.
- Google Cloud Load Balancer.

---

# Common Pitfalls

## ❌ Reverse proxy is only for performance.

Wrong.

It also provides:

- Security.
- Routing.
- TLS management.
- Availability.

---

## ❌ Backend servers should be publicly accessible.

Wrong.

Better architecture:

```
Internet

↓

Reverse Proxy

↓

Private Backend Servers
```

---

## ❌ Reverse proxy automatically solves scalability.

Wrong.

It helps distribute traffic, but applications must also support:

- Database scaling.
- Caching.
- Stateless design.

---

## ❌ SSL termination means communication is insecure.

Not always.

Internal communication can still use:

```
HTTPS

or

mTLS
```

---

## ❌ Reverse proxy and firewall are the same.

Wrong.

Firewall controls access.

Reverse proxy manages application traffic.

---

# Spring Boot / Java Notes

Spring Boot applications commonly run behind reverse proxies in production.

Typical architecture:

```
User

↓

Nginx Reverse Proxy

↓

Spring Boot Application

↓

Database
```

---

# Example Production Flow

User calls:

```
https://api.shop.com/products
```

Flow:

```
Browser

↓

HTTPS Request

↓

Nginx Reverse Proxy

↓

Spring Boot Controller

↓

Service Layer

↓

Repository

↓

Database
```

---

# Nginx with Spring Boot Example

Nginx configuration:

```nginx
server {

    listen 443 ssl;

    server_name api.company.com;


    location / {

        proxy_pass http://localhost:8080;

    }

}
```

Meaning:

```
Incoming HTTPS traffic

        ↓

Forward to Spring Boot

localhost:8080
```

---

# Spring Boot Proxy Awareness

When behind a proxy:

```properties
server.forward-headers-strategy=framework
```

Spring Boot can correctly understand:

- Original client IP.
- HTTPS scheme.
- Forwarded requests.

---

# Important Headers

Reverse proxies add headers:

## X-Forwarded-For

Original client IP.

Example:

```
X-Forwarded-For:
192.168.1.10
```

---

## X-Forwarded-Proto

Original protocol.

Example:

```
https
```

---

## X-Real-IP

Real client IP.

---

# Common Spring Boot Production Issues

## 502 Bad Gateway

Meaning:

Reverse proxy cannot connect to backend.

Check:

- Spring Boot running?
- Correct port?
- Network connectivity?

---

## 504 Gateway Timeout

Meaning:

Backend response is too slow.

Check:

- Database performance.
- API processing time.
- Proxy timeout settings.

---

## Wrong Client IP

Problem:

Application sees:

```
Proxy IP
```

instead of:

```
Real user IP
```

Solution:

Configure forwarded headers.

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

HTTPS
```

Usually caused by incorrect HTTPS configuration.

---

# Security Best Practices

## 1. Hide Backend Servers

Do not expose:

```
Spring Boot Port 8080
```

directly to Internet.

---

## 2. Use HTTPS

Client communication should use:

```
HTTPS + TLS
```

---

## 3. Restrict Backend Access

Only allow:

```
Reverse Proxy

↓

Application Server
```

---

## 4. Enable Logging

Monitor:

- Requests.
- Errors.
- Latency.
- Suspicious traffic.

---

# Quick Revision

- Reverse proxy sits between users and backend servers.
- It hides internal architecture.
- Common responsibilities:
  - Routing.
  - Load balancing.
  - SSL termination.
  - Security.
  - Caching.
- Spring Boot applications commonly run behind reverse proxies.
- Nginx is a popular reverse proxy.
- Backend servers should not be directly exposed to the Internet.

---

# Interview Questions

### 1. What is a reverse proxy?

A reverse proxy is a server that receives client requests and forwards them to backend servers.

---

### 2. Why do we use reverse proxy?

For:

- Load balancing.
- Security.
- Routing.
- SSL termination.
- High availability.

---

### 3. Difference between proxy and reverse proxy?

Proxy protects clients.

Reverse proxy protects servers.

---

### 4. Why hide backend servers?

To reduce attack surface and allow better traffic control.

---

### 5. What is SSL termination?

Decrypting HTTPS traffic at the reverse proxy instead of every backend server.

---

### 6. Is Nginx a reverse proxy?

Yes, Nginx is widely used as a reverse proxy.

---

### 7. How does reverse proxy help microservices?

It routes requests to different services.

Example:

```
/user

↓

User Service


/payment

↓

Payment Service
```

---

### 8. What is X-Forwarded-For?

An HTTP header containing the original client IP when using a proxy.

---

### 9. Difference between reverse proxy and load balancer?

Reverse proxy is a gateway layer.

Load balancer distributes traffic among servers.

Many tools provide both features.

---

### 10. Why should Spring Boot developers understand reverse proxy?

Because production applications are usually deployed behind gateways, load balancers, and reverse proxies.

---

# References

- [[Proxy]]
- [[Load Balancer]]
- [[Firewall]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[DNS]]
- [[NAT]]
- [[VPN]]
- [[Spring Boot Architecture]]