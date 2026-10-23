---
tags: [interview, cheat-sheet, summary]
---

# ⭐ Interview Cheat Sheet — Spring Boot Scheduler Module

Related: [[00-Home|Home]]. This note is a **fast-recall summary**. Click any topic to go deep.

## 🚀 30-Second Elevator Pitch (if asked "walk me through this project")
> "It's a standalone Spring Boot batch service — no web layer — where `@Scheduled` methods trigger jobs on cron/fixed-rate timers. Each job typically pulls data via JDBC/`JdbcTemplate`, calls external partner REST/SOAP APIs with retry logic for resilience, and pushes results out via email, Kafka, or cloud storage. Cross-cutting concerns like logging and timing are handled with AOP so business logic stays clean. It's configured for Spring Cloud (Eureka + Config Server) so it fits into a broader microservices environment, and uses Java 21 virtual threads so I/O-heavy jobs don't block each other."

## 🧠 One-Liner Definitions (rapid fire)

| Term | One-liner |
|---|---|
| `@EnableScheduling` | Turns on Spring's scheduling engine → [[01 - Spring Scheduling Basics]] |
| `@Scheduled` | Marks a method to run on cron/fixedRate/fixedDelay |
| `fixedRate` | Interval measured start-to-start; can overlap |
| `fixedDelay` | Interval measured end-to-start; never overlaps |
| `TaskScheduler` | Interface actually firing scheduled tasks |
| Virtual Threads | Lightweight JVM-managed threads, great for I/O-bound jobs → [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] |
| `@Configuration`/`@Bean` | How you manually wire non-`@Component` objects → [[03 - Configuration Classes Overview]] |
| `@Qualifier` | Disambiguates between multiple beans of the same type |
| AOP / `@Aspect` | Cross-cutting logic (logging, retries) applied without touching target code → [[04 - AOP - Aspect Oriented Programming]] |
| `@Around` + `ProceedingJoinPoint` | Most powerful advice — can skip/modify/retry the call |
| Retry pattern | Loop + `MAX_RETRIES`/`RETRY_DELAY_MS`, always restore interrupt flag → [[05 - Retry Pattern]] |
| `JdbcTemplate` / `RowMapper` | Boilerplate-free JDBC + row-to-object mapping → [[06 - Data Access Layer - JDBC and Repository Pattern]] |
| `RestTemplate` vs `WebClient` vs `@RestClient` | Blocking (legacy) vs reactive vs new declarative (3.2+) → [[07 - REST and SOAP Clients]] |
| `WebServiceTemplate` | SOAP client, pairs with JAXB marshalling |
| Eureka / Config Server / Cloud Bus | Discovery / centralized config / broadcast config refresh → [[08 - Spring Cloud and Microservices]] |
| `@RefreshScope` | Bean rebuilt live when config changes, no restart |
| Kafka vs RabbitMQ | Log-based streaming vs traditional broker/queue → [[09 - Kafka and RabbitMQ Messaging]] |
| `@Cacheable`/`@CacheEvict` | Declarative caching → [[10 - Redis Caching]] |
| MSAL4J + EWS | Modern OAuth2 auth for Microsoft 365 email → [[11 - Email Services]] |
| Apache POI | Excel generation/reading library → [[12 - File Processing]] |
| `@Async` self-invocation bug | Calling `this.method()` bypasses the proxy → [[13 - Concurrency - Async and Threads]] |
| `/actuator/scheduledtasks` | Endpoint listing all registered scheduled jobs → [[14 - Monitoring - Actuator and Logging]] |
| JWT | Signed (not encrypted) token; stateless auth → [[15 - Security - JWT OAuth2 MSAL]] |
| Client Credentials grant | OAuth2 flow for machine-to-machine (no user) auth |
| `@MockBean` vs `@Mock` | In-Spring-context mock vs plain Mockito mock → [[16 - Testing in Spring Boot]] |
| Jackson vs Gson vs JAXB | Default JSON / alt JSON / XML-SOAP serialization → [[17 - Serialization - Jackson Gson JAXB]] |
| `@Transactional` rollback default | Only rolls back on unchecked exceptions unless configured → [[18 - Database and Transactions]] |
| Multi-module split | core → engine/data/integration → service, monitoring cuts across → [[19 - Recommended Multi-Module Structure]] |

## 🔥 The "Gotcha" Questions Interviewers Love (self-invocation & proxies)
Spring's proxy-based features — **AOP, `@Async`, `@Transactional`, `@Cacheable`** — **all share the same limitation**: calling the annotated method via `this.` from inside the same class bypasses the proxy, so the behavior silently doesn't apply. If you can explain *why* (proxies only intercept calls coming from *outside* the bean, through the container-managed reference), you'll stand out. See it explained three ways: [[04 - AOP - Aspect Oriented Programming]], [[13 - Concurrency - Async and Threads]], [[18 - Database and Transactions]].

## 🧩 Likely System-Design Style Questions

**"How would you make this scheduler resilient if a downstream partner API goes down for an hour?"**
→ Combine [[05 - Retry Pattern]] (with exponential backoff) + circuit-breaker thinking (stop retrying after N consecutive failures, alert instead) + [[14 - Monitoring - Actuator and Logging]] to detect/alert, and consider queuing failed work (Kafka/RabbitMQ, [[09 - Kafka and RabbitMQ Messaging]]) for later reprocessing instead of losing it.

**"How would you scale this if the number of scheduled jobs grows 10x?"**
→ [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] to stop jobs blocking each other; consider splitting jobs across multiple service instances registered via [[08 - Spring Cloud and Microservices]] Eureka, with a distributed lock (e.g., ShedLock) to prevent the same job firing twice if you scale horizontally.

**"How do you make sure two instances of this service (scaled for HA) don't both run the same `@Scheduled` job at the same time?"**
→ This is a **very common follow-up** — mention **ShedLock** or **Quartz with a JDBC job store** as the standard solutions: they use a DB row as a distributed lock so only one instance actually executes a given scheduled task per trigger.

**"Why AOP for logging instead of just adding log statements in every method?"**
→ DRY + separation of concerns — see [[04 - AOP - Aspect Oriented Programming]]. One central place to change logging format/behavior for the entire service layer.

## ✅ Final Tip
If you get stuck on any acronym or class name during an interview, describe **what problem it solves** rather than freezing on the exact definition — that's exactly how every note in this vault is written, and it's what interviewers actually want to hear.
