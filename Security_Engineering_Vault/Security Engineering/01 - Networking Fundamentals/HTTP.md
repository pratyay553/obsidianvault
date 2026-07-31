# HTTP (HyperText Transfer Protocol)

## Definition

**HTTP (HyperText Transfer Protocol)** is an **Application Layer protocol** used for communication between a **client and server** over a network.

It defines how:

- Requests are sent.
- Responses are returned.
- Data is transferred between applications.

> **Simple Definition**
>
> "HTTP is the language that browsers and servers use to communicate with each other."

Example:

```
Browser

↓

HTTP Request

↓

Web Server

↓

HTTP Response

↓

Browser
```

---

## Why it Matters

Almost every modern application uses HTTP.

Examples:

- Websites
- REST APIs
- Mobile applications
- Microservices
- Cloud applications

When you open:

```
https://amazon.com
```

or call:

```
GET /users
```

HTTP is responsible for communication.

For a Spring Boot developer, HTTP is the foundation of:

- REST API development.
- Client-server communication.
- Microservice communication.
- API debugging.

---

## How it Works

HTTP follows a simple:

```
Request → Response
```

model.

The client sends a request.

The server processes it.

The server sends a response.

Example:

```
Client

GET /products

        ↓

Server

200 OK

Product List
```

---

# HTTP Request Structure

An HTTP request contains:

```
Request Line

Headers

Body
```

Example:

```
GET /users HTTP/1.1

Host: api.example.com

Authorization: Bearer token

Content-Type: application/json


{
 "id":10
}
```

---

## HTTP Methods

HTTP methods define what action the client wants.

| Method | Purpose | Example |
|-|-|-|
| GET | Retrieve data | Get users |
| POST | Create data | Create order |
| PUT | Update complete data | Update user |
| PATCH | Update partial data | Change email |
| DELETE | Remove data | Delete account |
| HEAD | Get headers only | Check resource |

---

## HTTP Response Structure

Server sends:

```
Status Line

Headers

Body
```

Example:

```
HTTP/1.1 200 OK

Content-Type: application/json


[
 {
  "id":1,
  "name":"John"
 }
]
```

---

# HTTP Status Codes

Status codes tell the client what happened.

## 1xx - Information

Example:

```
100 Continue
```

---

## 2xx - Success

| Code | Meaning |
|-|-|
| 200 | OK |
| 201 | Created |
| 204 | No Content |

Example:

```
User created successfully
```

---

## 3xx - Redirection

| Code | Meaning |
|-|-|
| 301 | Permanent redirect |
| 302 | Temporary redirect |
| 304 | Not modified |

Example:

```
HTTP → HTTPS redirect
```

---

## 4xx - Client Error

| Code | Meaning |
|-|-|
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |

Example:

```
Invalid input

or

Missing permission
```

---

## 5xx - Server Error

| Code | Meaning |
|-|-|
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

Example:

```
Application crashed
```

---

# HTTP Headers

Headers provide additional information.

Examples:

## Content-Type

Defines data format.

Example:

```
Content-Type: application/json
```

---

## Authorization

Used for authentication.

Example:

```
Authorization: Bearer JWT_TOKEN
```

---

## Cookie

Stores client information.

Example:

```
Cookie: sessionId=12345
```

---

## Cache-Control

Controls caching.

Example:

```
Cache-Control: max-age=3600
```

---

# HTTP is Stateless

HTTP does not remember previous requests.

Example:

Request 1:

```
Login
```

Request 2:

```
Get Profile
```

Server does not automatically know the user.

To maintain state, we use:

- Cookies
- Sessions
- JWT tokens

---

# HTTP Versions

## HTTP/1.0

Basic request-response communication.

---

## HTTP/1.1

Introduced:

- Persistent connections.
- Better caching.
- Host headers.

Still widely used.

---

## HTTP/2

Improvements:

- Multiplexing.
- Header compression.
- Faster communication.

---

## HTTP/3

Uses:

```
QUIC over UDP
```

Advantages:

- Faster connection setup.
- Better performance on unstable networks.

---

# HTTP vs HTTPS

| HTTP | HTTPS |
|-|-|
| No encryption | Encrypted |
| Port 80 | Port 443 |
| Data visible | Secure data |
| No TLS | Uses TLS |

Example:

HTTP:

```
GET password=123
```

Anyone intercepting can read it.

HTTPS:

```
Encrypted Data
```

Only the server can decrypt it.

---

# Common Pitfalls

## ❌ HTTP and HTTPS are the same.

Wrong.

HTTPS is HTTP + TLS encryption.

---

## ❌ HTTP guarantees security.

Wrong.

HTTP sends data in plain text.

Use HTTPS for security.

---

## ❌ HTTP is a Transport Layer protocol.

Wrong.

HTTP works at:

```
Application Layer
```

TCP works at:

```
Transport Layer
```

---

## ❌ GET should modify data.

Wrong.

GET should only retrieve information.

---

## ❌ POST is always secure.

Wrong.

POST only sends data.

Security depends on HTTPS and authentication.

---

## ❌ HTTP maintains user sessions automatically.

Wrong.

HTTP is stateless.

Sessions require:

- Cookies.
- Tokens.
- Server-side storage.

---

# Spring Boot / Java Notes

Spring Boot is heavily based on HTTP.

A REST controller:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping
    public List<User> getUsers(){
        return service.getUsers();
    }
}
```

Creates an HTTP endpoint:

```
GET /users
```

---

## Request Flow in Spring Boot

```
Client

↓

HTTP Request

↓

Tomcat Server

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

↓

HTTP Response
```

---

## HTTP Methods in Spring Boot

GET:

```java
@GetMapping("/users")
```

POST:

```java
@PostMapping("/users")
```

PUT:

```java
@PutMapping("/users/{id}")
```

DELETE:

```java
@DeleteMapping("/users/{id}")
```

---

## Common Spring Boot HTTP Problems

### 404 Not Found

Possible reasons:

- Wrong URL.
- Wrong controller mapping.
- Endpoint does not exist.

---

### 400 Bad Request

Possible reasons:

- Invalid JSON.
- Validation failure.
- Missing required fields.

---

### 401 Unauthorized

Possible reasons:

- Missing token.
- Invalid authentication.

---

### 403 Forbidden

Possible reason:

- User authenticated but lacks permission.

---

### 500 Internal Server Error

Possible reasons:

- Application bug.
- Database failure.
- Null pointer exception.

---

## Quick Revision

- HTTP is an Application Layer protocol.
- HTTP follows Request → Response model.
- HTTP is stateless.
- Common methods:
  - GET
  - POST
  - PUT
  - PATCH
  - DELETE
- HTTPS = HTTP + TLS.
- HTTP normally uses TCP.
- HTTP ports:
  - 80 (HTTP)
  - 443 (HTTPS)
- REST APIs are built using HTTP.

---

# Interview Questions

### 1. What is HTTP?

HTTP is an Application Layer protocol used for communication between clients and servers.

---

### 2. How does HTTP work?

Client sends a request, server processes it, and returns a response.

---

### 3. Is HTTP stateful or stateless?

HTTP is stateless.

---

### 4. What are common HTTP methods?

GET, POST, PUT, PATCH, DELETE.

---

### 5. Difference between GET and POST?

GET retrieves data.

POST creates or sends data.

---

### 6. What is the difference between HTTP and HTTPS?

HTTPS uses TLS encryption to secure HTTP communication.

---

### 7. Which port does HTTP use?

Port 80.

---

### 8. Which port does HTTPS use?

Port 443.

---

### 9. What happens when you type a URL in a browser?

Flow:

```
DNS Lookup

↓

TCP Handshake

↓

TLS Handshake (HTTPS)

↓

HTTP Request

↓

Server Response
```

---

### 10. Why should a Spring Boot developer understand HTTP?

Because REST APIs, microservices, authentication, and client-server communication are built on HTTP.

---

## References

- [[OSI Model]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[DNS]]
- [[HTTPS]]
- [[TLS]]
- [[REST API]]
- [[Firewall]]
- [[Reverse Proxy]]
- [[Load Balancer]]