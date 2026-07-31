---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Intermediate
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Service Discovery with Eureka

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Eureka is a service registry where microservices self-register their host/port on startup and query it to discover other services by logical name — eliminating hardcoded URLs and enabling client-side load balancing across multiple instances.

## 🧱 Key Concepts & Mechanics

### Architecture
```
OrderService (Eureka Client)
  → registers: ORDER-SERVICE @ host:8081
  → heartbeat every 30s

PaymentService (Eureka Client)
  → registers: PAYMENT-SERVICE @ host:8082
  → discovers: ORDER-SERVICE → resolves to host:8081

Eureka Server (Registry)
  → stores: { ORDER-SERVICE: [host:8081, host:8083], PAYMENT-SERVICE: [host:8082] }
  → evicts stale registrations after 90s (3 missed heartbeats)
```

### Self-Preservation Mode
- Eureka stops evicting registrations if >85% of heartbeats are missing
- Protects against mass deregistration during network partitions
- Disable in dev (`eureka.server.enable-self-preservation=false`) to avoid stale entries

### Client-Side Load Balancing (Spring Cloud LoadBalancer)
- Replaces deprecated Ribbon
- `@LoadBalanced RestTemplate` / `@LoadBalanced WebClient` → resolves `http://SERVICE-NAME/path` to actual host
- Algorithms: Round Robin (default), Random, Weighted

### Eureka Server Clustering
- Multiple Eureka servers register with each other as peers
- `eureka.client.serviceUrl.defaultZone` points to peer URLs
- CAP: Eureka favors **availability** over consistency — AP system

## 💻 Production Code Example

```yaml
# Eureka Server — application.yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false    # server doesn't register with itself
    fetchRegistry: false
  server:
    enable-self-preservation: false   # dev only
```

```java
// ── Eureka Server ─────────────────────────────────────────────────────
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) { SpringApplication.run(...); }
}
```

```yaml
# Client service — application.yml
spring:
  application:
    name: order-service   # becomes the service name in the registry

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true          # register with IP, not hostname
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
```

```java
// ── Eureka Client Service ──────────────────────────────────────────────
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication { ... }

// ── Load-balanced RestTemplate ────────────────────────────────────────
@Configuration
public class WebConfig {

    @Bean
    @LoadBalanced   // resolves SERVICE-NAME to actual host via Eureka
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

// ── Service calling another service by logical name ────────────────────
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RestTemplate restTemplate;

    public PaymentResult chargeCustomer(UUID orderId, BigDecimal amount) {
        // "payment-service" is resolved via Eureka — no hardcoded URL
        return restTemplate.postForObject(
            "http://payment-service/api/v1/payments",
            new ChargeRequest(orderId, amount),
            PaymentResult.class
        );
    }
}

// ── Programmatic service discovery ────────────────────────────────────
@Service
@RequiredArgsConstructor
public class DiscoveryService {

    private final DiscoveryClient discoveryClient;

    public List<String> getInstanceUrls(String serviceName) {
        return discoveryClient.getInstances(serviceName).stream()
            .map(si -> si.getUri().toString())
            .toList();
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Not setting `spring.application.name` — service registers as `UNKNOWN`.
  **Fix:** Always set a meaningful name; it becomes the service identifier in the registry.

- **Mistake:** Forgetting `@LoadBalanced` on `RestTemplate` — service name is treated as a literal hostname.
  **Fix:** Annotate the `RestTemplate` or `WebClient` bean with `@LoadBalanced`.

- **Mistake:** Using Eureka without Resilience4j circuit breakers — one failing downstream service can cascade failures.
  **Fix:** Always pair service discovery with circuit breakers. See [[Resiliency-Circuit-Breaker-Resilience4j]].

- **Mistake:** Relying on Eureka in Kubernetes — K8s has its own DNS-based service discovery.
  **Fix:** In K8s, use K8s Service DNS + Kubernetes client-side load balancing. Eureka is for VMs / non-K8s environments.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is service discovery and why is it needed in microservices?
   - **A:** In microservices, instances start and stop dynamically with changing IPs/ports. Hardcoded URLs break on restarts. Service discovery maintains a live registry — services register themselves and query by logical name, getting current instance addresses.

2. **Q:** What is the difference between client-side and server-side load balancing?
   - **A:** Server-side (e.g., Nginx, AWS ALB): a separate component distributes traffic — clients send to one endpoint. Client-side (Spring Cloud LoadBalancer + Eureka): the client fetches the instance list from the registry and picks one itself — more flexible, no single LB bottleneck.

3. **Q:** What happens if the Eureka server goes down?
   - **A:** Clients cache the last-known registry locally. They continue resolving service addresses from the cache for a configurable period. New registrations/deregistrations won't propagate, but existing communication continues.

## 🔗 Related Notes
- [[API-Gateway-Spring-Cloud-Gateway]]
- [[Resiliency-Circuit-Breaker-Resilience4j]]
- [[Inter-Service-Communication-Feign-gRPC]]
