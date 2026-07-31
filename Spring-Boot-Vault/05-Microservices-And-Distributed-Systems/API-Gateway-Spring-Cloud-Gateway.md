---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Advanced
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 API Gateway — Spring Cloud Gateway

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Cloud Gateway is a reactive API gateway that sits in front of all microservices, routing requests by path/host predicates and applying cross-cutting filters (auth, rate limiting, circuit breaking, logging) in one place so individual services don't need to implement them.

## 🧱 Key Concepts & Mechanics

### Core Concepts

| Term | Meaning |
|------|---------|
| **Route** | A mapping: predicate → downstream URI + optional filters |
| **Predicate** | Condition to match (path, method, header, query param, host) |
| **Filter** | Pre/post processing — modify request or response |
| **Global Filter** | Applies to every route (logging, auth, tracing) |
| **GatewayFilter** | Applies to a specific route only |

### Built-In Filters

| Filter | Purpose |
|--------|---------|
| `RewritePath` | Rewrites the downstream URL path |
| `StripPrefix` | Removes path prefix before forwarding |
| `AddRequestHeader` | Injects header into downstream request |
| `RequestRateLimiter` | Token-bucket rate limiting (requires Redis) |
| `CircuitBreaker` | Wraps route in Resilience4j circuit breaker |
| `Retry` | Retries on failure |
| `SetStatus` | Override response status |

### Reactive Architecture
- Built on Spring WebFlux (Netty, not Servlet)
- **Non-blocking** — handles thousands of concurrent connections on a small thread pool
- Do NOT use blocking code (JDBC, RestTemplate) in gateway filters — use `WebClient`

## 💻 Production Code Example

```yaml
# application.yml
spring:
  cloud:
    gateway:
      default-filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 100
            redis-rate-limiter.burstCapacity: 200
            key-resolver: "#{@ipKeyResolver}"
      routes:
        - id: order-service
          uri: lb://order-service           # lb:// = load-balanced via Eureka
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - StripPrefix=0
            - name: CircuitBreaker
              args:
                name: orderCircuitBreaker
                fallbackUri: forward:/fallback/orders

        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/v1/products/**
            - Method=GET                    # only route GET requests
          filters:
            - AddRequestHeader=X-Gateway-Source, api-gateway
            - name: Retry
              args:
                retries: 3
                statuses: SERVICE_UNAVAILABLE

        - id: auth-service
          uri: lb://auth-service
          predicates:
            - Path=/api/v1/auth/**
          # no auth filter on auth routes
```

```java
// ── Global auth filter ────────────────────────────────────────────────
@Component
@Slf4j
public class AuthenticationFilter implements GlobalFilter, Ordered {

    private final JwtUtil jwtUtil;
    private static final List<String> PUBLIC_PATHS =
        List.of("/api/v1/auth/", "/api/v1/products");

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getPath().toString();

        if (PUBLIC_PATHS.stream().anyMatch(path::startsWith)) {
            return chain.filter(exchange);
        }

        String authHeader = exchange.getRequest()
            .getHeaders().getFirst(HttpHeaders.AUTHORIZATION);

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        try {
            String token = authHeader.substring(7);
            Claims claims = jwtUtil.extractClaims(token);
            // Forward user info to downstream services
            ServerHttpRequest mutatedRequest = exchange.getRequest().mutate()
                .header("X-User-Id", claims.getSubject())
                .header("X-User-Roles", claims.get("roles", String.class))
                .build();
            return chain.filter(exchange.mutate().request(mutatedRequest).build());
        } catch (JwtException e) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
    }

    @Override
    public int getOrder() { return -1; }  // run before routing
}

// ── Rate limiting key resolver (by IP) ───────────────────────────────
@Bean
public KeyResolver ipKeyResolver() {
    return exchange -> Mono.just(
        Objects.requireNonNull(
            exchange.getRequest().getRemoteAddress()).getAddress().getHostAddress()
    );
}

// ── Fallback controller ───────────────────────────────────────────────
@RestController
public class FallbackController {

    @GetMapping("/fallback/orders")
    public Mono<Map<String, String>> ordersFallback() {
        return Mono.just(Map.of(
            "error", "Order service is currently unavailable",
            "code", "SERVICE_UNAVAILABLE"
        ));
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using blocking I/O (`RestTemplate`, `JdbcTemplate`) inside gateway filters.
  **Fix:** Use `WebClient` for HTTP calls. Gateway runs on Netty event loop — blocking freezes all requests on that thread.

- **Mistake:** Forwarding user authentication tokens to downstream services and re-validating them there.
  **Fix:** Validate tokens at the gateway and forward claims as trusted headers (`X-User-Id`, `X-User-Roles`). Downstream services trust gateway headers (secured by network policy).

- **Mistake:** Rate limiting without Redis — the default in-memory rate limiter doesn't work across multiple gateway instances.
  **Fix:** Configure `spring-boot-starter-data-redis-reactive` and use `RedisRateLimiter`.

- **Mistake:** Forgetting `lb://` prefix — the route uses the literal string as a URL instead of resolving via Eureka.
  **Fix:** Use `lb://service-name` for load-balanced routing.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is an API Gateway and why use one?
   - **A:** An API gateway is the single entry point for all client traffic. It centralizes cross-cutting concerns — authentication, rate limiting, SSL termination, request routing, response caching, tracing — so individual microservices don't each implement them. It also hides internal service topology from clients.

2. **Q:** What is the difference between a Global Filter and a GatewayFilter?
   - **A:** Global filters apply to every route — use for cross-cutting concerns (auth, logging, tracing). GatewayFilters attach to specific routes — use for route-specific behavior (path rewriting, retry on a particular service).

3. **Q:** How does Spring Cloud Gateway differ from Zuul?
   - **A:** Zuul 1.x is servlet-based and blocking. Spring Cloud Gateway is reactive (WebFlux/Netty), non-blocking, and handles much higher concurrency on fewer threads. Zuul 2 is also async, but Spring Cloud Gateway is the standard in the Spring ecosystem.

4. **Q:** How do you implement rate limiting in Spring Cloud Gateway?
   - **A:** Use the `RequestRateLimiter` filter with `RedisRateLimiter` (token bucket algorithm). Configure `replenishRate` (requests/second sustained), `burstCapacity` (max burst), and a `KeyResolver` (by IP, user ID, or API key). Requires Redis.

## 🔗 Related Notes
- [[Service-Discovery-Eureka]]
- [[Resiliency-Circuit-Breaker-Resilience4j]]
- [[Asymmetric-JWT-RS256-Implementation]]
