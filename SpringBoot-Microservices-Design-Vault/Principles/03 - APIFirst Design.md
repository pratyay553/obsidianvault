---
tags: [microservices, springboot, principle, api]
principle: 3
status: to-review
created: 2026-11-04
---

# 3. API-First Design

> Design the contract (OpenAPI / AsyncAPI) before implementation, and treat it as the product.

Back to [[00 - Index (MOC)]]

## Why it matters
Consumers and producers can work in parallel; contracts become testable and reviewable.

## How to apply in Spring Boot
- Write `openapi.yaml` first, review it, then generate server stubs with `openapi-generator`.
- Use consistent REST conventions: nouns, plural resources, correct status codes, RFC 7807 Problem Details.
- Add consumer-driven contract tests (Spring Cloud Contract or Pact).
- Publish docs with springdoc-openapi.

## Example
```java
@RestControllerAdvice
class ApiErrors {
    @ExceptionHandler(OrderNotFoundException.class)
    ProblemDetail notFound(OrderNotFoundException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        pd.setTitle("Order not found");
        return pd;
    }
}
```

## Common pitfalls
- Leaking JPA entities as API models.
- Inconsistent error formats between services.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[04 - Loose Coupling and High Cohesion]]
- [[15 - Versioning and Backward Compatibility]]

---
Previous: [[02 - Database per Service]] | Next: [[04 - Loose Coupling and High Cohesion]]
