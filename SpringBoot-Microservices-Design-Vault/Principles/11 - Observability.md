---
tags: [microservices, springboot, principle, operations]
principle: 11
status: to-review
created: 2026-11-04
---

# 11. Observability

> Make the system explain itself: structured logs, metrics and distributed traces, correlated by trace id.

Back to [[00 - Index (MOC)]]

## Why it matters
You cannot debug what you cannot see, especially across dozens of services.

## How to apply in Spring Boot
- Spring Boot Actuator + Micrometer for metrics, exported to Prometheus.
- Micrometer Tracing with OpenTelemetry; propagate trace context across HTTP and messaging.
- JSON logs including `traceId` and `spanId`.
- Define SLIs/SLOs and alert on symptoms, not causes.

## Example
```yaml
management:
  endpoints.web.exposure.include: health,info,prometheus
  tracing.sampling.probability: 0.1
  metrics.tags.application: ${spring.application.name}
logging.pattern.level: "%5p [${spring.application.name},%X{traceId},%X{spanId}]"
```

## Common pitfalls
- Logging PII or secrets.
- High-cardinality metric labels.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[07 - Resilience and Design for Failure]]
- [[13 - Automation CICD and Infrastructure as Code]]

---
Previous: [[10 - Distributed Data Consistency Saga and Outbox]] | Next: [[12 - Security by Design]]
