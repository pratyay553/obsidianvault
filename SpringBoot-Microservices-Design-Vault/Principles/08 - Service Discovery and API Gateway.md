---
tags: [microservices, springboot, principle, infrastructure]
principle: 8
status: to-review
created: 2026-11-04
---

# 8. Service Discovery and API Gateway

> Clients find services dynamically, and a single gateway handles edge concerns like routing, auth, rate limiting.

Back to [[00 - Index (MOC)]]

## Why it matters
Instances come and go; hard-coded addresses break. A gateway keeps cross-cutting edge logic out of services.

## How to apply in Spring Boot
- On Kubernetes use built-in DNS and Services; otherwise Eureka or Consul.
- Use Spring Cloud Gateway as the edge: routing, rate limits, token relay.
- Consider a BFF (Backend for Frontend) per client type.
- Keep business logic out of the gateway.

## Example
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: orders
          uri: lb://order-service
          predicates: [ Path=/api/orders/** ]
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args: { redis-rate-limiter.replenishRate: 20, redis-rate-limiter.burstCapacity: 40 }
```

## Common pitfalls
- Gateway becoming a new monolith.
- Single gateway instance as a SPOF.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[07 - Resilience and Design for Failure]]
- [[12 - Security by Design]]

---
Previous: [[07 - Resilience and Design for Failure]] | Next: [[09 - Asynchronous and EventDriven Communication]]
