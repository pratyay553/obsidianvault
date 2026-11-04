---
tags: [microservices, springboot, principle, config]
principle: 6
status: to-review
created: 2026-11-04
---

# 6. Externalized Configuration

> Configuration lives outside the artifact; the same build runs in every environment (12-factor, factor III).

Back to [[00 - Index (MOC)]]

## Why it matters
One immutable image promoted across dev, test, prod reduces 'works on my machine' bugs.

## How to apply in Spring Boot
- Use environment variables, profiles and `application-{profile}.yml`.
- Centralise with Spring Cloud Config, Kubernetes ConfigMaps or Consul.
- Keep secrets in Vault or Kubernetes Secrets, never in Git.
- Bind with type-safe `@ConfigurationProperties`.

## Example
```java
@ConfigurationProperties(prefix = "payment")
@Validated
public record PaymentProperties(@NotBlank String baseUrl,
                                @DefaultValue("2s") Duration timeout) {}
```

## Common pitfalls
- Hard-coded URLs and credentials.
- Environment-specific code branches (`if (prod)`).

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[14 - Containerization and Elastic Scalability]]
- [[12 - Security by Design]]

---
Previous: [[05 - Statelessness]] | Next: [[07 - Resilience and Design for Failure]]
