---
tags: [api-design, grpc, websockets, graphql, mqtt, amqp, performance, architecture]
aliases: [REST Alternatives, Beyond REST, API Communication Protocols]
---

# Why REST Lags — and What Replaces It

> [!abstract] Core Idea
> REST's overhead comes from its architecture itself, not just bad implementation:
> - **Statelessness** → every request re-sends context (auth, headers)
> - **JSON parsing** → text-based, verbose, CPU-costly to serialize/deserialize
> - **Synchronous request-response** → no native push/streaming
> - **HTTP handshake overhead** → new connection cost per interaction (worse on HTTP/1.1)
>
> Related: [[HttpServletRequest vs HttpServletResponse]] · [[REST API Design]] · [[HTTP-1.1 vs HTTP-2 vs HTTP-3]]

When you need **higher throughput**, **lower latency**, or **real-time streaming**, reach for one of the protocols below instead of forcing REST to do a job it wasn't built for.

---

## 1. gRPC — Best for Microservices & High-Throughput RPC

> [!info] Transport & Format
> HTTP/2 transport + **Protocol Buffers (Protobuf)** binary serialization, instead of text-based JSON over HTTP/1.1.

### Why it's faster
- **Binary serialization** — Protobuf payloads are far smaller and cheaper to parse than JSON → [[Protocol Buffers - Protobuf]]
- **Multiplexing** — HTTP/2 runs many requests/responses concurrently on **one** TCP connection, eliminating head-of-line blocking → [[HTTP-2 Multiplexing]]
- **Streaming built-in** — client-side, server-side, and bidirectional streaming natively supported

### Best Use Case
Internal **service-to-service** communication where speed and strict typing matter.

> [!tip] Trade-off
> gRPC is not human-readable in transit and browsers can't call it natively (needs [[gRPC-Web]] as a proxy layer) — poor fit for public-facing APIs.

Related: [[Microservices Architecture]] · [[Protocol Buffers - Protobuf]] · [[Service Mesh]]

---

## 2. WebSockets — Best for Real-Time, Bidirectional Communication

> [!info] The Problem It Solves
> REST is strictly request-response. If a client wants fresh data, it must **poll** repeatedly — wasted bandwidth and CPU on both ends.

### Why it's faster
- **Persistent connection** — one TCP connection opened via an HTTP handshake, then kept alive; no reconnect cost per message
- **Full-duplex** — client and server can each push data independently, at any time
- **Low overhead** — frame headers as small as **2–10 bytes**, versus full HTTP headers per REST call

### Best Use Case
Chat apps, live dashboards, collaborative editing tools, live ticker/price feeds.

> [!tip] Trade-off
> No built-in request/response semantics or retry — you build your own message protocol on top. Compare to [[Server-Sent Events - SSE]] for simpler one-way streaming needs.

Related: [[Polling vs WebSockets vs SSE]] · [[Socket.IO]] · [[Real-Time Architecture Patterns]]

---

## 3. GraphQL — Best for Over-Fetching / Under-Fetching Problems

> [!info] The Problem It Solves
> REST endpoints often return **too much data** (over-fetching) or force **multiple round-trips** to assemble a full view (under-fetching).

### Why it's faster (in practice)
- **Precise querying** — clients request exactly the fields they need, in a single round-trip
- **Single endpoint** — no more stitching together several REST calls to build one screen

### Best Use Case
Complex client-facing apps (especially mobile) with varied data needs across screens.

> [!tip] Trade-off
> Still typically rides over HTTP, so it doesn't fix transport-level overhead — and naive resolvers can introduce the **N+1 query problem** → [[GraphQL N+1 Problem]]. Caching is also harder than REST's URL-based caching.

Related: [[GraphQL vs REST]] · [[DataLoader Pattern]] · [[API Gateway]]

---

## 4. MQTT / AMQP — Best for IoT & Event-Driven Systems

> [!info] The Problem It Solves
> IoT devices and resource-constrained environments can't afford standard HTTP/REST overhead.

### MQTT
- Lightweight **publish-subscribe** protocol for low-bandwidth, high-latency, or unreliable networks
- Fixed header as small as **2 bytes**
- Related: [[Publish-Subscribe Pattern]] · [[MQTT QoS Levels]]

### AMQP
- Robust **message queuing** protocol for reliable, complex routing between enterprise systems
- Related: [[Message Queues - RabbitMQ vs Kafka]] · [[Event-Driven Architecture]]

### Best Use Case
Smart devices, sensor data ingestion, heavy async background message processing.

---

## Summary Comparison

| Protocol | Transport | Data Format | Communication Style | Best Suited For |
|---|---|---|---|---|
| **REST** | HTTP/1.1 / HTTP/2 | JSON / XML | Request–Response | Public APIs, CRUD operations |
| **gRPC** | HTTP/2 | Binary (Protobuf) | Request–Response / Streaming | Internal microservices |
| **WebSockets** | TCP | Text / Binary | Full-Duplex / Streaming | Real-time / live feeds |
| **GraphQL** | HTTP/1.1 / HTTP/2 | JSON | Query–Response (client-driven) | Complex frontend/mobile clients |
| **MQTT** | TCP | Binary | Publish–Subscribe | IoT & edge devices |
| **AMQP** | TCP | Binary | Message Queuing | Enterprise messaging / async jobs |

---

## Decision Guide

```text
Need strict typing + fast internal service calls?      → gRPC
Need live, two-way updates in a browser?                → WebSockets
Need a client to shape its own response, one round-trip? → GraphQL
Need lightweight pub/sub for constrained devices?        → MQTT
Need reliable enterprise message routing/queuing?        → AMQP
Need simple, cacheable, public-facing CRUD?               → REST (still fine!)
```

> [!warning] Don't Over-Engineer
> REST remains the right default for public APIs, CRUD-heavy services, and anything prioritizing simplicity, caching, and broad client compatibility. These alternatives solve *specific* bottlenecks — adopt them when you actually hit that bottleneck, not preemptively.

---

## Related Notes
- [[HttpServletRequest vs HttpServletResponse]]
- [[REST API Design]]
- [[HTTP-1.1 vs HTTP-2 vs HTTP-3]]
- [[Microservices Architecture]]
- [[Protocol Buffers - Protobuf]]
- [[GraphQL vs REST]]
- [[Message Queues - RabbitMQ vs Kafka]]
- [[Event-Driven Architecture]]
- [[API Gateway]]
- [[Server-Sent Events - SSE]]

## Open Questions / To Explore
- [ ] When does gRPC-Web make sense vs. a REST facade in front of gRPC?
- [ ] How do you handle auth/session state over a persistent WebSocket connection?
- [ ] GraphQL subscriptions vs WebSockets — when to use which?
- [ ] MQTT vs AMQP: how to choose between them for a mixed IoT + enterprise system?
