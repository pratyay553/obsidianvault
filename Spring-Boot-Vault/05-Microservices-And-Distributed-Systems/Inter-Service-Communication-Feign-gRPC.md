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

# 📌 Inter-Service Communication — Feign & gRPC

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Feign is a declarative HTTP client that generates REST call implementations from annotated interfaces (ideal for public/external-facing APIs), while gRPC uses Protocol Buffers and HTTP/2 for strongly-typed, high-performance binary communication between internal services.

## 🧱 Key Concepts & Mechanics

### Feign vs gRPC vs RestTemplate

| | Feign | gRPC | RestTemplate/WebClient |
|-|-------|------|----------------------|
| Protocol | HTTP/1.1 | HTTP/2 (binary) | HTTP/1.1 |
| Contract | Java interface | `.proto` schema | None (manual) |
| Type safety | ✅ Java types | ✅ Proto types | ❌ Manual |
| Load balancing | ✅ (with Eureka) | ✅ (client-side) | ✅ (with @LoadBalanced) |
| Streaming | ❌ | ✅ Bidirectional | ✅ (WebClient) |
| Overhead | Medium | Very low | Medium |
| Best for | Service-to-service REST | High-throughput internal | Flexibility |

### OpenFeign Key Features
- `@FeignClient(name = "service-name")` — registers the interface as a Spring bean
- Integrates with Eureka for load balancing
- Supports `@CircuitBreaker`, `@Retry` via Resilience4j
- Custom `RequestInterceptor` — forward auth headers, add API keys

### gRPC Key Concepts
- **Protobuf** (`.proto` file) — language-agnostic schema + code generator
- **Stub types**: `BlockingStub` (sync), `Stub` (async callback), `FutureStub` (Guava future)
- **Service types**: Unary, Server-streaming, Client-streaming, Bidirectional
- Runs on HTTP/2 — multiplexed streams, header compression, binary framing

## 💻 Production Code Example

```java
// ══════════════════════════════════════════════════════════
// FEIGN
// ══════════════════════════════════════════════════════════

// ── Enable Feign ──────────────────────────────────────────
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication { ... }

// ── Feign client interface ────────────────────────────────
@FeignClient(
    name = "payment-service",          // Eureka service name
    fallback = PaymentClientFallback.class,
    configuration = FeignConfig.class
)
public interface PaymentFeignClient {

    @PostMapping("/api/v1/payments")
    PaymentResult charge(@RequestBody ChargeRequest req);

    @GetMapping("/api/v1/payments/{id}")
    PaymentResult getById(@PathVariable("id") UUID id);

    @GetMapping("/api/v1/payments")
    Page<PaymentSummary> list(@RequestParam int page, @RequestParam int size);
}

// ── Fallback implementation ────────────────────────────────
@Component
public class PaymentClientFallback implements PaymentFeignClient {
    @Override
    public PaymentResult charge(ChargeRequest req) {
        return PaymentResult.queued(req.orderId());
    }
    @Override
    public PaymentResult getById(UUID id) { throw new ServiceUnavailableException(); }
    @Override
    public Page<PaymentSummary> list(int page, int size) { return Page.empty(); }
}

// ── Request interceptor — forward auth token ──────────────
@Configuration
public class FeignConfig {

    @Bean
    public RequestInterceptor authInterceptor() {
        return requestTemplate -> {
            // Forward the JWT from the incoming request to downstream service
            ServletRequestAttributes attrs =
                (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            if (attrs != null) {
                String auth = attrs.getRequest().getHeader("Authorization");
                if (auth != null) requestTemplate.header("Authorization", auth);
            }
        };
    }

    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.BASIC;   // NONE, BASIC, HEADERS, FULL
    }
}

// ══════════════════════════════════════════════════════════
// gRPC
// ══════════════════════════════════════════════════════════

// ── Proto definition (src/main/proto/inventory.proto) ─────
// syntax = "proto3";
// package com.example.inventory;
// option java_multiple_files = true;
//
// service InventoryService {
//   rpc CheckStock (StockRequest) returns (StockResponse);
//   rpc WatchStock (StockRequest) returns (stream StockUpdate);
// }
// message StockRequest { string product_id = 1; int32 quantity = 2; }
// message StockResponse { bool available = 1; int32 current_stock = 2; }
// message StockUpdate { string product_id = 1; int32 delta = 2; }

// ── gRPC server implementation ────────────────────────────
@GrpcService
@RequiredArgsConstructor
public class InventoryGrpcService
        extends InventoryServiceGrpc.InventoryServiceImplBase {

    private final InventoryRepository repo;

    @Override
    public void checkStock(StockRequest req,
                            StreamObserver<StockResponse> observer) {
        try {
            int stock = repo.getStock(req.getProductId());
            observer.onNext(StockResponse.newBuilder()
                .setAvailable(stock >= req.getQuantity())
                .setCurrentStock(stock)
                .build());
            observer.onCompleted();
        } catch (Exception e) {
            observer.onError(Status.INTERNAL
                .withDescription(e.getMessage())
                .asRuntimeException());
        }
    }
}

// ── gRPC client ──────────────────────────────────────────
@Service
public class InventoryGrpcClient {

    private final InventoryServiceGrpc.InventoryServiceBlockingStub stub;

    public InventoryGrpcClient(@GrpcClient("inventory-service")
                                Channel channel) {
        this.stub = InventoryServiceGrpc.newBlockingStub(channel)
            .withDeadlineAfter(3, TimeUnit.SECONDS);
    }

    public boolean isAvailable(String productId, int quantity) {
        StockResponse resp = stub.checkStock(
            StockRequest.newBuilder()
                .setProductId(productId)
                .setQuantity(quantity)
                .build()
        );
        return resp.getAvailable();
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Feign client method parameter names not matching the service's expected parameter names — Feign needs explicit `@PathVariable("name")` and `@RequestParam("name")`.
  **Fix:** Always provide the name value in annotations: `@PathVariable("id")` not just `@PathVariable`.

- **Mistake:** Using `@FeignClient` without circuit breaker or retry — a slow downstream service blocks Feign threads.
  **Fix:** Configure `feign.circuitbreaker.enabled=true` and a fallback class, plus Resilience4j retry.

- **Mistake:** gRPC blocking stub in a reactive (WebFlux) service — blocks the event loop.
  **Fix:** Use `FutureStub` + `Mono.fromFuture()` or the Reactor gRPC library for fully reactive gRPC calls.

- **Mistake:** Not setting deadlines on gRPC stubs — calls block indefinitely if the server hangs.
  **Fix:** Always chain `.withDeadlineAfter(N, TimeUnit.SECONDS)` on the stub.

## ❓ Frequently Asked Interview Questions

1. **Q:** When would you choose Feign over WebClient?
   - **A:** Feign is simpler and more declarative — define an interface and Spring generates the implementation. It integrates naturally with Eureka, Resilience4j, and Spring Security. WebClient is better when you need reactive/non-blocking calls, streaming, or very fine-grained control over the HTTP interaction.

2. **Q:** What are the advantages of gRPC over REST for internal microservice communication?
   - **A:** gRPC uses HTTP/2 (multiplexed, header-compressed, binary), Protocol Buffers (compact binary serialization 3-10x smaller than JSON), strong typing from `.proto` contracts, and supports bidirectional streaming. For high-frequency internal service calls, this adds up to significantly lower latency and CPU usage.

3. **Q:** How does Feign integrate with Eureka?
   - **A:** The `name` in `@FeignClient(name = "payment-service")` becomes a logical service name resolved via Eureka. Spring Cloud OpenFeign uses `Spring Cloud LoadBalancer` to pick a live instance from the registry, so Feign calls are automatically load-balanced.

4. **Q:** What is a Feign `RequestInterceptor` used for?
   - **A:** To mutate every outgoing Feign request — typically to forward the caller's `Authorization` header to downstream services, add API keys, set custom correlation ID headers, or inject tracing headers.

## 🔗 Related Notes
- [[Service-Discovery-Eureka]]
- [[Resiliency-Circuit-Breaker-Resilience4j]]
- [[API-Gateway-Spring-Cloud-Gateway]]
