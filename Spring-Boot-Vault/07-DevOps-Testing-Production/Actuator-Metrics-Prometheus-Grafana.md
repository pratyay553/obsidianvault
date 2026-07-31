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

# 📌 Actuator, Metrics, Prometheus & Grafana

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Boot Actuator exposes production-ready HTTP endpoints (health, metrics, info) that Prometheus scrapes and stores as time-series data, which Grafana then visualizes as dashboards — giving you real-time observability into JVM health, request rates, error rates, and custom business metrics.

## 🧱 Key Concepts & Mechanics

### Actuator Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/actuator/health` | App health (UP/DOWN) + component detail |
| `/actuator/metrics` | List all available metrics |
| `/actuator/metrics/{name}` | Query a specific metric |
| `/actuator/prometheus` | Metrics in Prometheus text format |
| `/actuator/info` | App info (version, git commit) |
| `/actuator/env` | Environment properties (⚠️ sensitive) |
| `/actuator/loggers` | View and change log levels at runtime |
| `/actuator/threaddump` | JVM thread dump |
| `/actuator/heapdump` | JVM heap dump |
| `/actuator/httptrace` | Recent HTTP exchanges |

### Micrometer
- Spring Boot's metrics facade — implementation-agnostic
- Pluggable backends: Prometheus, Datadog, CloudWatch, InfluxDB, Dynatrace
- Core metric types: `Counter`, `Gauge`, `Timer`, `DistributionSummary`, `LongTaskTimer`

### Health Indicators
- Auto-configured: DB, Redis, RabbitMQ, Kafka, Disk space
- Custom: implement `HealthIndicator` or `ReactiveHealthIndicator`
- Aggregate status: `UP` if all indicators are UP; `DOWN` otherwise

### Key Metrics to Monitor

| Metric | Meaning |
|--------|---------|
| `http.server.requests` | Request rate, latency, error rate by endpoint |
| `jvm.memory.used` | Heap and non-heap usage |
| `jvm.gc.pause` | GC pause durations |
| `hikaricp.connections.active` | Active DB connection pool usage |
| `process.cpu.usage` | JVM CPU usage |
| `spring.cache.gets` | Cache hit/miss rates |

## 💻 Production Code Example

```yaml
# application.yml — Actuator config
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics, loggers
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized   # hide details from anonymous users
      show-components: when-authorized
      probes:
        enabled: true                 # /actuator/health/liveness + /readiness
  metrics:
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
    distribution:
      percentiles-histogram:
        http.server.requests: true    # enables P50/P90/P99 histograms
      percentiles:
        http.server.requests: 0.5, 0.9, 0.95, 0.99
  info:
    git:
      mode: full                      # requires spring-boot-starter-actuator + git plugin

# Prometheus scrape config (prometheus.yml)
# scrape_configs:
#   - job_name: 'spring-app'
#     metrics_path: '/actuator/prometheus'
#     scrape_interval: 15s
#     static_configs:
#       - targets: ['app:8080']
```

```java
// ── Secure actuator endpoints ─────────────────────────────────────────
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/actuator/health/**").permitAll()
            .requestMatchers("/actuator/**").hasRole("ACTUATOR")
            .anyRequest().authenticated()
        );
        return http.build();
    }
}

// ── Custom health indicator ───────────────────────────────────────────
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    private final ExternalApiClient client;

    public ExternalApiHealthIndicator(ExternalApiClient client) {
        this.client = client;
    }

    @Override
    public Health health() {
        try {
            PingResult result = client.ping();
            return Health.up()
                .withDetail("responseTimeMs", result.latencyMs())
                .withDetail("version", result.apiVersion())
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}

// ── Custom business metrics ───────────────────────────────────────────
@Service
@RequiredArgsConstructor
public class OrderMetricsService {

    private final MeterRegistry registry;
    private final Counter ordersCreated;
    private final Counter ordersFailed;
    private final Timer orderProcessingTimer;

    public OrderMetricsService(MeterRegistry registry) {
        this.registry = registry;
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total orders successfully created")
            .tag("environment", "production")
            .register(registry);
        this.ordersFailed = Counter.builder("orders.failed")
            .description("Total orders that failed")
            .register(registry);
        this.orderProcessingTimer = Timer.builder("orders.processing.duration")
            .description("Order processing time")
            .publishPercentiles(0.5, 0.9, 0.99)
            .register(registry);
    }

    public OrderDto createOrder(CreateOrderRequest req) {
        return orderProcessingTimer.record(() -> {
            try {
                OrderDto order = doCreateOrder(req);
                ordersCreated.increment();
                return order;
            } catch (Exception e) {
                ordersFailed.increment(Tags.of("reason", e.getClass().getSimpleName()));
                throw e;
            }
        });
    }

    // Gauge — current value (not cumulative)
    @PostConstruct
    public void registerGauges() {
        Gauge.builder("orders.queue.depth", orderQueue, Queue::size)
            .description("Current order processing queue depth")
            .register(registry);
    }
}

// ── Custom actuator endpoint ──────────────────────────────────────────
@Component
@Endpoint(id = "feature-flags")
public class FeatureFlagsEndpoint {

    private final FeatureFlagService featureFlagService;

    @ReadOperation
    public Map<String, Boolean> flags() {
        return featureFlagService.getAllFlags();
    }

    @WriteOperation
    public void toggleFlag(@Selector String flagName, boolean enabled) {
        featureFlagService.setFlag(flagName, enabled);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Exposing all actuator endpoints publicly — `/actuator/env` and `/actuator/heapdump` leak secrets and memory.
  **Fix:** Expose only `health`, `info`, `prometheus` publicly. Require `ACTUATOR` role for the rest.

- **Mistake:** No health probes configured for Kubernetes — K8s needs separate liveness/readiness endpoints.
  **Fix:** Enable `management.endpoint.health.probes.enabled=true`. Use `/actuator/health/liveness` and `/actuator/health/readiness` in K8s probes.

- **Mistake:** Not tagging metrics with `application` and `environment` — Prometheus shows metrics from all apps merged.
  **Fix:** Set `management.metrics.tags.application=${spring.application.name}` and environment tag.

- **Mistake:** Enabling `percentiles-histogram: true` on high-cardinality metrics — creates millions of time series in Prometheus.
  **Fix:** Apply histogram only to key latency metrics. Avoid high-cardinality tags (user IDs, UUIDs) in metric tag values.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is Micrometer and how does it relate to Spring Boot Actuator?
   - **A:** Micrometer is a vendor-neutral metrics instrumentation library — the "SLF4J for metrics." Spring Boot Actuator auto-configures Micrometer and registers JVM, HTTP, and cache metrics automatically. You choose the backend (Prometheus, Datadog, CloudWatch) by adding the appropriate dependency; your metric code doesn't change.

2. **Q:** What is the difference between liveness and readiness probes?
   - **A:** Liveness checks if the app is alive — if it fails, Kubernetes restarts the pod. Readiness checks if the app is ready to receive traffic — if it fails, K8s removes the pod from the load balancer but doesn't restart it. A Spring Boot app might be alive but not ready (still connecting to DB). Use separate endpoints: `/actuator/health/liveness` and `/actuator/health/readiness`.

3. **Q:** How do you expose a custom business metric in Spring Boot?
   - **A:** Inject `MeterRegistry` and use its fluent builders: `Counter.builder("name").register(registry)`, `Timer.builder("name").register(registry)`, `Gauge.builder("name", obj, fn).register(registry)`. Add tags for dimensions. Metrics are automatically exposed on `/actuator/prometheus`.

4. **Q:** What is a Counter vs Timer vs Gauge?
   - **A:** Counter — monotonically increasing number (orders created, errors). Timer — measures duration + counts calls, with optional percentile histograms (HTTP request duration). Gauge — tracks a current instantaneous value (queue depth, active connections) — can go up and down.

## 🔗 Related Notes
- [[Docker-Kubernetes-Deployment]]
- [[Async-Processing-Async-CompletableFuture]]
- [[Unit-And-Integration-Testing-Testcontainers]]
