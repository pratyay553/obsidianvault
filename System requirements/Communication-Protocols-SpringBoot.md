---
tags: [system-design, spring-boot, interview-prep, backend, protocols]
title: Communication Protocols in Spring Boot
created: 2026-08-24
---

# 🌐 Communication Protocols — Spring Boot & Interview Guide

> [!info] How to use this note
> Each protocol has: **Definition → Real-world Spring Boot use case → Advantages → Disadvantages → 30-second Interview Answer**.
> Use the "30-second Interview Answer" section as your verbal script when asked "Explain X protocol" in an interview.

---

## 🗺️ Quick Comparison Table

| Protocol | Layer | Connection Type | Real-time? | Spring Boot Starter | Best For |
|---|---|---|---|---|---|
| HTTP/HTTPS | Application (L7) | Request-Response | No | `spring-boot-starter-web` | REST APIs |
| WebSocket | Application (L7, over TCP) | Full-duplex, persistent | Yes | `spring-boot-starter-websocket` | Chat, live dashboards |
| REST (over HTTP) | Application | Request-Response | No | `spring-boot-starter-web` | CRUD APIs |
| GraphQL | Application (over HTTP) | Request-Response | No | `spring-boot-starter-graphql` | Flexible client queries |
| gRPC | Application (over HTTP/2) | Request-Response / Streaming | Yes (streaming) | `grpc-spring-boot-starter` | Microservice-to-microservice |
| SOAP | Application (over HTTP/SMTP) | Request-Response | No | `spring-boot-starter-web-services` | Legacy enterprise/banking systems |
| SSE (Server-Sent Events) | Application (over HTTP) | Server → Client stream | Yes (one-way) | `spring-boot-starter-web` | Live feeds, notifications |
| AMQP (RabbitMQ) | Application | Message Queue (async) | Near real-time | `spring-boot-starter-amqp` | Order processing, decoupled services |
| Kafka Protocol | Application | Pub/Sub Log-based | Near real-time | `spring-kafka` | Event streaming, analytics pipelines |
| MQTT | Application (lightweight) | Pub/Sub | Yes | `spring-integration-mqtt` | IoT devices |
| STOMP | Application (over WebSocket) | Pub/Sub messaging | Yes | `spring-boot-starter-websocket` | Chat apps with topic subscriptions |
| TCP | Transport (L4) | Connection-oriented | N/A | Java Sockets / Netty | Low-level custom protocols |
| UDP | Transport (L4) | Connectionless | Yes | Java `DatagramSocket` | Video streaming, gaming |
| FTP/SFTP | Application | Connection-oriented | No | Spring Integration FTP | File transfer batch jobs |
| JMS | Application (Java abstraction) | Message Queue | Near real-time | `spring-boot-starter-artemis`/ActiveMQ | Enterprise messaging |

---

## 1️⃣ HTTP / HTTPS

### Definition
Stateless, request-response protocol where a client sends a request and the server returns a response, then the connection (logically) ends.

### Real-world Spring Boot Use Case
Every `@RestController` endpoint — e.g., an e-commerce app's `/api/products` endpoint that a React frontend calls to list products. HTTPS is used in production for the TLS/SSL encryption layer (via `server.ssl.*` properties or a reverse proxy like Nginx).

### ✅ Advantages
- Simple, stateless, widely supported by every client (browsers, mobile, Postman).
- Easy to cache (GET requests), scale horizontally (no session pinning needed if stateless).
- Human-readable, easy to debug.

### ❌ Disadvantages
- Overhead per request (headers, handshake) — not efficient for real-time/high-frequency updates.
- Stateless nature means auth/session must be handled separately (JWT, cookies).
- HTTP/1.1 has head-of-line blocking (solved partially in HTTP/2).

> [!quote] Interview Answer
> "HTTP is a stateless, request-response protocol — the client always initiates. In Spring Boot, it powers our REST controllers. I'd highlight that we use HTTPS in production for encryption, and note HTTP/1.1's head-of-line blocking problem, which HTTP/2 (multiplexing) and HTTP/3 (QUIC/UDP-based) solve."

---

## 2️⃣ WebSocket

### Definition
Provides a **full-duplex, persistent** connection over a single TCP connection after an initial HTTP "upgrade" handshake. Both client and server can push data anytime.

### Real-world Spring Boot Use Case
A live stock-trading dashboard or chat application. Spring Boot uses `@EnableWebSocket` / `@ServerEndpoint`, often paired with **STOMP** for topic-based pub-sub (`/topic/prices`), so multiple clients get pushed updates without polling.

### ✅ Advantages
- True bidirectional, low-latency communication.
- Single connection reused — much less overhead than repeated HTTP polling.
- Ideal for real-time features (chat, live scores, collaborative editing).

### ❌ Disadvantages
- More complex to scale horizontally (need sticky sessions or a message broker like Redis/RabbitMQ to sync state across instances).
- Firewalls/proxies sometimes block or mishandle WebSocket upgrades.
- No built-in request-response semantics — you must build your own message protocol (which is why STOMP is often layered on top).

> [!quote] Interview Answer
> "WebSocket upgrades an HTTP connection into a persistent, full-duplex channel — great for chat or live dashboards. In Spring Boot, I'd combine it with STOMP for pub-sub messaging semantics, and I'd mention the scaling challenge: since it's stateful, you need sticky sessions or an external broker to fan out messages across multiple server instances."

---

## 3️⃣ REST (Representational State Transfer)

### Definition
An architectural style (not a protocol itself) built on top of HTTP, using standard verbs (GET/POST/PUT/DELETE) and resource-based URLs.

### Real-world Spring Boot Use Case
Standard CRUD microservice — `@RestController` with `@GetMapping`, `@PostMapping` etc., returning JSON via Jackson. E.g., a `UserController` exposing `/api/users`.

### ✅ Advantages
- Stateless and cacheable, scales well.
- Uses standard HTTP status codes — easy to reason about.
- Huge ecosystem/tooling support (Swagger/OpenAPI, Postman).

### ❌ Disadvantages
- Over-fetching/under-fetching data (client gets fixed response shape) — solved by GraphQL.
- Multiple round trips sometimes needed for related resources (N+1 problem at API level).
- Versioning APIs can get messy over time.

> [!quote] Interview Answer
> "REST is an architectural style on top of HTTP using resource-oriented URLs and standard verbs. I'd contrast it with GraphQL: REST is simple and cacheable but can over-fetch or under-fetch data, whereas GraphQL lets the client specify exactly what it needs."

---

## 4️⃣ GraphQL

### Definition
A query language for APIs where the client specifies the exact shape of data it wants in a single request, typically over a single HTTP POST endpoint.

### Real-world Spring Boot Use Case
A mobile app that needs a nested response (user + their orders + order items) in **one call** instead of three REST calls. Implemented via `spring-boot-starter-graphql` with `.graphqls` schema files and `@QueryMapping`/`@SchemaMapping` resolvers.

### ✅ Advantages
- Client controls exactly what fields it needs — no over/under-fetching.
- Single endpoint, strongly typed schema, self-documenting.
- Great for aggregating data from multiple services (BFF pattern).

### ❌ Disadvantages
- Harder to cache than REST (single endpoint, varying queries).
- N+1 query problem on the backend if resolvers aren't optimized (solved with DataLoader pattern).
- Steeper learning curve; overkill for simple CRUD services.

> [!quote] Interview Answer
> "GraphQL lets clients query exactly the fields they need in one round trip, which is great for mobile clients with limited bandwidth or nested data needs. The trade-off is caching becomes harder since it's typically a single POST endpoint, and you have to watch out for the N+1 problem in resolvers."

---

## 5️⃣ gRPC

### Definition
A high-performance RPC framework from Google using **HTTP/2** and **Protocol Buffers (protobuf)** for binary serialization. Supports unary and streaming calls (client, server, or bidirectional streaming).

### Real-world Spring Boot Use Case
Internal microservice-to-microservice communication in a high-throughput system — e.g., an `OrderService` calling `InventoryService` synchronously with strict low-latency requirements. Used heavily in polyglot microservice architectures (Java service talking to a Go/Python service).

### ✅ Advantages
- Extremely fast — binary protobuf serialization + HTTP/2 multiplexing.
- Strongly typed contracts (`.proto` files) generate client/server stubs automatically — great for polyglot teams.
- Native support for streaming (e.g., real-time telemetry from IoT).

### ❌ Disadvantages
- Not natively browser-friendly (needs gRPC-Web + proxy for frontend use).
- Binary format isn't human-readable — harder to debug without tooling (e.g., `grpcurl`, BloomRPC).
- Steeper setup (proto compilation, code generation) compared to REST.

> [!quote] Interview Answer
> "gRPC uses HTTP/2 and Protocol Buffers for fast, strongly-typed internal service-to-service communication. I'd use it between microservices, not for browser-facing APIs, since browsers don't support gRPC natively. The main win is performance and contract-first strongly typed stubs; the trade-off is it's not human-readable and needs extra tooling to debug."

---

## 6️⃣ SOAP (Simple Object Access Protocol)

### Definition
An XML-based messaging protocol with a strict contract (WSDL), typically over HTTP, with built-in standards for security (WS-Security) and transactions.

### Real-world Spring Boot Use Case
Integrating with legacy banking, insurance, or government systems that still expose SOAP web services. Spring Boot uses `spring-boot-starter-web-services` with JAXB for XML marshalling.

### ✅ Advantages
- Strict contract (WSDL) — strong typing and validation built in.
- Built-in standards for security, transactions, and reliability (important for banking/finance).
- Protocol-agnostic (can run over HTTP, SMTP, etc.).

### ❌ Disadvantages
- Verbose XML payloads — slower, larger than JSON/REST.
- Complex to implement and consume compared to REST.
- Considered legacy; rarely used for new projects today.

> [!quote] Interview Answer
> "SOAP is an XML-based protocol with a strict WSDL contract, mostly seen today in legacy enterprise integrations like banking or insurance systems. It's verbose compared to REST/JSON but offers strong typing and built-in security/transaction standards, which is why regulated industries still rely on it."

---

## 7️⃣ Server-Sent Events (SSE)

### Definition
A one-way, server-to-client streaming protocol over plain HTTP — the server keeps the connection open and pushes text-based events.

### Real-world Spring Boot Use Case
A notification bell that updates live, or a progress bar for a long-running job (e.g., file upload processing status). Implemented via Spring's `SseEmitter` or reactive `Flux<ServerSentEvent<T>>` in WebFlux.

### ✅ Advantages
- Simpler than WebSocket when you only need server → client updates.
- Works over plain HTTP — no special upgrade handshake, auto-reconnect built into the browser `EventSource` API.
- Lightweight for one-directional real-time features.

### ❌ Disadvantages
- One-directional only (client can't send data back on the same channel).
- Limited browser connection pool (6 per domain in HTTP/1.1) can be a bottleneck.
- Not ideal for binary data (text/event-stream based).

> [!quote] Interview Answer
> "SSE is a lightweight, one-way server-to-client streaming protocol over plain HTTP, great for notifications or progress updates where I don't need the client to talk back. It's simpler than WebSocket to set up, but if I need true bidirectional communication, I'd reach for WebSocket instead."

---

## 8️⃣ AMQP (Advanced Message Queuing Protocol) — RabbitMQ

### Definition
A message-queue protocol enabling asynchronous, decoupled communication between services via **producers, exchanges, queues, and consumers**.

### Real-world Spring Boot Use Case
An e-commerce `OrderService` publishes an "OrderPlaced" event to RabbitMQ; `InventoryService`, `EmailService`, and `PaymentService` independently consume it. Implemented via `spring-boot-starter-amqp` with `RabbitTemplate` and `@RabbitListener`.

### ✅ Advantages
- Decouples services — publisher doesn't need to know who consumes the message.
- Reliable delivery with acknowledgments, retries, dead-letter queues.
- Smooths traffic spikes (buffer between fast producer and slower consumer).

### ❌ Disadvantages
- Added infrastructure complexity (broker to maintain, monitor).
- Eventual consistency — not suitable when you need an immediate synchronous response.
- Message ordering guarantees can be tricky at scale.

> [!quote] Interview Answer
> "AMQP, via RabbitMQ, is how I'd decouple services asynchronously — like an OrderService publishing an event that Inventory and Email services consume independently. It's great for reliability and buffering spikes, but it introduces eventual consistency, so it's not right when the caller needs an immediate response."

---

## 9️⃣ Kafka Protocol

### Definition
A distributed, append-only log-based pub-sub protocol designed for high-throughput event streaming, with consumers tracking their own read offset.

### Real-world Spring Boot Use Case
A ride-sharing app streaming millions of GPS location events per second for analytics, or an audit-log/event-sourcing system. Implemented via `spring-kafka`'s `KafkaTemplate` and `@KafkaListener`.

### ✅ Advantages
- Extremely high throughput, horizontally scalable (partitions).
- Messages are persisted (replayable) — great for event sourcing/analytics, unlike RabbitMQ which typically deletes after consumption.
- Strong ordering guarantee within a partition.

### ❌ Disadvantages
- Heavier operational overhead (Zookeeper/KRaft, partition management, tuning).
- Not designed for simple task-queue style use cases (RabbitMQ is simpler there).
- Consumer offset management adds complexity if not using Spring Kafka's abstractions correctly.

> [!quote] Interview Answer
> "Kafka is a distributed log built for high-throughput event streaming — think millions of events per second, like clickstreams or GPS pings. Unlike RabbitMQ, messages persist and can be replayed, which makes it great for event sourcing. The trade-off is more operational complexity."

---

## 🔟 MQTT (Message Queuing Telemetry Transport)

### Definition
A lightweight publish-subscribe protocol designed for constrained devices and low-bandwidth, high-latency networks.

### Real-world Spring Boot Use Case
An IoT platform where thousands of temperature sensors publish readings to a broker (e.g., Mosquitto/HiveMQ), and a Spring Boot service subscribes via `spring-integration-mqtt` to process and store the data.

### ✅ Advantages
- Extremely lightweight — minimal header overhead, ideal for battery/bandwidth-constrained IoT devices.
- QoS levels (0, 1, 2) let you tune reliability vs. overhead.
- "Last Will and Testament" feature notifies subscribers if a device disconnects unexpectedly.

### ❌ Disadvantages
- Not designed for large payloads or complex request-response interactions.
- Requires a separate broker (Mosquitto, HiveMQ) — added infra.
- Security needs to be layered on top (TLS, auth) — not as built-in as SOAP's WS-Security.

> [!quote] Interview Answer
> "MQTT is a lightweight pub-sub protocol built for IoT — thousands of low-power sensors publishing small payloads to a broker. Its QoS levels let you choose delivery guarantees, but it's not meant for large payloads or complex request-response — it's purely optimized for constrained devices."

---

## 1️⃣1️⃣ STOMP (Simple Text Oriented Messaging Protocol)

### Definition
A simple, text-based messaging protocol that runs **on top of WebSocket** to add pub-sub / broker-style semantics (subscribe/send to destinations).

### Real-world Spring Boot Use Case
A group chat app: Spring Boot config (`@EnableWebSocketMessageBroker`) sets up a `/topic` broker; clients subscribe to `/topic/room1` and send messages to `/app/chat`. This is the standard "Spring Boot chat app" pattern.

### ✅ Advantages
- Adds structured pub-sub messaging (topics/queues) on top of raw WebSocket, which has no built-in routing.
- Simple text-based frame format, easy to debug.
- Well integrated into Spring's `spring-messaging` module (works like a mini message broker).

### ❌ Disadvantages
- Still inherits WebSocket's scaling challenges (needs external broker like RabbitMQ for multi-instance deployments via `enableStompBrokerRelay`).
- Adds a layer of abstraction/complexity vs. raw WebSocket for very simple use cases.

> [!quote] Interview Answer
> "STOMP sits on top of WebSocket to give you pub-sub semantics — topics and destinations — instead of raw message frames. It's the standard approach for building chat apps in Spring Boot with `@EnableWebSocketMessageBroker`. For production scale across multiple instances, I'd relay it through an external broker like RabbitMQ."

---

## 1️⃣2️⃣ TCP (Transmission Control Protocol)

### Definition
A transport-layer, connection-oriented protocol guaranteeing reliable, ordered delivery of data via handshakes and acknowledgments.

### Real-world Spring Boot Use Case
Rarely used directly in Spring Boot business logic (HTTP/gRPC/AMQP all run on top of TCP), but you'd use raw TCP (via Netty/Java Sockets) for a custom protocol — e.g., a custom trading engine protocol or integrating with legacy hardware/POS systems that speak raw TCP.

### ✅ Advantages
- Reliable, ordered, guarantees delivery (retransmission on packet loss).
- Foundation for almost every other protocol above (HTTP, gRPC, AMQP, MQTT all use TCP).

### ❌ Disadvantages
- Slower than UDP due to handshake and acknowledgment overhead.
- Building a custom protocol on raw TCP means you must handle framing, security, and error-handling yourself.

> [!quote] Interview Answer
> "TCP is the reliable, ordered transport-layer protocol underneath HTTP, gRPC, and AMQP. I'd rarely touch it directly in Spring Boot unless integrating with legacy hardware or building a custom low-level protocol with Netty — for standard business services, I'd use a higher-level protocol built on top of it."

---

## 1️⃣3️⃣ UDP (User Datagram Protocol)

### Definition
A connectionless transport-layer protocol — sends data without guaranteeing delivery, order, or duplicate protection, prioritizing speed.

### Real-world Spring Boot Use Case
Rare in typical Spring Boot web apps, but used for things like sending metrics to StatsD/Graphite, or a custom live video/gaming feed component built alongside a Spring Boot backend.

### ✅ Advantages
- Very low latency — no handshake, no acknowledgment overhead.
- Good for scenarios where occasional data loss is acceptable (live video, metrics).

### ❌ Disadvantages
- No guarantee of delivery, ordering, or duplication protection.
- Rarely appropriate for core business transactions (payments, orders) where data integrity matters.

> [!quote] Interview Answer
> "UDP trades reliability for speed — no handshake, no delivery guarantee. I wouldn't use it for business-critical Spring Boot APIs, but it fits use cases like metrics collection or live video/gaming where a few dropped packets don't matter, but latency does."

---

## 1️⃣4️⃣ FTP / SFTP

### Definition
Protocols for transferring files between systems; SFTP adds SSH-based encryption on top of FTP's plain-text transfer.

### Real-world Spring Boot Use Case
A nightly batch job where a Spring Boot service (using `spring-integration-sftp` or Spring Batch) pulls a vendor's CSV file from an SFTP server, processes it, and loads records into a database.

### ✅ Advantages
- Simple, well-established standard for bulk file transfer.
- SFTP adds encryption + authentication (via SSH), suitable for sensitive data (e.g., financial batch files).

### ❌ Disadvantages
- Not real-time — batch-oriented by nature.
- Plain FTP is insecure (no encryption) — should never be used for sensitive data; always prefer SFTP/FTPS.

> [!quote] Interview Answer
> "FTP/SFTP is what I'd reach for in batch integration scenarios — e.g., pulling a partner's nightly CSV file via SFTP into a Spring Batch job. SFTP is the secure choice since it's SSH-encrypted; plain FTP should be avoided for anything sensitive."

---

## 1️⃣5️⃣ JMS (Java Message Service)

### Definition
A Java-standard API abstraction for messaging (point-to-point queues and pub-sub topics), implemented by providers like ActiveMQ Artemis, IBM MQ.

### Real-world Spring Boot Use Case
An enterprise Java shop using ActiveMQ Artemis for internal messaging between Spring Boot services — e.g., `spring-boot-starter-artemis` with `@JmsListener` to process queued tasks like report generation requests.

### ✅ Advantages
- Standard Java API — portable across JMS providers (not locked into one broker's client API).
- Supports both point-to-point (queue) and pub-sub (topic) models.
- Mature, well-integrated into the Spring ecosystem (`JmsTemplate`).

### ❌ Disadvantages
- Java-ecosystem specific — not ideal for polyglot microservices (unlike Kafka/AMQP which have broad language support).
- Generally less throughput-optimized than Kafka for large-scale event streaming.

> [!quote] Interview Answer
> "JMS is Java's standard messaging API abstraction — think of it as the JDBC equivalent for messaging. In Spring Boot, I'd use it with a provider like ActiveMQ Artemis via `JmsTemplate`. It's great in all-Java shops, but for polyglot or high-throughput event streaming, I'd lean toward Kafka or AMQP instead."

---

## 🎯 Choosing the Right Protocol — Interview Decision Framework

> [!tip] The "which protocol would you pick" answer pattern
> 1. **Is it synchronous request-response?** → REST (simple) or GraphQL (flexible queries) or gRPC (internal, high-perf).
> 2. **Does the client need real-time push?** → WebSocket (bidirectional) or SSE (one-way).
> 3. **Do services need to be decoupled/async?** → AMQP (task queue, reliable) or Kafka (event streaming, replayable, high volume).
> 4. **Is it IoT / constrained devices?** → MQTT.
> 5. **Legacy enterprise integration?** → SOAP.
> 6. **Bulk file transfer?** → SFTP.

---

## 🧠 Common Interview Follow-Up Questions

- **"REST vs gRPC — when would you choose which?"**
  REST for public-facing, browser-friendly, cacheable APIs; gRPC for internal, low-latency, high-throughput service-to-service calls with strict contracts.

- **"Kafka vs RabbitMQ?"**
  RabbitMQ = smart broker, simple routing, task queues, messages typically deleted after consumption. Kafka = dumb broker/smart consumer, persistent replayable log, built for high-throughput event streaming.

- **"WebSocket vs SSE?"**
  WebSocket = bidirectional; SSE = server-to-client only but simpler and works over plain HTTP.

- **"How do you scale WebSocket/STOMP across multiple Spring Boot instances?"**
  Use a message broker relay (RabbitMQ/Redis pub-sub) so a message published on one instance reaches clients connected to another instance.

- **"How does Spring Boot achieve async, non-blocking I/O?"**
  Via **Spring WebFlux** (reactive stack, `Mono`/`Flux`, Netty server) instead of the traditional Servlet/Tomcat blocking stack — relevant when discussing HTTP performance under high concurrency.

---

## 📌 Summary Cheat Sheet (for quick revision)

- **HTTP/REST** → default choice for CRUD APIs.
- **GraphQL** → client-driven flexible queries, aggregation layer.
- **gRPC** → fast internal microservice calls, streaming.
- **WebSocket/STOMP** → real-time bidirectional (chat, dashboards).
- **SSE** → simple one-way real-time push.
- **AMQP/RabbitMQ** → decoupled async task processing.
- **Kafka** → high-throughput event streaming, replayable log.
- **MQTT** → IoT, constrained devices.
- **SOAP** → legacy enterprise/regulated industries.
- **SFTP** → secure batch file transfer.
- **JMS** → Java-standard messaging abstraction.
- **TCP/UDP** → transport-layer foundation; rarely touched directly except for custom protocols.
