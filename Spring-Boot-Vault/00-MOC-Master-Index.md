# 🗺️ Spring Boot Mastery Vault — Master Index

> Use this file as your navigation hub. Every topic links back here. Track progress with the `status` field in each note's front matter and query it via Dataview.

---

## 📊 Dataview Progress Dashboard

```dataview
TABLE technology, level, status, last_reviewed
FROM "Spring-Boot-Vault"
WHERE file.name != "00-MOC-Master-Index"
SORT status ASC
```

---

## 📁 01 · Core Spring Framework
| Note | Level | Focus |
|------|-------|-------|
| [[IoC-Container-And-DI]] | Intermediate | Bean wiring, constructor injection, ApplicationContext |
| [[Bean-Lifecycle-And-Scopes]] | Intermediate | Singleton vs Prototype, PostConstruct, PreDestroy |
| [[Component-Scanning-And-Stereotypes]] | Beginner | @Component, @Service, @Repository, @Controller |
| [[Configuration-And-Properties]] | Intermediate | @ConfigurationProperties, profiles, externalized config |

## 📁 02 · Web & REST APIs
| Note | Level | Focus |
|------|-------|-------|
| [[RESTful-API-Design]] | Intermediate | Resource modeling, HTTP verbs, status codes |
| [[Controllers-And-Request-Mapping]] | Beginner | @RestController, @PathVariable, @RequestBody |
| [[Request-Validation-Jakarta]] | Intermediate | @Valid, ConstraintViolation, custom validators |
| [[Global-Exception-Handling]] | Intermediate | @ControllerAdvice, ProblemDetail (RFC 7807) |
| [[CORS-And-Headers]] | Intermediate | CorsConfiguration, security headers |

## 📁 03 · Data Access & Persistence
| Note | Level | Focus |
|------|-------|-------|
| [[Spring-Data-JPA-Fundamentals]] | Intermediate | JpaRepository, Pageable, Specifications |
| [[Entity-Mapping-And-Relationships]] | Intermediate | @OneToMany, fetch types, cascade |
| [[JPQL-Native-Queries-Derived-Methods]] | Advanced | Query methods, @Query, projections |
| [[Transaction-Management-Transactional]] | Advanced | Propagation, isolation, rollback rules |
| [[MongoDB-Integration]] | Intermediate | MongoRepository, @Document, aggregations |

## 📁 04 · Security & Auth
| Note | Level | Focus |
|------|-------|-------|
| [[Spring-Security-Architecture-Filters]] | Advanced | SecurityFilterChain, filter ordering |
| [[Asymmetric-JWT-RS256-Implementation]] | Advanced | RS256 keys, JwtDecoder, token validation |
| [[Refresh-Tokens-And-Session-Management]] | Advanced | Rotation strategy, token revocation |
| [[Method-Level-Security-PreAuthorize]] | Intermediate | @PreAuthorize, SpEL, @Secured |

## 📁 05 · Microservices & Distributed Systems
| Note | Level | Focus |
|------|-------|-------|
| [[Service-Discovery-Eureka]] | Intermediate | Client registration, load balancing |
| [[API-Gateway-Spring-Cloud-Gateway]] | Advanced | Route predicates, filters, rate limiting |
| [[Resiliency-Circuit-Breaker-Resilience4j]] | Advanced | CircuitBreaker, Retry, Bulkhead, TimeLimiter |
| [[Inter-Service-Communication-Feign-gRPC]] | Advanced | Feign clients, gRPC stubs, error handling |

## 📁 06 · Performance, Caching & Async
| Note | Level | Focus |
|------|-------|-------|
| [[Redis-Caching-Strategies]] | Advanced | @Cacheable, TTL, eviction, cache-aside |
| [[Async-Processing-Async-CompletableFuture]] | Advanced | @Async, thread pools, CompletableFuture chaining |
| [[Database-Optimization-NPlus1-Indexing]] | Advanced | N+1 problem, batch fetching, index strategy |

## 📁 07 · DevOps, Testing & Production
| Note | Level | Focus |
|------|-------|-------|
| [[Unit-And-Integration-Testing-Testcontainers]] | Advanced | @SpringBootTest, MockMvc, Testcontainers |
| [[Actuator-Metrics-Prometheus-Grafana]] | Intermediate | Health, metrics, custom endpoints |
| [[Docker-Kubernetes-Deployment]] | Advanced | Multi-stage Dockerfile, K8s manifests |

## 📁 08 · Interview Prep
| Note | Level | Focus |
|------|-------|-------|
| [[Top-100-Core-Spring-Questions]] | All | Concise Q&A bank |
| [[Scenario-Based-System-Design-Answers]] | Advanced | Full system design walkthroughs |
| [[Auth-Service-Architecture-Elevator-Pitch]] | Advanced | 5-minute auth architecture explanation |

---

*Last updated: 2026-07-23*
