# 75 - Tomcat Thread Pool

Tags: #frameworks #section13
Related: [[74 - Spring Boot Threading Model]] | [[44 - Thread Pool]] | [[72 - Performance Tuning]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand how Tomcat's thread pool works, how to tune it, and what happens when it fills up.

---

## Beginner Explanation
Every HTTP request in Spring Boot (with embedded Tomcat) is handled by a thread from Tomcat's thread pool. The pool has a maximum size. When all threads are busy, new requests queue up. When the queue is full too, requests are rejected.

---

## Analogy
A call centre: fixed number of agents (pool threads). If all are busy, callers wait on hold (queue). If hold queue is full, the call is dropped (request rejected / connection refused).

---

## Technical Deep Dive

### Default Tomcat thread pool settings
| Property | Default | Meaning |
|---|---|---|
| `max-threads` | 200 | Max simultaneous request-handling threads |
| `min-spare-threads` | 10 | Minimum idle threads always ready |
| `accept-count` | 100 | Queue depth when all threads are busy |
| `connection-timeout` | 20000ms | Time to wait for a request after connection |

### Lifecycle of a request
```
Client connects → Tomcat NIO acceptor (single thread) accepts → 
queued → worker thread from pool picks up → 
controller → service → DB → response written → thread returned to pool
```

### The bottleneck
If your service layer blocks for 500ms (DB query), a 200-thread pool handles at most:
```
200 threads / 0.5s per request = 400 requests/second max
```
Beyond that, requests queue. If queue fills → `Connection refused`.

### Tuning in Spring Boot
```yaml
server:
  tomcat:
    threads:
      max: 400          # increase for I/O-heavy workloads
      min-spare: 20     # keep more threads warm
    accept-count: 200   # queue depth before rejecting
    connection-timeout: 10000
```

Or with virtual threads (Java 21 + Spring Boot 3.2+):
```yaml
spring:
  threads:
    virtual:
      enabled: true     # Tomcat uses one virtual thread per request — no limit needed
```

### Monitoring Tomcat threads
```java
// Spring Boot Actuator — expose metrics
// GET /actuator/metrics/tomcat.threads.busy
// GET /actuator/metrics/tomcat.threads.current
// GET /actuator/metrics/tomcat.threads.config.max
```

Or via JMX: `Catalina:type=ThreadPool,name="http-nio-8080"`

### Common symptoms of pool exhaustion
- Response time spikes suddenly (queue builds)
- HTTP 503 or connection timeouts
- Thread dump: 200 threads all BLOCKED on DB/HTTP call
- Actuator: `tomcat.threads.busy == tomcat.threads.config.max`

---

## Code

```java
// Custom Tomcat thread pool configuration
@Configuration
public class TomcatConfig {

    @Bean
    public TomcatServletWebServerFactory tomcatFactory() {
        return new TomcatServletWebServerFactory() {
            @Override
            protected void customizeConnector(Connector connector) {
                super.customizeConnector(connector);
                ProtocolHandler handler = connector.getProtocolHandler();
                if (handler instanceof AbstractProtocol<?> protocol) {
                    protocol.setMaxThreads(400);
                    protocol.setMinSpareThreads(20);
                    protocol.setConnectionTimeout(10_000);
                }
            }
        };
    }
}
```

---

## Interview Questions

**Q: What happens when Tomcat's thread pool is exhausted?**
> New requests queue in an accept queue (default depth 100). Once the queue fills, Tomcat refuses connections, resulting in HTTP 503 or TCP connection refused errors.

**Q: How do virtual threads change Tomcat's threading model?**
> With `spring.threads.virtual.enabled=true` (Spring Boot 3.2+, Java 21), Tomcat creates one virtual thread per request instead of borrowing from a fixed pool. Virtual threads are cheap, so the effective "pool size" becomes practically unlimited — no tuning needed for I/O-bound workloads.

**Q: How do you monitor Tomcat thread pool saturation?**
> Spring Boot Actuator: `/actuator/metrics/tomcat.threads.busy`. Alert when `busy ≥ max × 0.8` (80% saturation).

---

## Cheat Sheet
| Property | Default | Tuning guidance |
|---|---|---|
| `max-threads` | 200 | Increase for I/O-bound; keep ≤ cores for CPU-bound |
| `min-spare-threads` | 10 | Set to expected idle load |
| `accept-count` | 100 | Queue before rejection |
| Virtual threads | Disabled | Enable for Java 21+ I/O workloads |
| Saturation metric | `tomcat.threads.busy` | Alert at 80% |
