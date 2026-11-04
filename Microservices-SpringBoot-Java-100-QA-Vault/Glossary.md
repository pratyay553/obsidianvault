---
tags: [microservices, glossary]
---

# Glossary

Short, simple meanings. Back to [[00 - Index]].

| Term | Meaning |
|---|---|
| **API** | Rules that let one program call another, usually through URLs. |
| **API Gateway** | Single entry point that routes client requests to services and handles auth and rate limits. |
| **Asynchronous** | Start a task and continue without waiting for it to finish. |
| **Auto-configuration** | Spring Boot creates beans automatically based on your libraries and settings. |
| **Backoff** | Waiting longer after each failed retry (1s, 2s, 4s). |
| **Bean** | An object created and managed by Spring. |
| **Blocking** | A thread waits until the result arrives. |
| **Bounded Context** | A boundary where a business word has one clear meaning; a good service boundary. |
| **Bulkhead** | Limits resources per dependency so one slow service cannot take everything. |
| **Cache** | Fast temporary storage for data that is read often. |
| **CAP Theorem** | In a network failure you must choose consistency or availability. |
| **CI/CD** | Automatic build, test and release of code. |
| **Circuit Breaker** | Stops calls to a failing service so it can recover and you fail fast. |
| **Container** | A running package of an app and everything it needs (Docker). |
| **Contract Testing** | Tests that check provider and consumer agree on an API. |
| **CQRS** | Separate models for writing and reading data. |
| **DDD** | Domain-Driven Design: design software around the business domain. |
| **Dead Letter Queue** | Place where messages that keep failing are sent for investigation. |
| **Dependency Injection** | Spring gives a class the objects it needs instead of the class creating them. |
| **DTO** | Simple object used to carry data in and out of an API. |
| **Event Sourcing** | Store every change as an event instead of only the current state. |
| **Eventual Consistency** | Data copies become the same after a short delay. |
| **Fallback** | Plan B result when a call fails. |
| **Feign** | Library to call REST APIs by writing an interface. |
| **Idempotent** | Doing it many times gives the same result as doing it once. |
| **JWT** | Signed token with user claims used for authentication. |
| **Kafka** | Distributed event log used for messaging and streaming. |
| **Kubernetes** | System that runs and manages containers at scale. |
| **Liveness Probe** | Kubernetes check: is this Pod alive, or should it be restarted? |
| **Load Balancer** | Spreads traffic over many copies of a service. |
| **Message Broker** | Software that holds messages between services (Kafka, RabbitMQ). |
| **Micrometer** | Metrics library used by Spring Boot. |
| **mTLS** | Both client and server prove identity and traffic is encrypted. |
| **N+1 Problem** | One query for a list, then one extra query per item; slow. |
| **OAuth2** | Standard for giving limited access without sharing a password. |
| **Observability** | Understanding a system from its logs, metrics and traces. |
| **OIDC** | Layer on OAuth2 that tells who the user is. |
| **Outbox Pattern** | Save data and the event in one transaction, publish the event later. |
| **Pagination** | Returning data in small pages. |
| **Profile** | Named set of settings for one environment (dev, prod). |
| **Rate Limiting** | Limiting how many requests a client can make in a time window. |
| **Readiness Probe** | Kubernetes check: can this Pod receive traffic? |
| **Resilience4j** | Java library for circuit breaker, retry, bulkhead, rate limiter. |
| **Saga** | A multi-service business transaction made of local steps with compensations. |
| **Service Discovery** | Finding services by name, not fixed address. |
| **Service Mesh** | Sidecar proxies that handle security and traffic between services. |
| **Stateless** | Does not remember anything between requests. |
| **Testcontainers** | Starts real Docker containers (DB, Kafka) for tests. |
| **Trace / Span** | Journey of one request across services / one step of it. |
| **Virtual Thread** | Very light Java 21 thread, great for waiting on I/O. |
