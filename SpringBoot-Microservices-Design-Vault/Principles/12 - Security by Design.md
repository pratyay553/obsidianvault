---
tags: [microservices, springboot, principle, security]
principle: 12
status: to-review
created: 2026-11-04
---

# 12. Security by Design

> Zero trust: authenticate and authorise every call, encrypt in transit, keep least privilege, protect secrets.

Back to [[00 - Index (MOC)]]

## Why it matters
Microservices multiply the attack surface; internal network is not a trust boundary.

## How to apply in Spring Boot
- Use OAuth2/OIDC with an identity provider (Keycloak, Auth0); services are resource servers validating JWTs.
- Propagate user identity, use service-to-service credentials (client credentials or mTLS).
- Validate all input; enforce authorisation at the service, not just the gateway.
- Scan dependencies and images (Dependabot, Trivy).

## Example
```java
@Bean
SecurityFilterChain security(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(a -> a
            .requestMatchers("/actuator/health/**").permitAll()
            .requestMatchers(HttpMethod.POST, "/orders").hasAuthority("SCOPE_orders.write")
            .anyRequest().authenticated())
        .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
        .build();
}
```

## Common pitfalls
- Trusting any request from inside the cluster.
- Long-lived tokens and shared secrets.

## Checklist
- [ ] Applied in our current service(s)
- [ ] Covered by automated tests or pipeline checks
- [ ] Documented in an ADR if we deviated

## My notes
-

## Related
- [[08 - Service Discovery and API Gateway]]
- [[06 - Externalized Configuration]]

---
Previous: [[11 - Observability]] | Next: [[13 - Automation CICD and Infrastructure as Code]]
