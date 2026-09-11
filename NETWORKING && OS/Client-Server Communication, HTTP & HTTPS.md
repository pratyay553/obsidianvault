---
title: "Client-Server Communication, HTTP & HTTPS"
aliases:
  - HTTP
  - HTTPS
  - HTTP Protocol
  - HTTPS Protocol
  - Client Server Communication
tags:
  - computer-networks
  - http
  - https
  - networking
  - backend
  - web
  - java
  - spring-boot
  - system-design
difficulty: beginner
status: learning
category: computer-networks
---

# Client-Server Communication, HTTP & HTTPS

> [!abstract] Learning Goal
> Understand how a browser, mobile application, or another backend service communicates with a server over a network using DNS, TCP, TLS, HTTP and HTTPS — and understand how this maps to Java/Spring Boot applications.

---

# 1. One-Line Definition

## Client

A **client** is a program or device that requests a service or resource from another system.

Examples:

- Web browser
- Mobile application
- React/Next.js frontend
- Postman
- Another backend service
- `curl`

For example, when you open:

```text
https://example.com/products
```

your browser is acting as the **client**.

---

## Server

A **server** is a program or computer that waits for requests, processes them, and sends responses.

For a Spring Boot application:

```text
Browser
   |
   | HTTP Request
   ↓
Spring Boot Application
   |
   | Business Logic
   ↓
Database
   |
   ↓
Spring Boot Application
   |
   | HTTP Response
   ↓
Browser
```

The word "server" can mean two related things:

### Physical/virtual machine

A computer running somewhere in a data centre or cloud.

### Server application

A program running on that computer and listening for requests.

For example:

```text
AWS EC2 / Cloud Server
        |
        ↓
Linux
        |
        ↓
JVM
        |
        ↓
Spring Boot Application
        |
        ↓
Port 8080
```

So when we say:

> "The server received the request"

we usually mean the **server application** received it.

---

# 2. What Is Communication?

Communication simply means:

> One system sends information to another system.

For example:

```text
Client
  |
  | "Give me product #101"
  ↓
Server
  |
  | "Here is product #101"
  ↓
Client
```

The client and server need a common set of rules to understand each other.

Those rules are called a **protocol**.

---

# 3. What Is a Protocol?

A protocol is a predefined set of rules that tells systems:

- How to communicate
- What format to use
- What information should be included
- How messages should be interpreted
- What should happen when something goes wrong

A simple analogy is ordering food.

```text
Customer → Restaurant

Customer:
"One chicken biryani, please."

Restaurant:
"Order received."

Restaurant:
"Your order is ready."

Customer:
"Thank you."
```

Both sides understand the meaning because they follow common rules.

Computers work similarly.

---

# 4. Client-Server Communication

The basic model is:

```text
             Network
               
Client --------------------> Server
       Request

Client <-------------------- Server
       Response
```

The client normally starts the conversation.

The server waits for requests.

For a web application:

```text
Browser
   |
   | Request
   ↓
Internet
   |
   ↓
Web Server
   |
   ↓
Application
   |
   ↓
Database
```

The response travels back:

```text
Database
   |
   ↓
Application
   |
   ↓
Web Server
   |
   ↓
Internet
   |
   ↓
Browser
```

---

# 5. Real-World Analogy

Imagine a restaurant.

```text
Customer       = Client
Restaurant     = Server
Waiter         = Communication layer
Menu/order     = Request
Prepared food  = Response
Restaurant rules = Protocol
```

The customer says:

> "Give me one biryani."

The restaurant processes the request.

Then:

> "Here is your biryani."

Similarly:

```text
Client
  |
  | GET /products/101
  ↓
Server
  |
  | Find product
  ↓
Database
  |
  | Product found
  ↓
Server
  |
  | 200 OK + JSON
  ↓
Client
```

---

# 6. What Is HTTP?

## HTTP = HyperText Transfer Protocol

HTTP is an application-layer protocol used for communication between clients and servers.

It defines how requests and responses should look.

Very simply:

```text
Client
  |
  | HTTP Request
  ↓
Server
  |
  | HTTP Response
  ↓
Client
```

HTTP tells both sides:

- How to create a request
- How to create a response
- How to identify the requested resource
- How to represent data
- How to indicate success/failure
- How to send metadata
- How to manage caching
- How to handle authentication-related information

---

# 7. HTTP Is Not the Internet

This is an important distinction.

HTTP is **one protocol used over the Internet**.

The Internet is the huge network infrastructure that connects computers.

HTTP is one communication rule used by applications.

Think:

```text
Internet
   |
   +---- HTTP
   |
   +---- HTTPS
   |
   +---- DNS
   |
   +---- SMTP
   |
   +---- SSH
```

Each protocol solves a different problem.

---

# 8. HTTP Request and Response

HTTP communication mainly consists of:

```text
REQUEST
   ↓
SERVER PROCESSING
   ↓
RESPONSE
```

---

## HTTP Request

A request contains information such as:

- HTTP method
- URL/path
- Headers
- Optional body

Example:

```http
GET /api/products/101 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer <token>
```

---

## HTTP Response

The server responds with:

- Status code
- Headers
- Optional body

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 101,
  "name": "Gold Ring",
  "price": 45000
}
```

---

# 9. Anatomy of an HTTP Request

A request can be visualised as:

```text
HTTP Request
│
├── Method
├── URL
├── Headers
└── Body
```

Example:

```http
POST /api/orders HTTP/1.1
Host: shop.example.com
Content-Type: application/json
Authorization: Bearer abc123

{
  "productId": 101,
  "quantity": 2
}
```

---

# 10. HTTP Method

The method tells the server what the client wants to do.

Common methods:

| Method | Typical Meaning |
|---|---|
| GET | Retrieve data |
| POST | Create/process data |
| PUT | Replace/update a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |
| HEAD | Get headers without response body |
| OPTIONS | Ask what communication options are supported |

---

# 11. GET

GET is commonly used to retrieve information.

Example:

```http
GET /api/products/101
```

Meaning:

> "Server, please give me product 101."

Example response:

```json
{
  "id": 101,
  "name": "Gold Ring",
  "price": 45000
}
```

GET requests should generally not change server state.

---

# 12. POST

POST is commonly used to create a new resource or trigger server-side processing.

Example:

```http
POST /api/orders
```

Body:

```json
{
  "productId": 101,
  "quantity": 2
}
```

The server may create:

```text
Order #5001
```

---

# 13. PUT

PUT generally means replacing the representation of an existing resource.

Example:

```http
PUT /api/users/101
```

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Think:

> "Replace/update this resource with this representation."

---

# 14. PATCH

PATCH is normally used for partial modification.

Example:

```http
PATCH /api/users/101
```

```json
{
  "email": "new@example.com"
}
```

Only the specified field needs to change.

---

# 15. DELETE

DELETE is used to remove a resource.

```http
DELETE /api/products/101
```

Meaning:

> "Delete product 101."

---

# 16. HTTP URL

Consider:

```text
https://api.example.com:443/products/101?currency=INR
```

Break it down:

```text
https://
   ↓
Scheme

api.example.com
   ↓
Host

:443
   ↓
Port

/products/101
   ↓
Path

?currency=INR
   ↓
Query parameter
```

---

# 17. Domain Name

A domain such as:

```text
example.com
```

is human-friendly.

Computers ultimately communicate using IP addresses.

For example:

```text
example.com
      ↓
DNS
      ↓
203.0.113.10
```

The actual IP is determined through DNS resolution.

---

# 18. DNS

## DNS = Domain Name System

DNS translates domain names into IP addresses.

Without DNS, users would need to remember IP addresses.

Instead of:

```text
https://142.250.x.x
```

we use:

```text
https://example.com
```

Conceptually:

```text
Browser
   |
   | "What IP belongs to example.com?"
   ↓
DNS
   |
   | "Here is the IP"
   ↓
Browser
```

DNS is not HTTP.

DNS helps the client discover **where the server is**.

---

# 19. Port

A computer can run multiple network services.

Ports help identify which service should receive traffic.

Common ports:

| Port | Common Usage |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 9092 | Kafka commonly |

Example:

```text
IP Address
192.168.1.10

Port 80
→ HTTP server

Port 443
→ HTTPS server

Port 5432
→ PostgreSQL
```

A port is not a physical socket.

It is a logical endpoint used by networking software.

---

# 20. What Actually Happens When You Open a Website?

This is one of the most important flows to understand.

Suppose you enter:

```text
https://shop.example.com
```

A simplified journey is:

```text
1. Browser reads URL
        ↓
2. DNS resolution
        ↓
3. Server IP discovered
        ↓
4. TCP connection established
        ↓
5. TLS handshake
        ↓
6. HTTPS request sent
        ↓
7. Server processes request
        ↓
8. Server sends HTTPS response
        ↓
9. Browser receives response
        ↓
10. Browser renders page
```

Let's understand each step.

---

# 21. Step 1 — Browser Reads the URL

Browser sees:

```text
https://shop.example.com/products
```

It understands:

```text
Protocol = HTTPS
Host     = shop.example.com
Path     = /products
```

---

# 22. Step 2 — DNS Resolution

The browser needs the server's IP address.

Conceptually:

```text
shop.example.com
        ↓
DNS lookup
        ↓
IP address
```

There may be caching at several levels.

Possible places include:

```text
Browser cache
      ↓
Operating system cache
      ↓
Router / local resolver
      ↓
DNS resolver
      ↓
Authoritative DNS server
```

The exact path depends on the environment.

---

# 23. Step 3 — TCP Connection

Once the client knows the destination IP and port, a network connection can be established.

For traditional HTTP/1.1 and HTTP/2 over TLS, TCP is commonly involved.

HTTPS commonly uses:

```text
TCP
  ↓
TLS
  ↓
HTTP
```

The TCP connection begins with a handshake.

---

# 24. TCP Three-Way Handshake

A simplified TCP handshake:

```text
Client                  Server

  SYN  ------------------>

       <------------------ SYN-ACK

  ACK  ------------------>
```

Meaning roughly:

```text
Client:
"Can we establish a connection?"

Server:
"Yes. I received you and I am ready."

Client:
"Okay."
```

TCP provides reliable, ordered byte-stream delivery.

---

# 25. Step 4 — TLS Handshake

If the URL uses HTTPS, TLS is established.

Conceptually:

```text
TCP connection
      ↓
TLS handshake
      ↓
Secure encrypted channel
      ↓
HTTP communication
```

TLS helps provide:

1. Encryption
2. Server authentication
3. Integrity protection

---

# 26. HTTP vs HTTPS

## HTTP

```text
HTTP
  ↓
Data can travel without TLS encryption
```

## HTTPS

```text
HTTP
  +
TLS
  ↓
Encrypted HTTP communication
```

A useful mental model:

```text
HTTP = communication protocol

HTTPS = HTTP carried securely using TLS
```

HTTPS is not a completely different application protocol from HTTP.

It is essentially HTTP over a secure TLS connection.

---

# 27. Why Do We Need HTTPS?

Imagine sending:

```text
username = pratyay
password = myPassword
```

over an unencrypted connection.

A malicious party able to observe the traffic could potentially read sensitive information.

HTTPS protects the communication channel using encryption.

Conceptually:

```text
Client
  |
  | encrypted traffic
  ↓
Internet
  |
  | encrypted traffic
  ↓
Server
```

An attacker may see network traffic exists, but should not be able to simply read the protected HTTP contents.

---

# 28. Three Main Security Goals of TLS

Remember:

```text
TLS
│
├── Confidentiality
├── Integrity
└── Authentication
```

## Confidentiality

Others should not be able to read the protected data.

Example:

```text
password
credit card data
session information
```

---

## Integrity

Data should not be silently modified during transmission.

For example:

```text
Original:
amount = ₹1000

Attacker tries:
amount = ₹9000
```

TLS provides mechanisms to detect tampering.

---

## Authentication

The client can verify the identity of the server through the certificate and trust chain.

For example:

```text
Client
   |
   | "Are you really shop.example.com?"
   ↓
Server Certificate
   ↓
Certificate validation
   ↓
Trusted Certificate Authority
```

---

# 29. What Is a TLS Certificate?

A TLS certificate is a digitally signed document containing information associated with a domain and its public key.

It helps the client verify that the server is authorised for that domain.

Simplified:

```text
Certificate
│
├── Domain information
├── Public key
├── Validity information
└── Digital signature from issuer
```

---

# 30. Certificate Authority

A **Certificate Authority (CA)** is an organisation trusted to issue certificates.

Examples include well-known public CAs such as:

- Let's Encrypt
- DigiCert
- GlobalSign

The browser/operating system maintains a set of trusted root certificates.

Conceptually:

```text
Trusted Root CA
      ↓
Intermediate CA
      ↓
Website Certificate
      ↓
example.com
```

The browser validates the certificate chain.

---

# 31. Public Key and Private Key

TLS uses public-key cryptography as part of establishing trust and secure communication.

Very simplified:

```text
Public Key
→ Can be shared

Private Key
→ Must remain secret
```

The private key should stay on the server side and must be protected carefully.

Do not think:

> "HTTPS encrypts everything using one public key."

That is an oversimplification.

Modern TLS uses asymmetric cryptography for authentication/key establishment and symmetric cryptography for efficient bulk encryption.

---

# 32. Symmetric Encryption

Symmetric encryption uses the same secret key for encryption and decryption.

```text
Message
   |
   | encrypt using secret key
   ↓
Encrypted data
   |
   | decrypt using same secret key
   ↓
Message
```

It is efficient and therefore suitable for encrypting large amounts of application data.

---

# 33. Why Not Use Public-Key Encryption for Everything?

Public-key cryptography is computationally more expensive.

A simplified TLS mental model is:

```text
Asymmetric cryptography
        ↓
Securely establish/authenticate keys
        ↓
Symmetric session keys
        ↓
Encrypt application traffic efficiently
```

This hybrid approach gives both security and performance.

---

# 34. Step 5 — HTTP Request

After the secure connection is established, the client sends an HTTP request.

Example:

```http
GET /api/products/101 HTTP/1.1
Host: api.shop.com
Accept: application/json
Authorization: Bearer <token>
```

With HTTPS, the HTTP request is carried inside the encrypted TLS connection.

---

# 35. Step 6 — Server Receives Request

The request may travel through infrastructure such as:

```text
Internet
   ↓
Load Balancer
   ↓
Reverse Proxy
   ↓
API Gateway
   ↓
Spring Boot Application
```

Not every system has every component.

A simple application may be:

```text
Client
   ↓
Spring Boot
```

A large production system may look like:

```text
Client
   ↓
CDN
   ↓
Load Balancer
   ↓
API Gateway
   ↓
Service
   ↓
Database / Redis / Kafka
```

---

# 36. Step 7 — Spring Boot Processes the Request

Suppose the client sends:

```http
GET /api/products/101
```

A Spring Boot application may process it approximately like:

```text
HTTP Request
     ↓
Embedded Web Server
     ↓
Spring MVC
     ↓
Controller
     ↓
Service
     ↓
Repository
     ↓
PostgreSQL
```

Then:

```text
PostgreSQL
     ↓
Repository
     ↓
Service
     ↓
Controller
     ↓
JSON Response
```

---

# 37. Spring Boot Controller Example

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public ProductResponse getProduct(@PathVariable Long id) {
        return productService.getProduct(id);
    }
}
```

A request:

```http
GET /api/products/101
```

can reach:

```java
getProduct(101)
```

The controller is the HTTP-facing entry point for this endpoint.

---

# 38. Step 8 — Server Sends Response

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 101,
  "name": "Gold Ring",
  "price": 45000
}
```

The response contains a status code and may contain headers and a body.

---

# 39. HTTP Response Structure

Think:

```text
HTTP Response
│
├── Status Code
├── Headers
└── Body
```

Example:

```http
HTTP/1.1 200 OK

Content-Type: application/json
Cache-Control: max-age=60

{
  "message": "Success"
}
```

---

# 40. HTTP Headers

Headers provide metadata about the request or response.

Think of headers as:

> Additional information attached to the HTTP message.

Examples:

```http
Content-Type: application/json
Authorization: Bearer token
Accept: application/json
User-Agent: Mozilla/5.0
Cache-Control: no-cache
Cookie: sessionId=abc123
```

---

# 41. Important Request Headers

## Host

Identifies the target host.

```http
Host: api.example.com
```

---

## Accept

Tells the server which response formats the client can understand.

```http
Accept: application/json
```

---

## Content-Type

Tells the server what format the request body uses.

```http
Content-Type: application/json
```

Example:

```json
{
  "name": "Rahul"
}
```

---

## Authorization

Carries authentication credentials such as a bearer token.

```http
Authorization: Bearer eyJ...
```

---

## User-Agent

Provides information about the client software.

```http
User-Agent: Mozilla/5.0
```

---

# 42. Important Response Headers

## Content-Type

```http
Content-Type: application/json
```

Means:

> The response body is JSON.

---

## Content-Length

Can indicate the size of the message body.

```http
Content-Length: 1520
```

---

## Cache-Control

Controls caching behaviour.

```http
Cache-Control: max-age=3600
```

---

## Set-Cookie

Server can ask the client to store a cookie.

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure
```

---

# 43. HTTP Body

The body contains actual data when a request or response needs one.

Example request:

```http
POST /api/users
Content-Type: application/json

{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

The JSON is the request body.

Example response:

```json
{
  "id": 101,
  "name": "Rahul"
}
```

The JSON is the response body.

---

# 44. Path Parameters

Example:

```text
GET /api/products/101
```

Here:

```text
101
```

is a path parameter.

Spring Boot:

```java
@GetMapping("/{id}")
public ProductResponse get(@PathVariable Long id) {
    ...
}
```

---

# 45. Query Parameters

Example:

```text
GET /api/products?category=ring&page=2
```

Here:

```text
category=ring
page=2
```

are query parameters.

They are commonly used for:

- Filtering
- Searching
- Sorting
- Pagination

Example:

```text
/api/products?page=2&size=20&sort=price
```

---

# 46. HTTP Status Codes

Status codes tell the client what happened.

They are grouped into five classes:

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client-side error
5xx → Server-side error
```

Remember:

```text
2xx = generally successful
4xx = request/problem from client side
5xx = server-side failure
```

This is a useful interview-level mental model, but real systems can have more nuanced cases.

---

# 47. Important 2xx Codes

## 200 OK

Request succeeded.

```http
200 OK
```

Common for successful GET requests.

---

## 201 Created

A resource was successfully created.

```http
201 Created
```

Common after successful POST creation.

---

## 204 No Content

Request succeeded but there is no response body.

```http
204 No Content
```

Common for certain DELETE or update operations.

---

# 48. Important 3xx Codes

## 301 Moved Permanently

Resource has permanently moved.

## 302 Found

Historically used for temporary redirection, with behaviour depending on client/context.

## 304 Not Modified

Useful with HTTP caching.

It can mean:

> The cached version can still be used.

---

# 49. Important 4xx Codes

## 400 Bad Request

Request is invalid.

Example:

```json
{
  "age": "hello"
}
```

when the API expects a number.

---

## 401 Unauthorized

The request lacks valid authentication credentials.

Common example:

```text
Missing/invalid authentication
```

---

## 403 Forbidden

The server understood the request, but the authenticated client is not allowed to perform the operation.

Example:

```text
CUSTOMER tries to access ADMIN endpoint
```

---

## 404 Not Found

Requested resource/endpoint was not found.

```http
GET /api/products/999999
```

could result in:

```http
404 Not Found
```

---

## 409 Conflict

Request conflicts with the current state of the resource.

Example:

```text
Trying to create an account with an email
that already exists.
```

---

## 429 Too Many Requests

Client has sent too many requests within a given period.

Commonly associated with:

```text
Rate limiting
```

---

# 50. Important 5xx Codes

## 500 Internal Server Error

Generic server-side error.

Example:

```text
Unexpected NullPointerException
```

---

## 502 Bad Gateway

A gateway/proxy received an invalid response from an upstream server.

Example:

```text
Client
  ↓
Nginx
  ↓
Spring Boot
```

If Nginx cannot get a valid response from Spring Boot, 502 may occur.

---

## 503 Service Unavailable

Service is temporarily unable to handle requests.

Possible reasons:

- Service down
- Maintenance
- Overload
- Dependency failure

---

## 504 Gateway Timeout

A gateway/proxy did not receive a timely response from an upstream service.

---

# 51. Cookies

A cookie is small data stored by the client and associated with a website/domain.

Example:

```http
Set-Cookie: sessionId=abc123
```

The browser stores it.

Later it may send:

```http
Cookie: sessionId=abc123
```

This allows HTTP applications to maintain state across multiple requests.

---

# 52. Why Are Cookies Needed?

HTTP is fundamentally stateless at the protocol level.

Imagine:

```text
Request 1:
"Login as Rahul"

Request 2:
"Give me my profile"

Request 3:
"Give me my orders"
```

Without some mechanism to associate requests with the same user, the server would not automatically know that all three requests belong to the same authenticated session.

Cookies can help solve this.

---

# 53. Session-Based Authentication

A traditional session flow:

```text
Browser
   |
   | POST /login
   ↓
Server
   |
   | Create session
   ↓
Session Store
   |
   ↓
Set-Cookie: JSESSIONID=abc
   |
   ↓
Browser
```

Later:

```text
Browser
   |
   | Cookie: JSESSIONID=abc
   ↓
Server
   |
   | Find session
   ↓
Session Store
   |
   ↓
User identified
```

---

# 54. JWT-Based Authentication

Another common approach is a token.

Example:

```http
Authorization: Bearer <JWT>
```

Flow:

```text
Client
   |
   | Login
   ↓
Server
   |
   | JWT
   ↓
Client
```

Later:

```text
Client
   |
   | Authorization: Bearer JWT
   ↓
Server
   |
   | Validate token
   ↓
Controller
```

JWT and session-based authentication solve related authentication problems in different ways.

Do not assume:

> JWT = automatically more secure than sessions.

Security depends heavily on implementation, storage, expiration, revocation strategy, transport security, and application design.

---

# 55. Important Cookie Security Flags

## HttpOnly

```text
HttpOnly
```

Helps prevent JavaScript from directly reading the cookie.

Useful against certain cookie-theft scenarios involving XSS.

---

## Secure

```text
Secure
```

Cookie should be sent only over secure HTTPS connections.

---

## SameSite

Controls when cookies are sent in cross-site contexts.

Common values:

```text
Strict
Lax
None
```

`SameSite=None` generally requires `Secure`.

---

# 56. HTTP Is Stateless

This is an important backend concept.

HTTP itself does not automatically remember previous requests.

For example:

```text
Request 1 → Server
Request 2 → Server
Request 3 → Server
```

Each request is an independent HTTP message.

Applications introduce mechanisms such as:

- Cookies
- Sessions
- Tokens
- Databases
- Caches

to maintain application-level state.

---

# 57. REST API

REST APIs commonly use HTTP as their communication protocol.

Example:

```text
GET    /api/products
GET    /api/products/101
POST   /api/products
PUT    /api/products/101
PATCH  /api/products/101
DELETE /api/products/101
```

The HTTP method expresses the intended operation.

The URL identifies the resource.

---

# 58. Example REST API

Imagine a salon application.

```text
GET    /api/services
GET    /api/services/10
POST   /api/services
PATCH  /api/services/10
DELETE /api/services/10
```

A request:

```http
GET /api/services/10
Accept: application/json
```

Response:

```http
200 OK
Content-Type: application/json

{
  "id": 10,
  "name": "Haircut",
  "price": 500
}
```

---

# 59. HTTP and Spring Boot

Spring Boot provides the infrastructure needed to build HTTP APIs.

Typical architecture:

```text
Client
  ↓
HTTP
  ↓
Embedded Server
  ↓
Spring MVC
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
```

Example:

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

---

# 60. What Happens Inside Spring Boot?

Suppose:

```http
GET /api/users/101
```

A simplified flow:

```text
Browser/Postman
      ↓
TCP/TLS
      ↓
Embedded Tomcat
      ↓
Servlet processing
      ↓
DispatcherServlet
      ↓
Controller mapping
      ↓
UserController
      ↓
UserService
      ↓
UserRepository
      ↓
PostgreSQL
```

Response:

```text
PostgreSQL
      ↓
Repository
      ↓
Service
      ↓
Controller
      ↓
Jackson JSON serialization
      ↓
HTTP Response
```

---

# 61. JSON

REST APIs frequently use JSON.

Example:

```json
{
  "id": 101,
  "name": "Rahul",
  "age": 25
}
```

JSON is a data representation format.

Important:

> JSON is not a network protocol.

HTTP is the protocol.

JSON is commonly the format used inside HTTP request/response bodies.

---

# 62. Content-Type vs Accept

This is a common interview question.

### Content-Type

Answers:

> "What format is the data I am sending?"

Example:

```http
Content-Type: application/json
```

### Accept

Answers:

> "What response format can I accept?"

Example:

```http
Accept: application/json
```

Memory trick:

```text
Content-Type
→ What I SEND

Accept
→ What I WANT BACK
```

---

# 63. HTTP Versions

Important HTTP versions:

```text
HTTP/1.0
HTTP/1.1
HTTP/2
HTTP/3
```

You do not need to master every implementation detail initially.

Understand the progression.

---

# 64. HTTP/1.1

HTTP/1.1 became widely used and introduced important features such as persistent connections.

Multiple requests can reuse a TCP connection.

But HTTP/1.1 has limitations around request/response multiplexing.

---

# 65. HTTP/2

HTTP/2 improves efficiency using mechanisms including:

- Binary framing
- Multiplexing
- Header compression
- Stream prioritisation mechanisms

One TCP connection can carry multiple concurrent HTTP streams.

Conceptually:

```text
One TCP connection
       |
       +---- Request A
       |
       +---- Request B
       |
       +---- Request C
```

This can reduce some inefficiencies associated with HTTP/1.1.

---

# 66. HTTP/3

HTTP/3 uses:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
```

Instead of:

```text
HTTP/1.1 / HTTP/2
   ↓
TCP
```

QUIC provides transport features such as reliable delivery and stream multiplexing while operating over UDP.

You do not need to implement QUIC to understand HTTP/3.

---

# 67. Important Protocol Stack

A useful mental model:

```text
Application Layer
        |
      HTTP
        |
   TLS (HTTPS)
        |
      TCP
        |
       IP
        |
  Network/Link
```

For HTTP without TLS:

```text
HTTP
 ↓
TCP
 ↓
IP
```

For HTTPS over HTTP/1.1 or HTTP/2:

```text
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP
```

For HTTP/3:

```text
HTTP/3
 ↓
QUIC
 ↓
UDP
 ↓
IP
```

---

# 68. DNS vs TCP vs TLS vs HTTP

This distinction is extremely important.

| Technology | Main Job |
|---|---|
| DNS | Find the IP address for a domain |
| TCP | Reliable transport connection |
| TLS | Secure/authenticate the connection |
| HTTP | Application-level web communication |
| IP | Addressing and routing packets |

Memory model:

```text
DNS
"Where is the server?"

TCP
"Let's establish reliable communication."

TLS
"Let's make the communication secure."

HTTP
"Here is the actual web request."
```

---

# 69. Complete HTTPS Request Journey

Remember this diagram:

```text
                 CLIENT
                   |
                   |
              URL entered
                   |
                   ↓
                 DNS
                   |
             Find server IP
                   |
                   ↓
                 TCP
                   |
          Establish connection
                   |
                   ↓
                 TLS
                   |
        Secure/authenticate channel
                   |
                   ↓
                HTTP
                   |
            Send HTTP request
                   |
                   ↓
              Load Balancer
                   |
                   ↓
             Spring Boot
                   |
                   ↓
               Service
                   |
                   ↓
              PostgreSQL
                   |
                   ↓
             Spring Boot
                   |
                   ↓
             HTTP Response
                   |
                   ↓
                  TLS
                   |
                   ↓
                  TCP
                   |
                   ↓
                Browser
```

This is one of the most useful diagrams to memorise.

---

# 70. What Happens When You Call a REST API?

Suppose your frontend calls:

```text
https://api.salon.com/api/bookings/101
```

The simplified flow is:

```text
Next.js / React
       |
       | HTTPS
       ↓
DNS
       ↓
IP address
       ↓
TCP
       ↓
TLS
       ↓
HTTP GET
       ↓
Load Balancer
       ↓
Spring Boot
       ↓
Security Filter
       ↓
Controller
       ↓
Service
       ↓
Repository
       ↓
PostgreSQL
       ↓
Response
       ↓
JSON
       ↓
Frontend
```

---

# 71. Authentication in the HTTP Pipeline

A Spring Security application may process the request approximately like:

```text
HTTP Request
     ↓
Security Filters
     ↓
Authentication
     ↓
Authorization
     ↓
Controller
```

For example:

```http
GET /api/admin/users
Authorization: Bearer <JWT>
```

Spring Security may:

```text
Extract JWT
    ↓
Validate JWT
    ↓
Identify user
    ↓
Check authorities
    ↓
Allow / deny request
```

---

# 72. Authentication vs Authorization

## Authentication

Answers:

> "Who are you?"

Example:

```text
User = Rahul
```

## Authorization

Answers:

> "What are you allowed to do?"

Example:

```text
Rahul = CUSTOMER

/api/admin/users
→ Access denied
```

Memory trick:

```text
Authentication
→ WHO?

Authorization
→ WHAT ARE YOU ALLOWED TO DO?
```

---

# 73. CORS

CORS = Cross-Origin Resource Sharing.

It becomes important when browser-based frontend and backend have different origins.

Example:

```text
Frontend:
https://app.example.com

Backend:
https://api.example.com
```

The browser enforces same-origin security rules.

The server can provide CORS response headers to indicate which cross-origin requests are allowed.

Example:

```http
Access-Control-Allow-Origin: https://app.example.com
```

Important:

> CORS is primarily a browser security mechanism. It is not a general server-to-server security mechanism.

---

# 74. Same-Origin

An origin consists conceptually of:

```text
scheme + host + port
```

For example:

```text
https://example.com:443
```

Changing any of these can produce a different origin.

Examples:

```text
https://example.com
http://example.com
https://api.example.com
https://example.com:8443
```

These can represent different origins.

---

# 75. Common HTTP Problems

When an API fails, do not randomly change code.

First identify:

```text
DNS?
 ↓
TCP?
 ↓
TLS?
 ↓
HTTP?
 ↓
Application?
 ↓
Database?
```

This gives you a systematic debugging path.

---

# 76. Debugging DNS Problems

Symptoms:

```text
Domain cannot be resolved
```

Possible causes:

- Wrong DNS record
- Domain configuration problem
- DNS propagation/caching issue
- Resolver problem

Tools:

```bash
nslookup example.com
```

or:

```bash
dig example.com
```

---

# 77. Debugging Connectivity Problems

Check whether the destination is reachable and whether the relevant port is available.

Useful tools:

```bash
ping example.com
```

and:

```bash
curl -v https://example.com
```

Remember:

> Ping uses ICMP and does not prove that HTTP/HTTPS is working.

A server may block ICMP while HTTP works normally.

---

# 78. Debugging HTTP with curl

Example:

```bash
curl -v https://api.example.com/users/101
```

The verbose option can help you inspect connection and HTTP details.

For a POST:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"Rahul"}' \
  https://api.example.com/users
```

---

# 79. Browser Developer Tools

When debugging frontend API problems, open:

```text
Browser
→ Developer Tools
→ Network
```

Inspect:

- Request URL
- Method
- Status
- Request headers
- Response headers
- Request payload
- Response body
- Cookies
- Timing
- CORS-related information

This is one of the most useful tools for a full-stack developer.

---

# 80. Common Errors and First Questions

### 400

Ask:

```text
Is my request malformed?
Is validation failing?
```

### 401

Ask:

```text
Did I send authentication?
Is the token valid/expired?
```

### 403

Ask:

```text
Is the user authenticated but lacking permission?
```

### 404

Ask:

```text
Is the URL correct?
Does the resource exist?
```

### 409

Ask:

```text
Is there a business/state conflict?
```

### 429

Ask:

```text
Did I hit rate limiting?
```

### 500

Ask:

```text
What exception occurred on the server?
```

Check application logs.

### 502

Ask:

```text
Did the proxy/load balancer receive a valid response from upstream?
```

### 503

Ask:

```text
Is the service unavailable or overloaded?
```

### 504

Ask:

```text
Did an upstream service take too long?
```

---

# 81. HTTP Request Debugging Checklist

When an API does not work:

```text
1. Check URL
2. Check DNS
3. Check port
4. Check HTTP method
5. Check request headers
6. Check authentication
7. Check request body
8. Check query parameters
9. Check status code
10. Check response body
11. Check Spring Boot logs
12. Check database
13. Check external dependencies
```

---

# 82. Production Architecture

A production Spring Boot application may look like:

```text
                   Internet
                      |
                      ↓
                    DNS
                      |
                      ↓
                    CDN
                      |
                      ↓
               Load Balancer
                      |
              +-------+-------+
              |               |
              ↓               ↓
        Spring Boot A   Spring Boot B
              |               |
              +-------+-------+
                      |
                PostgreSQL
                      |
                  Redis
                      |
                   Kafka
```

The exact architecture depends on the system.

---

# 83. Where HTTP Fits in System Design

When designing a distributed system, HTTP often becomes the communication mechanism between components.

Example:

```text
Frontend
   |
 HTTPS
   ↓
API Gateway
   |
 HTTP/HTTPS
   ↓
User Service
   |
 HTTP/gRPC
   ↓
Order Service
```

Other technologies may be used depending on requirements:

```text
HTTP/REST
gRPC
Kafka
WebSocket
```

Choosing communication technology is a system-design decision.

---

# 84. HTTP vs Kafka

Do not confuse synchronous request/response communication with asynchronous messaging.

### HTTP

```text
Client
  |
  | Request
  ↓
Server
  |
  | Response
  ↓
Client
```

Usually request-response oriented.

### Kafka

```text
Producer
   |
   | Event
   ↓
Kafka
   |
   +---- Consumer A
   |
   +---- Consumer B
   |
   +---- Consumer C
```

Kafka is designed for event streaming and asynchronous communication.

---

# 85. HTTP vs WebSocket

HTTP commonly follows:

```text
Request
   ↓
Response
```

WebSocket provides a persistent connection that allows both sides to send messages after the connection is established.

Useful for:

- Live chat
- Real-time dashboards
- Notifications
- Live tracking

For example, a GPS tracking application may use:

```text
Vehicle
   ↓
GPS data
   ↓
Backend
   ↓
WebSocket
   ↓
Live dashboard
```

---

# 86. HTTP vs gRPC

REST/HTTP APIs are common for:

```text
Frontend ↔ Backend
Public APIs
Browser communication
```

gRPC is often useful for:

```text
Service ↔ Service
```

It commonly uses HTTP/2 as its transport foundation and Protocol Buffers for message definitions.

Do not treat REST and gRPC as universally competing choices.

The correct choice depends on the communication requirements.

---

# 87. HTTP Caching

HTTP supports caching mechanisms.

Example:

```http
Cache-Control: max-age=3600
```

A browser or intermediary cache may reuse a response according to the caching rules.

Caching can reduce:

- Latency
- Server load
- Bandwidth usage

In production systems, caching can exist at multiple layers:

```text
Browser cache
     ↓
CDN cache
     ↓
Reverse proxy cache
     ↓
Application cache
     ↓
Redis
```

---

# 88. Idempotency

An operation is idempotent if making the same operation multiple times has the same intended final effect as making it once.

Common HTTP semantics:

```text
GET     → generally idempotent
PUT     → generally idempotent
DELETE  → generally idempotent
POST    → generally NOT idempotent
```

Example:

```text
PUT /users/101
```

Setting:

```json
{
  "name": "Rahul"
}
```

repeatedly should leave the resource in the same intended state.

---

# 89. Why Idempotency Matters in Payments

Imagine:

```text
POST /payments
```

The client sends the request.

Network timeout occurs.

The client does not know whether payment succeeded.

If it blindly retries:

```text
POST /payments
```

the system could potentially process the payment twice.

A common solution is an **idempotency key**.

Example:

```http
Idempotency-Key: payment-123456
```

The server can recognise retries of the same logical operation.

This is an important production/backend concept.

---

# 90. HTTP Timeouts

Never assume network calls will always respond immediately.

A request may experience:

```text
DNS delay
TCP connection delay
TLS handshake delay
Server processing delay
Database delay
Network delay
```

Production applications should use appropriate timeouts.

Examples:

```text
Connection timeout
Read timeout
Request timeout
```

Without proper timeout handling, threads/connections can remain occupied too long and cause cascading failures.

---

# 91. Retries

Retries can help with temporary failures.

But blindly retrying every failure is dangerous.

For example:

```text
POST payment
   ↓
timeout
   ↓
retry
   ↓
duplicate payment
```

Therefore, retry strategy should consider:

- Idempotency
- Error type
- Maximum attempts
- Backoff
- Jitter
- Timeout budget

---

# 92. Rate Limiting

A server may limit how many requests a client can make.

Example:

```text
100 requests/minute
```

If the limit is exceeded:

```http
429 Too Many Requests
```

Rate limiting helps protect:

- APIs
- Databases
- Services
- Infrastructure

from excessive traffic.

---

# 93. Reverse Proxy

A reverse proxy sits between clients and backend servers.

Example:

```text
Client
  ↓
Nginx
  ↓
Spring Boot
```

It can provide:

- TLS termination
- Routing
- Load balancing
- Compression
- Caching
- Security controls

---

# 94. TLS Termination

In some architectures, HTTPS terminates at the load balancer or reverse proxy.

Example:

```text
Client
   |
 HTTPS
   ↓
Load Balancer
   |
 HTTP / HTTPS
   ↓
Spring Boot
```

This is called **TLS termination** at the load balancer.

Whether internal traffic should also use TLS depends on the security architecture.

For sensitive production systems, encrypting traffic between internal services can also be important.

---

# 95. HTTP Security Checklist

For a production web application:

```text
HTTPS enabled
Secure cookies
HttpOnly cookies where appropriate
SameSite configured appropriately
Authentication implemented correctly
Authorization enforced server-side
Input validation
Output encoding where relevant
CSRF protection where applicable
CORS configured carefully
Rate limiting
Security headers
Dependency updates
Secrets protected
Logs monitored
```

HTTPS alone does not make an application secure.

---

# 96. HTTP Does Not Mean "Only Websites"

HTTP can be used for communication between:

```text
Browser → Backend
Mobile → Backend
Backend → Backend
IoT → Backend
CLI → API
Postman → API
```

Example:

```text
Order Service
      |
      | HTTP
      ↓
Payment Service
```

So HTTP is broader than just browser pages.

---

# 97. Practical Spring Boot Exercise

Build this endpoint:

```http
GET /api/hello
```

Response:

```json
{
  "message": "Hello from Spring Boot"
}
```

Controller:

```java
@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public Map<String, String> hello() {
        return Map.of(
            "message",
            "Hello from Spring Boot"
        );
    }
}
```

Call it:

```bash
curl http://localhost:8080/api/hello
```

Observe:

```text
HTTP request
     ↓
Spring Boot
     ↓
Controller
     ↓
JSON
     ↓
HTTP response
```

---

# 98. Practical Exercise — POST

Create:

```http
POST /api/users
```

Request:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Observe:

```text
Request method
Request URL
Content-Type
Request body
Response status
Response body
```

Use:

- Postman
- curl
- Browser Developer Tools

---

# 99. Practical Exercise — Debugging

Intentionally create:

```text
404
401
403
400
500
```

Then investigate each one.

Example:

```text
Wrong URL
   ↓
404

Missing token
   ↓
401

Insufficient role
   ↓
403

Invalid JSON
   ↓
400

Unexpected server exception
   ↓
500
```

This is much more valuable than memorising status codes alone.

---

# 100. Mental Model to Remember

When you type:

```text
https://api.example.com/users/101
```

think:

```text
URL
 ↓
DNS
 ↓
IP
 ↓
TCP / QUIC
 ↓
TLS (for HTTPS)
 ↓
HTTP
 ↓
Load Balancer / Proxy
 ↓
Spring Boot
 ↓
Security
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Database
 ↓
Response
 ↓
Client
```

---

# 101. Most Important Distinctions

Memorise these:

```text
DNS
→ Finds IP

IP
→ Provides addressing/routing

TCP
→ Reliable transport

UDP
→ Datagram transport

TLS
→ Secure communication + authentication/integrity mechanisms

HTTP
→ Application-level request/response protocol

HTTPS
→ HTTP over TLS

JSON
→ Data representation format

REST
→ Architectural style commonly implemented using HTTP

Spring Boot
→ Application framework used to implement backend services
```

---

# 102. Interview Questions

## Beginner

### Q1. What is client-server architecture?

A model where a client requests services/resources and a server processes those requests and returns responses.

### Q2. What is HTTP?

HTTP is an application-layer protocol used for communication between clients and servers.

### Q3. What is HTTPS?

HTTPS is HTTP carried over TLS, providing encrypted communication and server authentication mechanisms.

### Q4. What is DNS?

DNS resolves domain names to IP addresses.

### Q5. What is a port?

A logical network endpoint used to identify a service on a host.

### Q6. What is an HTTP request?

A structured message sent by a client to a server.

### Q7. What is an HTTP response?

A structured message returned by a server to a client.

---

# 103. Intermediate Interview Questions

### Q8. What happens when you enter a URL into a browser?

Explain:

```text
URL parsing
→ DNS
→ connection establishment
→ TLS for HTTPS
→ HTTP request
→ server processing
→ HTTP response
→ browser processing/rendering
```

### Q9. Difference between HTTP and HTTPS?

HTTP does not provide TLS protection.

HTTPS uses TLS to protect HTTP communication and authenticate the server.

### Q10. Difference between 401 and 403?

```text
401
→ authentication credentials are missing/invalid

403
→ server understood the request but access is forbidden
```

### Q11. Difference between Content-Type and Accept?

```text
Content-Type
→ format of message body being sent

Accept
→ response formats the client can accept
```

### Q12. Is HTTP stateful?

HTTP itself is stateless; applications use cookies, sessions, tokens and other mechanisms to maintain state.

### Q13. Why is HTTPS required?

To protect confidentiality and integrity of application traffic and authenticate the server.

### Q14. Why does HTTPS use TLS?

TLS provides the security layer that protects HTTP communication.

---

# 104. Advanced Interview Questions

### Q15. Explain TLS handshake.

At a high level:

```text
Client
  ↓
Negotiates TLS parameters
  ↓
Server provides certificate
  ↓
Certificate validated
  ↓
Key establishment
  ↓
Secure session established
  ↓
Encrypted application data
```

Exact details differ between TLS versions and negotiated cipher suites.

### Q16. Why does TLS use symmetric encryption for application data?

Because symmetric cryptography is efficient for bulk data encryption.

### Q17. Why are certificates needed?

To help authenticate the server's identity and bind a domain identity to a public key through a trusted certificate chain.

### Q18. What is TLS termination?

When TLS encryption is terminated at a proxy/load balancer instead of directly at the backend application.

### Q19. What is idempotency?

An operation is idempotent when repeating it produces the same intended final state as performing it once.

### Q20. Why are timeouts important?

Without timeouts, slow or unavailable dependencies can hold resources for too long and cause cascading failures.

---

# 105. System Design Connection

HTTP knowledge becomes extremely important when designing distributed systems.

Suppose you build a food-delivery platform:

```text
                    Client
                      |
                    HTTPS
                      |
                Load Balancer
                      |
                 API Gateway
                      |
          +-----------+-----------+
          |           |           |
       User        Order       Payment
       Service     Service      Service
          |           |           |
          +-----------+-----------+
                      |
                   Redis
                      |
                 PostgreSQL
                      |
                    Kafka
```

You must think about:

- HTTP latency
- Connection reuse
- TLS overhead
- Timeouts
- Retries
- Rate limiting
- Load balancing
- Authentication
- Authorization
- Caching
- Idempotency
- Observability
- Circuit breakers
- Service-to-service communication

This is where networking knowledge becomes system-design knowledge.

---

# 106. Production Debugging Mental Model

When something fails in production:

```text
                    REQUEST FAILED
                         |
          +--------------+--------------+
          |              |              |
         DNS           Network        HTTP
          |              |              |
     Resolution      TCP/TLS        4xx/5xx
          |              |              |
          +--------------+--------------+
                         |
                    Application
                         |
                     Database
                         |
                  External Services
```

Do not immediately assume:

> "Spring Boot is broken."

The problem can be anywhere in the chain.

---

# 107. Obsidian Knowledge Graph

Recommended related notes:

```text
Client-Server Communication
        |
        +---- DNS
        |
        +---- IP Addressing
        |
        +---- TCP
        |
        +---- UDP
        |
        +---- TLS
        |
        +---- HTTP
        |
        +---- HTTPS
        |
        +---- REST API
        |
        +---- Spring MVC
        |
        +---- Spring Security
        |
        +---- Cookies
        |
        +---- JWT
        |
        +---- CORS
        |
        +---- Load Balancer
        |
        +---- Reverse Proxy
        |
        +---- API Gateway
        |
        +---- Redis
        |
        +---- Kafka
        |
        +---- Distributed Systems
```

In Obsidian, these can eventually become separate linked notes.

---

# 108. What You Should Learn Next

After this topic, follow this sequence:

```text
1. Computer Network Fundamentals
        ↓
2. IP Addressing
        ↓
3. TCP / UDP
        ↓
4. DNS
        ↓
5. HTTP
        ↓
6. HTTPS / TLS
        ↓
7. REST API
        ↓
8. Spring MVC
        ↓
9. Spring Security
        ↓
10. Reverse Proxy / Nginx
        ↓
11. Load Balancer
        ↓
12. Redis
        ↓
13. Kafka
        ↓
14. Distributed Systems
        ↓
15. System Design
```

---

# 109. Final Mental Picture

If you remember only one picture from this note, remember this:

```text
                 USER
                  |
                  ↓
               BROWSER
                  |
             HTTPS Request
                  |
                  ↓
                 DNS
                  |
             Find IP Address
                  |
                  ↓
                 TCP
                  |
           Reliable Connection
                  |
                  ↓
                 TLS
                  |
          Secure Communication
                  |
                  ↓
                 HTTP
                  |
            Request / Response
                  |
                  ↓
            LOAD BALANCER
                  |
                  ↓
            SPRING BOOT
                  |
             +----+----+
             |         |
          Security   Controller
                       |
                     Service
                       |
                   Repository
                       |
                       ↓
                  PostgreSQL
                       |
                       ↓
                    Redis
                       |
                       ↓
                    Kafka
                       |
                       ↓
                HTTP Response
                       |
                       ↓
                    Browser
```

The important lesson is:

> **HTTP is only one part of web communication.**

A real request travels through multiple layers:

```text
DNS
 ↓
Network
 ↓
TCP/QUIC
 ↓
TLS
 ↓
HTTP
 ↓
Proxy/Load Balancer
 ↓
Application
 ↓
Database/Cache/Message Broker
```

Once you understand these layers separately, debugging, Spring Boot development, REST API design, cloud deployment and system design become much easier.

---

# 110. Quick Revision — 15 Questions

Before considering this topic complete, answer these without looking at the note:

1. What is a client?
2. What is a server?
3. What is a protocol?
4. What problem does DNS solve?
5. What problem does TCP solve?
6. What problem does TLS solve?
7. What is HTTP?
8. What is HTTPS?
9. What is the difference between Content-Type and Accept?
10. What is the difference between 401 and 403?
11. What is the difference between 404 and 500?
12. Why are cookies used?
13. What happens when you open an HTTPS website?
14. How does an HTTP request reach a Spring Boot controller?
15. Why are timeout, retry and idempotency important in production systems?

---

# 111. One-Minute Revision

```text
CLIENT
→ Sends request

SERVER
→ Processes request and sends response

DNS
→ Domain → IP

TCP
→ Reliable transport

TLS
→ Secure communication

HTTP
→ Request/response protocol

HTTPS
→ HTTP + TLS

COOKIE
→ Client-side state mechanism

SESSION
→ Server-side state associated with a client/session identifier

JWT
→ Token-based authentication mechanism

REST
→ Architectural style commonly implemented using HTTP

SPRING BOOT
→ Backend application that receives and processes HTTP requests

2xx
→ Success

4xx
→ Client/request-side problem

5xx
→ Server-side problem

LOAD BALANCER
→ Distributes traffic

REDIS
→ Fast in-memory data store/cache

KAFKA
→ Event streaming/message platform

SYSTEM DESIGN
→ Combining all these pieces into a reliable, scalable system
```

---

# 112. Personal Learning Rule

Do not try to memorise this entire note.

Instead, draw this on paper:

```text
Browser
   ↓
DNS
   ↓
TCP
   ↓
TLS
   ↓
HTTP
   ↓
Load Balancer
   ↓
Spring Boot
   ↓
Service
   ↓
Database
   ↓
Response
```

Then explain every arrow in your own words.

If you can explain:

> **"What happens from the moment I type `https://example.com` until the response reaches my browser?"**

without reading your notes, you have understood the foundation.

The next level is to connect this mental model with **TCP/IP, DNS internals, TLS handshake, Spring Security, reverse proxies, load balancing, Docker networking and distributed-system communication.**

---

# Related Obsidian Notes

- [[Computer Network Fundamentals]]
- [[OSI Model]]
- [[TCP vs UDP]]
- [[IP Addressing]]
- [[DNS]]
- [[TCP Three-Way Handshake]]
- [[TLS and SSL]]
- [[REST API]]
- [[HTTP Status Codes]]
- [[HTTP Headers]]
- [[Cookies and Sessions]]
- [[JWT Authentication]]
- [[CORS]]
- [[Spring MVC]]
- [[Spring Security]]
- [[Nginx and Reverse Proxy]]
- [[Load Balancer]]
- [[API Gateway]]
- [[Redis]]
- [[Kafka]]
- [[Distributed Systems]]
- [[System Design]]
- [[Production Debugging]]