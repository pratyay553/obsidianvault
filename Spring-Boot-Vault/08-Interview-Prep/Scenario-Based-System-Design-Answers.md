---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Advanced
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Scenario-Based System Design Answers

> Talk-track answers for common system design interview questions — structured as bullet-point outlines you can expand in conversation.

---

## 🏗️ Scenario 1: Design a URL Shortener

**Key interviewer signals:** scale, uniqueness, redirects, analytics

**Approach:**
- **Write path:** Client → API Gateway → Shortener Service → generate 7-char Base62 ID (counter via Redis `INCR` or nanoid) → store `{shortCode: originalUrl}` in PostgreSQL
- **Read path (hot path):** Client → Gateway → `GET /r/{code}` → check Redis (O(1)) → miss → hit PostgreSQL → 301/302 redirect → cache result
- **Scale:** Redis handles 100k+ reads/sec; DB holds ground truth; replicas for read-heavy load
- **Analytics:** Push click events to Kafka → consumer writes to ClickHouse/BigQuery
- **Custom aliases:** Check uniqueness in DB before accepting
- **Expiry:** TTL in both Redis and DB; background job purges expired entries
- **301 vs 302:** 301 (permanent) caches in browser — no analytics on repeat visits. 302 (temporary) always hits our server — use 302 for analytics

---

## 🏗️ Scenario 2: Design a Rate Limiter

**Key signals:** fairness, distributed, algorithms

**Approach:**
- **Algorithms:**
  - Token Bucket — smooth bursts; `redis.incrBy(key, 1)` + TTL; allows burst up to capacity
  - Sliding Window Log — exact, high memory; store timestamps in Redis sorted set
  - Fixed Window — simple, has edge-case burst at window boundary
- **Spring implementation:** Spring Cloud Gateway `RequestRateLimiter` filter + Redis `RedisRateLimiter`
- **Key granularity:** Per IP, per user ID, per API key — choose based on auth state
- **Headers:** Return `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` in response
- **Distributed:** Redis ensures shared state across gateway replicas — a single Redis (or Redis Cluster) is the source of truth
- **Response:** 429 Too Many Requests + `Retry-After` header

---

## 🏗️ Scenario 3: Design a Notification Service

**Key signals:** fan-out, reliability, deduplication, multi-channel

**Architecture:**
```
Order Service → Kafka (order.placed) → Notification Service
                                         ├─ Email Worker (SendGrid)
                                         ├─ SMS Worker (Twilio)
                                         └─ Push Worker (FCM/APNs)
```

**Design decisions:**
- **Kafka** for decoupling — producers don't know about notification logic; consumers scale independently per channel
- **At-least-once delivery** — Kafka guarantees; use idempotency key (`eventId + channel`) to deduplicate in a `notifications_sent` DB table
- **Preferences:** User notification preferences stored in DB — filter before sending
- **Retry:** Dead-letter topic for failed deliveries after 3 attempts; manual review via admin UI
- **Template engine:** Thymeleaf or Mustache for email templates; stored in DB for dynamic updates
- **Priority:** Critical notifications (password reset, fraud alert) go to a high-priority Kafka partition
- **Observability:** Track delivery rate, bounce rate, DLQ depth as custom Prometheus metrics

---

## 🏗️ Scenario 4: Design an Auth Service (RS256 JWT)

*See also: [[Auth-Service-Architecture-Elevator-Pitch]]*

**Endpoints:**
- `POST /auth/register` — hash password (BCrypt 12 rounds), store user
- `POST /auth/login` — verify password, issue access token (15 min RS256 JWT) + refresh token (30 days, stored hashed in DB)
- `POST /auth/refresh` — validate refresh token, rotate (invalidate old, issue new)
- `POST /auth/logout` — revoke refresh token family
- `GET /auth/.well-known/jwks.json` — public key endpoint for resource servers

**Token strategy:**
- Access token: short-lived stateless JWT — resource servers verify with public key, no DB call
- Refresh token: opaque random string, stored as SHA-256 hash in DB with `familyId` for theft detection

**Resource servers:**
- Fetch public key from JWKS endpoint at startup, cache with periodic refresh
- Validate `iss`, `aud`, `exp`, `iat` claims

**Scaling:**
- Auth service is stateless (JWT private key in secrets manager) → horizontal scaling
- DB: primary for writes, read replica for token lookups
- Rate limit `POST /auth/login` by IP to prevent brute force

---

## 🏗️ Scenario 5: Design an E-Commerce Order Service

**Key signals:** inventory, payments, consistency, failure handling

**Order lifecycle:** PENDING → RESERVED → PAID → FULFILLING → SHIPPED → DELIVERED

**Saga (Choreography):**
1. `POST /orders` → create order (PENDING) → publish `OrderCreated` event
2. Inventory Service consumes `OrderCreated` → reserve stock → publish `StockReserved` or `StockFailed`
3. Payment Service consumes `StockReserved` → charge → publish `PaymentSuccess` or `PaymentFailed`
4. Order Service consumes `PaymentSuccess` → mark PAID; consumes `PaymentFailed` → compensate (release stock) → mark FAILED

**Outbox pattern** (prevent dual-write):
- In ONE transaction: write order row + write to `outbox` table
- Background job publishes outbox messages to Kafka, marks as published

**Idempotency:**
- Client sends `Idempotency-Key` header
- Server stores `(idempotencyKey, response)` in Redis for 24h
- Duplicate request → return cached response

**Inventory concurrency:**
- `SELECT stock FROM inventory WHERE product_id = ? FOR UPDATE` (pessimistic lock) or
- `UPDATE inventory SET stock = stock - ? WHERE product_id = ? AND stock >= ?` (optimistic, check rowcount)

---

## 🏗️ Scenario 6: Design a Feed / Timeline (Twitter-Like)

**Key signals:** fan-out, celebrity problem, read vs write heavy

**Push model (fan-out on write):**
- User posts → write to followers' feed timelines in Redis sorted set (score = timestamp)
- Read: `ZREVRANGE feed:{userId} 0 49` — O(log n)
- **Problem:** Users with 10M followers → 10M writes on one post (write amplification)

**Pull model (fan-out on read):**
- User posts → write to own timeline only
- Read: fetch from all followees' timelines and merge
- **Problem:** High read latency for users following 500 accounts

**Hybrid (production approach):**
- Regular users: push model (fan-out on write)
- Celebrities (>threshold followers): pull model on read
- Merge celebrity posts with pre-computed follower feed at read time
- Cache merged feed in Redis with 5-minute TTL

**Technologies:**
- Posts: PostgreSQL (source of truth)
- Timelines: Redis sorted sets (feed cache)
- Fan-out: Kafka consumer group (async writes to follower timelines)
- Media: S3 + CDN

---

## 🏗️ Scenario 7: Design a Distributed Cache

**Key signals:** consistency, eviction, distributed, invalidation

**Levels:**
- L1: In-process Caffeine cache (microsecond, per-instance)
- L2: Redis cluster (millisecond, shared across all instances)
- L3: Database (source of truth)

**Write strategies:**
- Cache-aside: app manages cache, writes to DB + Redis separately (most common)
- Write-through: write to cache → DB synchronously (consistency, slower writes)
- Write-behind: write to cache → async flush to DB (high write throughput, risk of data loss)

**Invalidation:**
- TTL-based: simplest, eventual consistency acceptable
- Event-driven: DB change event → Kafka → cache invalidation consumer
- Tag-based: group related cache entries by tag, invalidate all by tag

**Cache eviction policies:**
- LRU (Least Recently Used) — Redis default
- LFU (Least Frequently Used) — better for skewed access patterns
- TTL only — simplest, appropriate for time-sensitive data

**Consistency guarantee:**
- Strong: Write-through, accept latency hit
- Eventual: TTL + cache-aside, accept temporary staleness
- Never guarantee strong consistency with separate cache + DB without distributed transactions

---

## 🏗️ Scenario 8: Diagnose Slow API Endpoint (Troubleshooting)

**Structured approach:**

1. **Measure:** Reproduce with `curl -w "%{time_total}" http://...` or check Grafana P99 latency
2. **Identify layer:**
   - Network? Check latency to other endpoints
   - App? Enable Actuator metrics for this route
   - DB? `EXPLAIN ANALYZE` the queries, check `pg_stat_statements`
3. **Check N+1:** Enable `show_sql=true` — count queries per request
4. **Check missing index:** `EXPLAIN` shows Seq Scan → add index
5. **Check external calls:** Are calls to downstream services timing out? Check circuit breaker metrics
6. **Check JVM:** GC pauses? `jvm.gc.pause` metric. Memory pressure? Heap dump analysis
7. **Check connection pool:** `hikaricp.connections.active` at or near max? Increase pool size or reduce query time
8. **Fix:** Add `JOIN FETCH` / index / cache / async → measure again

---

## Quick Reference: When to Use What

| Problem | Solution |
|---------|---------|
| Repeated expensive computation | `@Cacheable` + Redis |
| Many concurrent service calls | `CompletableFuture.allOf()` + `@Async` |
| Downstream service failing | Resilience4j CircuitBreaker + fallback |
| N+1 queries | `JOIN FETCH` or `@EntityGraph` |
| High write volume to many consumers | Kafka fan-out |
| Duplicate message processing | Idempotency key + dedup table |
| Service-to-service latency | gRPC (binary/HTTP2) |
| Dynamic queries with many optional filters | `JpaSpecificationExecutor` |
| Blue-green / zero-downtime deploy | K8s rolling update + readiness probes |
| Stale cache serving incorrect data | `@CacheEvict` on update + short TTL |
