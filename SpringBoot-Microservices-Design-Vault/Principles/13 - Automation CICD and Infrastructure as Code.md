---
tags: [microservices, springboot, principle, devops]
principle: 13
status: to-review
created: 2026-11-04
---

# 13. Automation: CI/CD and Infrastructure as Code

> Every build, test, scan and deploy is automated and repeatable; infrastructure is versioned code.

Back to [[00 - Index (MOC)]]

## Why it matters
Independent deployability only pays off if releasing is cheap and safe.

## How to apply in Spring Boot
- One pipeline per service: build, unit tests, contract tests, image scan, deploy.
- Test pyramid: many unit tests, integration tests with Testcontainers, few E2E.
- GitOps (Argo CD / Flux) and Terraform/Helm for environments.
- Use progressive delivery: canary, blue-green, feature flags.

## Example
```java
@SpringBootTest
@Testcontainers
class OrderRepositoryIT {
    @Container @ServiceConnection
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");

    @Autowired OrderRepository repo;

    @Test void savesOrder() { assertThat(repo.save(new OrderEntity()).getId()).isNotNull(); }
}
```

## Common pitfalls
- Manual hotfixes on servers.
- Slow, flaky pipelines that people bypass.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[11 - Observability]]
- [[14 - Containerization and Elastic Scalability]]

---
Previous: [[12 - Security by Design]] | Next: [[14 - Containerization and Elastic Scalability]]
