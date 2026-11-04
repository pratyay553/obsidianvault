---
tags: [microservices, springboot, principle, scalability]
principle: 14
status: to-review
created: 2026-11-04
---

# 14. Containerization and Elastic Scalability

> Package each service as a small, immutable container and let the platform scale and heal it.

Back to [[00 - Index (MOC)]]

## Why it matters
Consistent runtime, fast startup, and orchestrator-driven scaling and self-healing.

## How to apply in Spring Boot
- Build images with Buildpacks (`spring-boot:build-image`) or layered jars; run as non-root.
- Expose liveness and readiness probes via Actuator.
- Set CPU/memory requests and limits; configure JVM for containers.
- Autoscale with HPA; handle SIGTERM with graceful shutdown.

## Example
```yaml
# application.yml
server.shutdown: graceful
spring.lifecycle.timeout-per-shutdown-phase: 30s
management.endpoint.health.probes.enabled: true
```
```yaml
# k8s deployment snippet
livenessProbe:  { httpGet: { path: /actuator/health/liveness,  port: 8080 } }
readinessProbe: { httpGet: { path: /actuator/health/readiness, port: 8080 } }
resources: { requests: { cpu: 250m, memory: 512Mi }, limits: { memory: 768Mi } }
```

## Common pitfalls
- Stateful containers losing data on restart.
- No resource limits causing noisy neighbours.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[05 - Statelessness]]
- [[06 - Externalized Configuration]]

---
Previous: [[13 - Automation CICD and Infrastructure as Code]] | Next: [[15 - Versioning and Backward Compatibility]]
