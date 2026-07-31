---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: All
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Top 100 Core Spring Questions

> Concise Q&A bank. For deep-dives, follow the linked notes.

---

## 🔵 IoC & Dependency Injection (→ [[IoC-Container-And-DI]])

1. **What is Inversion of Control?**
   The container controls object creation; your class declares what it needs instead of creating it.

2. **What is the difference between IoC and DI?**
   IoC is the principle; DI is the mechanism Spring uses to implement it.

3. **Why prefer constructor injection over field injection?**
   Explicit deps, `final` fields, no Spring context needed for unit tests.

4. **When does Spring throw `NoUniqueBeanDefinitionException`?**
   Two or more beans match the required type with no `@Primary` or `@Qualifier`.

5. **What does `@Primary` do?**
   Marks the default bean when multiple candidates match the required type.

6. **What is `@Qualifier`?**
   Selects a specific bean by name at the injection point.

7. **What is `ApplicationContext` vs `BeanFactory`?**
   `ApplicationContext` adds AOP, events, `@PostConstruct`/`@PreDestroy`, i18n on top of basic IoC.

8. **When would you use `@Lazy`?**
   Defer expensive bean init or break constructor circular dependencies. Use sparingly.

9. **What is `@ConditionalOnProperty`?**
   Registers a bean only when a specific property value is present/matches.

---

## 🔵 Bean Lifecycle & Scopes (→ [[Bean-Lifecycle-And-Scopes]])

10. **What are the Spring bean scopes?**
    Singleton, Prototype, Request, Session, Application, WebSocket.

11. **What is the default scope?**
    Singleton — one instance per `ApplicationContext`.

12. **When does `@PostConstruct` fire?**
    After constructor + dependency injection, before the bean is exposed to the application.

13. **Does `@PreDestroy` fire for prototype beans?**
    No — Spring doesn't manage prototype bean destruction.

14. **How do you inject a prototype bean into a singleton?**
    `@Lookup`, `ObjectProvider<T>`, or `ApplicationContext.getBean()`.

15. **What does `@Scope("prototype")` do?**
    New instance created per injection point / `getBean()` call.

---

## 🔵 Component Scanning (→ [[Component-Scanning-And-Stereotypes]])

16. **What is `@SpringBootApplication` composed of?**
    `@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan`.

17. **What is the functional difference between `@Service` and `@Repository`?**
    `@Repository` adds persistence exception translation. `@Service` is semantic only.

18. **What does `@Configuration(proxyBeanMethods = false)` do?**
    Disables CGLIB proxying — `@Bean` methods can be called multiple times without singleton guarantee. Lighter weight.

19. **What happens if `@SpringBootApplication` is in a sub-package?**
    Sibling packages are not scanned. Place it in the root package.

---

## 🔵 Configuration & Properties (→ [[Configuration-And-Properties]])

20. **What is the property override order?**
    CLI args > env vars > profile YAML > base YAML > `@PropertySource` > defaults.

21. **`@Value` vs `@ConfigurationProperties` — when to use which?**
    `@ConfigurationProperties` for grouped config with validation; `@Value` for single scalars.

22. **How do you validate config properties at startup?**
    Add `@Validated` + Jakarta constraints to the `@ConfigurationProperties` class.

23. **How do you activate a Spring profile?**
    `spring.profiles.active=prod` in YAML, `SPRING_PROFILES_ACTIVE=prod` env var, or `--spring.profiles.active=prod`.

---

## 🔵 REST Controllers (→ [[Controllers-And-Request-Mapping]], [[RESTful-API-Design]])

24. **`@Controller` vs `@RestController`?**
    `@RestController` = `@Controller` + `@ResponseBody` on every method.

25. **`@PathVariable` vs `@RequestParam`?**
    `@PathVariable` extracts URI segment; `@RequestParam` extracts query string value.

26. **How do you return a 201 Created with a Location header?**
    Return `ResponseEntity.created(uri).body(dto)`.

27. **What HTTP status should a successful DELETE return?**
    204 No Content (no body).

28. **What is idempotency and which verbs are idempotent?**
    Same result regardless of call count. GET, PUT, DELETE are idempotent; POST is not.

---

## 🔵 Validation (→ [[Request-Validation-Jakarta]])

29. **`@NotNull` vs `@NotEmpty` vs `@NotBlank`?**
    `@NotNull` — not null. `@NotEmpty` — not null + not empty. `@NotBlank` — not null + not empty + not whitespace.

30. **How do you return structured validation errors?**
    Handle `MethodArgumentNotValidException` in `@RestControllerAdvice`, map `getFieldErrors()`.

31. **How do you validate service-layer method parameters?**
    `@Validated` on the class + constraint annotations on method parameters. Throws `ConstraintViolationException`.

---

## 🔵 Exception Handling (→ [[Global-Exception-Handling]])

32. **What is `@RestControllerAdvice`?**
    `@ControllerAdvice` + `@ResponseBody` — centralized JSON error response handler.

33. **Why doesn't `@RestControllerAdvice` catch Spring Security exceptions?**
    Security exceptions fire in servlet filters before the dispatcher. Handle via `AuthenticationEntryPoint` / `AccessDeniedHandler`.

34. **What is RFC 7807?**
    Standard JSON error format: `type`, `title`, `status`, `detail`, `instance`. Spring Boot 3 ships `ProblemDetail`.

---

## 🔵 Spring Data JPA (→ [[Spring-Data-JPA-Fundamentals]], [[Entity-Mapping-And-Relationships]])

35. **`Page` vs `Slice`?**
    `Page` runs a COUNT query. `Slice` doesn't — use for infinite scroll.

36. **What does `@Modifying` do?**
    Marks a `@Query` as UPDATE/DELETE (not SELECT). Requires `@Transactional` on caller.

37. **When do you need `countQuery` on a `@Query`?**
    When returning `Page<T>` and JPQL is too complex for Spring to auto-derive the count.

38. **Default fetch type for `@ManyToOne`?**
    `EAGER` — always override to `LAZY`.

39. **What is the N+1 problem and how do you fix it?**
    N entities → N extra queries for lazy associations. Fix: `JOIN FETCH`, `@EntityGraph`, or `@BatchSize`.

40. **What does `mappedBy` mean?**
    Marks the inverse side of a relationship — changes here are NOT persisted. The FK is owned by the other side.

41. **`CascadeType.REMOVE` on `@ManyToMany` — why is it dangerous?**
    Deletes all related entities, including ones shared with other parents.

42. **What is `orphanRemoval = true`?**
    Deletes a child entity when removed from the parent's collection. Never use on `@ManyToMany`.

---

## 🔵 Transactions (→ [[Transaction-Management-Transactional]])

43. **Why doesn't `@Transactional` work on `private` methods?**
    Spring AOP uses CGLIB proxies that can't override private methods.

44. **Why doesn't `@Transactional` work on self-invocation?**
    Calls to `this.method()` bypass the proxy. Extract to a separate bean.

45. **Default rollback behaviour?**
    Rolls back on `RuntimeException` and `Error`. Not on checked exceptions.

46. **`REQUIRED` vs `REQUIRES_NEW` propagation?**
    `REQUIRED` joins existing tx. `REQUIRES_NEW` suspends it and starts an independent one.

47. **What does `readOnly = true` do?**
    Hibernate skips dirty checking; DB may route to read replica. Does not enforce immutability.

48. **What is optimistic locking?**
    `@Version` field — JPA checks version on update; throws `OptimisticLockingFailureException` if stale.

---

## 🔵 Spring Security (→ [[Spring-Security-Architecture-Filters]], [[Asymmetric-JWT-RS256-Implementation]])

49. **What is `DelegatingFilterProxy`?**
    Bridges servlet container filters with Spring beans — delegates to `FilterChainProxy`.

50. **What is `OncePerRequestFilter`?**
    Guarantees a filter runs exactly once per request even for async/forward dispatches.

51. **HS256 vs RS256 JWT?**
    HS256: shared secret (everyone who verifies can also forge). RS256: private key signs, public key verifies — correct for multi-service.

52. **Where should refresh tokens be stored in the browser?**
    `HttpOnly; Secure; SameSite=Strict` cookie — inaccessible to JavaScript.

53. **What is token rotation?**
    Invalidate the used refresh token and issue a new one on each use. Reuse = theft detection.

54. **`hasRole('ADMIN')` vs `hasAuthority('ROLE_ADMIN')`?**
    `hasRole` auto-prepends `ROLE_`. `hasAuthority` matches the exact string.

55. **`@PreAuthorize` vs `@Secured`?**
    `@PreAuthorize` uses SpEL — supports parameters, bean calls, return values. `@Secured` is role-only, no SpEL.

56. **Why does CORS fail with `allowedOrigins("*")` and `allowCredentials(true)`?**
    Browsers reject wildcard origin with credentials — specify exact origins.

57. **Where should CORS be configured when Spring Security is active?**
    In `CorsConfigurationSource` bean + `http.cors(...)`. Security filter runs before MVC.

---

## 🔵 Microservices (→ [[Service-Discovery-Eureka]], [[Resiliency-Circuit-Breaker-Resilience4j]])

58. **What is service discovery?**
    Services self-register in a registry (Eureka) and query it by logical name — no hardcoded URLs.

59. **Client-side vs server-side load balancing?**
    Client-side: client picks instance from registry. Server-side: dedicated LB (Nginx, ALB) distributes traffic.

60. **What is a circuit breaker?**
    Monitors failure rate; OPEN state short-circuits calls to prevent cascading failures.

61. **Three states of a circuit breaker?**
    CLOSED (normal) → OPEN (fail fast) → HALF_OPEN (probe recovery).

62. **What is a Bulkhead?**
    Limits concurrent calls to a downstream service — isolates failure blast radius.

63. **What is a Bulkhead vs CircuitBreaker?**
    Bulkhead limits concurrency. Circuit breaker limits call rate based on error rate.

64. **What is an API Gateway and what does it do?**
    Single entry point: routes requests, enforces auth, rate limiting, circuit breaking, logging.

65. **Feign vs gRPC?**
    Feign: declarative HTTP/REST, easy Spring integration. gRPC: binary/HTTP2, high throughput, bidirectional streaming.

---

## 🔵 Performance & Caching (→ [[Redis-Caching-Strategies]], [[Async-Processing-Async-CompletableFuture]])

66. **`@Cacheable` vs `@CachePut`?**
    `@Cacheable` skips method on cache hit. `@CachePut` always runs method + updates cache.

67. **What is a cache stampede?**
    Many concurrent misses hitting DB simultaneously when a popular entry expires.

68. **How do you prevent cache stampede?**
    Redis distributed lock, refresh-ahead, or probabilistic early expiration.

69. **Default executor for `@Async`?**
    `SimpleAsyncTaskExecutor` — unbounded thread creation. Always replace with `ThreadPoolTaskExecutor`.

70. **How do you propagate `SecurityContext` to async threads?**
    `SecurityContextHolder.setStrategyName(MODE_INHERITABLETHREADLOCAL)` or `DelegatingSecurityContextExecutor`.

71. **`CompletableFuture.allOf()` vs `anyOf()`?**
    `allOf` waits for all to complete. `anyOf` completes when the first one does.

---

## 🔵 Testing (→ [[Unit-And-Integration-Testing-Testcontainers]])

72. **`@Mock` vs `@MockBean`?**
    `@Mock` is pure Mockito, no Spring. `@MockBean` registers the mock in the Spring context.

73. **Why use Testcontainers instead of H2?**
    H2 doesn't support PostgreSQL-specific types/functions. Testcontainers runs the real engine.

74. **`@WebMvcTest` vs `@SpringBootTest`?**
    `@WebMvcTest` loads only MVC layer — faster, isolated. `@SpringBootTest` loads full context.

75. **What is `@DataJpaTest`?**
    Loads only JPA repos, entities, DataSource — no controllers or services.

76. **What is Spring test context caching?**
    Spring reuses `ApplicationContext` across test classes with the same config to avoid reload.

77. **Why does `@MockBean` break context caching?**
    Different `@MockBean` configurations create different context keys → new context per combo.

---

## 🔵 DevOps & Deployment (→ [[Docker-Kubernetes-Deployment]], [[Actuator-Metrics-Prometheus-Grafana]])

78. **What is a multi-stage Dockerfile?**
    Build in a JDK image, copy artifact to minimal JRE image — 60-80% smaller production image.

79. **Liveness vs Readiness probe?**
    Liveness: alive (restart if fails). Readiness: ready for traffic (remove from LB if fails).

80. **What is a startup probe?**
    Gives slow-starting apps time before liveness kicks in — prevents restart loops.

81. **`-XX:MaxRAMPercentage=75.0` — what does it do?**
    JVM auto-detects container memory limit and uses 75% as max heap. Better than hardcoded `-Xmx`.

82. **What is Micrometer?**
    Vendor-neutral metrics facade (the "SLF4J for metrics"). Pluggable backends: Prometheus, Datadog, etc.

83. **Counter vs Gauge vs Timer?**
    Counter: monotonically increasing. Gauge: current instantaneous value. Timer: durations + count + percentiles.

84. **What does Spring Boot Actuator expose?**
    `/actuator/health`, `/actuator/metrics`, `/actuator/prometheus`, `/actuator/loggers`, etc.

---

## 🔵 General Architecture

85. **What is the difference between `@Transactional` on service vs repository?**
    Service should own transactions. Repository participates. Repository `@Transactional` creates unnecessary nesting.

86. **What is eventual consistency?**
    System guarantees data will become consistent over time (not immediately). Used with async messaging (Kafka, RabbitMQ).

87. **What is the Outbox pattern?**
    Write business entity + outbox message in one DB transaction. Background process publishes the message. Prevents dual-write.

88. **What is idempotency key and why use it?**
    Client-generated UUID sent with mutating requests. Server deduplicates retried requests using the key.

89. **What is a saga pattern?**
    Sequence of local transactions across services, each publishing events. Compensating transactions roll back on failure.

90. **What is the difference between orchestration and choreography?**
    Orchestration: central orchestrator (saga service) directs each step. Choreography: services react to events independently.

91. **What is CQRS?**
    Command Query Responsibility Segregation — separate write model (Commands) from read model (Queries). Often paired with Event Sourcing.

92. **What is Event Sourcing?**
    Store state as a sequence of immutable events, not mutable rows. Current state derived by replaying events.

93. **What is a dead letter queue (DLQ)?**
    Messages that fail processing after max retries are routed here for inspection and manual remediation.

94. **What is a blue-green deployment?**
    Run old (blue) and new (green) versions simultaneously. Switch traffic to green once validated. Instant rollback by switching back.

95. **What is a canary deployment?**
    Route a small percentage of traffic to the new version, monitor, gradually increase to 100%.

96. **What is distributed tracing?**
    Correlate logs and spans across services using a shared `traceId` (e.g., via Zipkin, Jaeger, OpenTelemetry).

97. **What is a connection pool and why does it matter?**
    Reuses DB connections instead of creating one per request — creating a TCP + TLS connection takes 50-200ms.

98. **What is HikariCP?**
    Spring Boot's default JDBC connection pool — fastest in benchmarks. Configured via `spring.datasource.hikari.*`.

99. **What is the difference between horizontal and vertical scaling?**
    Vertical: more CPU/RAM on one machine. Horizontal: more instances. Horizontal is preferred for microservices (stateless apps).

100. **What is CAP theorem?**
    A distributed system can guarantee at most two of: Consistency, Availability, Partition Tolerance. Eureka = AP; ZooKeeper = CP.
