---
tags: [microservices, springboot, principle, api]
principle: 15
status: to-review
created: 2026-11-04
---

# 15. Versioning and Backward Compatibility

> Evolve contracts without breaking consumers: additive changes, explicit versions, and deprecation policy.

Back to [[00 - Index (MOC)]]

## Why it matters
Services deploy independently, so old and new versions coexist in production.

## How to apply in Spring Boot
- Follow the expand and contract pattern for APIs, events and DB schemas.
- Tolerant reader: ignore unknown fields (`FAIL_ON_UNKNOWN_PROPERTIES=false`).
- Version in the URL (`/v1/`) or header; run old and new in parallel until usage hits zero.
- Automate compatibility checks (contract tests, schema registry rules).

## Example
```java
@RestController
@RequestMapping("/v2/orders")
class OrderControllerV2 { /* new shape; /v1 stays until consumers migrate */ }

// application.yml
// spring.jackson.deserialization.fail-on-unknown-properties: false
```

## Common pitfalls
- Breaking changes without notice.
- Dropping a DB column in the same release as the code change.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[03 - APIFirst Design]]
- [[09 - Asynchronous and EventDriven Communication]]

---
Previous: [[14 - Containerization and Elastic Scalability]] | Next: -
