---
tags: [microservices, springboot, java, interview, scenarios]
category: Real-World Scenarios
questions: 94-100
status: to-revise
---

# Real-World Scenarios (Q94 to Q100)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q94. Scenario: Service C is slow and the whole system hangs. What do you do? 🔴
> This is a **cascading failure**. Steps:
> 1. **Fix now:** set timeouts on all calls, add a **circuit breaker** with a fallback, limit threads with a **bulkhead**.
> 2. **Find the cause:** use traces and metrics to see why C is slow (DB query, GC, too much traffic).
> 3. **Reduce coupling:** replace the synchronous call with **async messaging** if the answer is not needed immediately; add **caching**.
> 4. **Protect:** rate limiting and autoscaling for C.
> 5. **Prevent:** load and chaos tests, alerts on latency.

> [!question]- Q95. Scenario: An order is created but the payment fails. How do you keep data consistent? 🔴
> Use the **Saga pattern** (no distributed transaction):
> 1. Order service creates the order with status `PENDING`.
> 2. Stock service reserves items.
> 3. Payment service charges the card.
> 4. If payment fails, it publishes `PaymentFailed`; stock is **released** and the order becomes `CANCELLED`.
>
> Use the **Outbox** pattern to publish events safely, **idempotent** consumers to handle duplicates, and show "pending" state to the user.

> [!question]- Q96. Scenario: How do you move a monolith to microservices? 🔴
> Use the **Strangler Fig pattern**: grow new services around the old app and replace it slowly.
> 1. Find boundaries (bounded contexts). Start with a part that changes often or is easy to separate.
> 2. Put a **gateway/proxy** in front of the monolith.
> 3. Build the new service, then route that part of traffic to it.
> 4. Move the data carefully (sync during transition, then cut over).
> 5. Remove the old code. Repeat.
>
> Never do a "big bang" rewrite; it is risky.
>
> 📖 **Simple words:**
> - **Strangler Fig** = named after a vine that slowly grows around a tree and replaces it.

> [!question]- Q97. Scenario: Users report errors in production. How do you find the failing service? 🟡
> 1. Check **dashboards** (RED metrics): which service has high errors or latency?
> 2. Take a failing request's **trace id** and open the **trace**: see which span failed.
> 3. Search **logs** by that trace id for the exact exception.
> 4. Check **recent deployments** or config changes (often the cause) and roll back if needed.
> 5. Check dependencies: database, Kafka lag, third-party API.
> 6. After the fix, write a short **post-mortem** and add an alert so you catch it earlier next time.

> [!question]- Q98. Scenario: How do you design a new microservice step by step? 🟡
> 1. **Define the purpose** and boundary (one business capability).
> 2. **Design the API first** (OpenAPI) and the events it publishes.
> 3. **Choose the data store** and design the schema with migrations.
> 4. **Build:** controller -> service -> repository, validation, error handling.
> 5. **Add resilience:** timeouts, retries, circuit breaker, idempotency.
> 6. **Add security:** token validation, roles.
> 7. **Add observability:** health, metrics, tracing, structured logs.
> 8. **Test:** unit, integration (Testcontainers), contract.
> 9. **Containerize and automate:** Dockerfile, CI/CD, Kubernetes manifests.
> 10. **Document:** README, API docs, runbook.

> [!question]- Q99. Scenario: The service is slow. How do you improve performance and use caching? 🟡
> 1. **Measure first:** use traces and metrics to find the slow part (DB query, remote call, GC).
> 2. **Database:** add indexes, fix N+1, use pagination, tune the connection pool.
> 3. **Cache** data that is read often and changes rarely (Redis). Use TTL (time to live) and plan **cache invalidation**.
> 4. **Parallelize** independent calls and use **async** for slow work.
> 5. Use **virtual threads** for I/O-heavy workloads and compress/trim large responses.
>
> 📖 **Simple words:**
> - **Cache** = fast temporary storage for data you read often.
> - **TTL** = how long a cached value stays valid.
>
> ```java
> @Cacheable(cacheNames = "products", key = "#id")
> public ProductResponse getProduct(Long id) { return mapper.toResponse(repository.findById(id).orElseThrow()); }
>
> @CacheEvict(cacheNames = "products", key = "#id")
> public void update(Long id, UpdateProductRequest req) { ... }
> // enable with @EnableCaching and spring-boot-starter-data-redis
> ```

> [!question]- Q100. What are the most common microservice mistakes and best practices? 🟡
> **Mistakes**
> - Splitting too small (nano-services) or sharing one database.
> - Long chains of synchronous calls (distributed monolith).
> - No timeouts, no monitoring, no automation.
> - Changing APIs without versioning.
> - Putting business logic in the gateway.
>
> **Best practices**
> - Model by business domain; one team owns a service.
> - Database per service; communicate by API and events.
> - Design for failure; make operations idempotent.
> - Automate build, test, deploy; observe everything.
> - Secure every service (zero trust); keep configuration outside the code.

---
Previous: [[12 - Docker Kubernetes and DevOps]] | Next: [[Glossary]]
