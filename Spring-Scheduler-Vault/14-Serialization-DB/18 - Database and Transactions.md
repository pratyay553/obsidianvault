---
tags: [database, transactions, jdbc, interview]
---

# Database & Transactions

Related: [[06 - Data Access Layer - JDBC and Repository Pattern]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Beyond just running queries, a scheduler doing multi-step data operations (read → transform → write across multiple tables) needs **transactions** to ensure partial failures don't leave data in a broken state.

## 🔑 Core Pieces

| Component | Role |
|---|---|
| **PostgreSQL driver** | JDBC driver that lets Java talk to a PostgreSQL database |
| **Connection pooling** (e.g., HikariCP, Spring Boot's default) | Reuses a pool of open DB connections instead of opening/closing one per query — expensive to create, so reuse matters a lot |
| `@Transactional` | Wraps a method in a database transaction — commits on success, rolls back on exception |

## 🛠️ `@Transactional` Example

```java
@Service
public class TripSyncService {

    @Transactional
    public void syncTrip(Trip trip) {
        tripRepository.save(trip);
        tripStatusRepository.updateStatus(trip.getId(), "SYNCED");
        auditRepository.logSync(trip.getId());
        // if ANY of the above throws, ALL are rolled back together
    }
}
```

### Key defaults to know
- By default, `@Transactional` only rolls back on **unchecked exceptions** (`RuntimeException`/`Error`) — a checked exception does **not** trigger rollback unless you explicitly configure `rollbackFor = Exception.class`.
- `@Transactional` on a **private method** or via **self-invocation** silently does nothing — same proxy limitation as `@Async`/AOP (see [[13 - Concurrency - Async and Threads]] and [[04 - AOP - Aspect Oriented Programming]]), because Spring implements it via a proxy wrapping the bean.

## 🏊 Why connection pooling matters for a scheduler
A scheduled job that opens a raw `Connection` per query, especially one looping over thousands of rows, will exhaust connections or crawl due to connection-setup overhead. Pooling (HikariCP, auto-configured by Spring Boot when a `DataSource` is on the classpath) keeps a warm set of ready connections.

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/scheduler_db
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
```

## ❓ Interview Q&A

**Q: Why doesn't `@Transactional` roll back on a checked exception by default?**
A: Spring's default rollback policy mirrors EJB conventions: unchecked exceptions (`RuntimeException`) usually indicate unexpected failures worth rolling back for, while checked exceptions are often treated as expected/recoverable business outcomes. Override with `@Transactional(rollbackFor = Exception.class)` if you need checked exceptions to roll back too.

**Q: What's the "self-invocation" problem with `@Transactional`, concretely?**
A: If method `A()` (no `@Transactional`) calls `this.B()` (which *is* `@Transactional`) within the *same class*, the transactional advice never runs — because the call goes directly to `this`, bypassing the Spring-generated proxy that would normally intercept it and start the transaction.

**Q: How many connections should a connection pool have for a scheduler with several `@Scheduled` jobs?**
A: There's no universal number — it depends on how many jobs can run concurrently (especially relevant with [[02 - Virtual Threads and SimpleAsyncTaskScheduler]], where many jobs can now genuinely overlap) and how long each holds a connection. Undersizing causes jobs to wait/timeout for a connection; oversizing wastes DB-side resources. Size it to your realistic peak concurrent DB usage, then load-test.

**Q: What does "connection pooling" actually save you, technically?**
A: Establishing a new DB connection involves a TCP handshake, authentication, and session setup on the DB server — all relatively expensive compared to running a query. Pooling amortizes that cost by keeping connections open and handing them out/back rather than recreating them per request.
