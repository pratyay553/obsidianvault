---
tags: [monitoring, actuator, logging, interview]
---

# Monitoring — Actuator & Logging

Related: [[04 - AOP - Aspect Oriented Programming]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
For a scheduler with **no UI**, monitoring is your only window into whether jobs are actually running and healthy. Two tools cover this: **Spring Boot Actuator** (operational endpoints) and **SLF4J + Logback** (application logs).

## 🔑 Actuator

| Endpoint | Shows |
|---|---|
| `/actuator/health` | Is the app up? Are dependencies (DB, disk, custom health indicators) OK? |
| `/actuator/metrics` | JVM/app metrics — memory, thread count, custom counters |
| `/actuator/scheduledtasks` | **Extremely relevant here** — lists every registered `@Scheduled` task, its trigger, and next execution time |
| `/actuator/info` | Build/version metadata |

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,scheduledtasks
```

### Custom Health Indicator (useful for a scheduler)
```java
@Component
public class LastRunHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        if (lastSuccessfulRun.isBefore(Instant.now().minus(Duration.ofHours(2)))) {
            return Health.down().withDetail("reason", "No successful run in 2 hours").build();
        }
        return Health.up().build();
    }
}
```
This pattern — a scheduled job hasn't run recently → mark unhealthy — is exactly how you'd catch a silently-stuck scheduler in production, and is a great thing to mention in an interview.

## 🔑 Logging: SLF4J + Logback

| Component | Role |
|---|---|
| **SLF4J** | A *facade* — your code logs against this interface, not a specific implementation |
| **Logback** | The actual logging *implementation* Spring Boot uses by default |
| `Logger` | The object you call `.info()`/`.warn()`/`.error()` on |
| `logback-spring.xml` | Configures appenders (console/file), patterns, log levels per package |

```java
private static final Logger log = LoggerFactory.getLogger(BirthdayEmailScheduler.class);
log.info("Sending {} birthday emails", count);
```

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/scheduler.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/scheduler.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder><pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern></encoder>
    </appender>
    <root level="INFO"><appender-ref ref="FILE" /></root>
</configuration>
```

## ❓ Interview Q&A

**Q: Why use SLF4J instead of calling Logback directly?**
A: SLF4J decouples your code from any specific logging implementation — you could swap Logback for Log4j2 without touching application code, since you only ever call the SLF4J API.

**Q: How would you detect that a scheduled job silently stopped firing?**
A: Expose `/actuator/scheduledtasks` (and/or `/actuator/health` with a custom `HealthIndicator` tracking last-successful-run timestamps), and wire external monitoring/alerting (e.g., Prometheus scraping `/actuator/prometheus`, or a "dead man's switch" alert) to page on-call if no successful run is seen within the expected interval.

**Q: What's the difference between `%thread` and having virtual threads (per [[02 - Virtual Threads and SimpleAsyncTaskScheduler]])?**
A: With traditional pooled threads, `%thread` names are stable/reusable (`pool-1-thread-3`). With one virtual thread per task, thread "names" are often generic/anonymous by default — you typically add a request/job ID via MDC (Mapped Diagnostic Context) instead of relying on the thread name to correlate logs.

**Q: How would you correlate all log lines from a single scheduled run?**
A: Use SLF4J's **MDC** to inject a unique run/job ID at the start of the scheduled method (`MDC.put("jobId", UUID.randomUUID().toString())`), include `%X{jobId}` in the Logback pattern, and clear the MDC in a `finally` block.
