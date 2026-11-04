---
tags: [microservices, springboot, java, interview, observability]
category: Observability and Monitoring
questions: 83-87
status: to-revise
---

# Observability and Monitoring (Q83 to Q87)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q83. What are the three pillars of observability? 🟢
> Observability = being able to understand what happens inside a system from outside data.
> 1. **Logs:** detailed text events ("payment failed for order 5").
> 2. **Metrics:** numbers over time (requests per second, error rate, CPU).
> 3. **Traces:** the path of one request across many services.
>
> Logs tell *what happened*, metrics tell *how much / how often*, traces tell *where the time went*.

> [!question]- Q84. What is distributed tracing and how do you add it in Spring Boot? 🟡
> When one user request passes through many services, a **trace id** is created and passed along (in headers). Each step is a **span** with its time. You can see which service was slow.
> Spring Boot 3 uses **Micrometer Tracing** (Spring Cloud Sleuth is retired) with OpenTelemetry. Export to Zipkin, Jaeger or Grafana Tempo. Trace ids propagate automatically with RestClient, WebClient, Feign and Kafka.
>
> 📖 **Simple words:**
> - **Trace** = the full journey of one request.
> - **Span** = one step in that journey.

> [!question]- Q85. How do you collect metrics with Micrometer, Prometheus and Grafana? 🟡
> **Micrometer** is the metrics facade in Spring Boot (like SLF4J for logs). Add `micrometer-registry-prometheus`, expose `/actuator/prometheus`, let **Prometheus** scrape it, and draw charts in **Grafana**. You can add custom business metrics too.
>
> 📖 **Simple words:**
> - **Scrape** = Prometheus periodically pulls metrics from your app.
>
> ```java
> @Component
> class OrderMetrics {
>     private final Counter placed;
>     OrderMetrics(MeterRegistry registry) {
>         this.placed = Counter.builder("orders.placed").register(registry);
>     }
>     void orderPlaced() { placed.increment(); }
> }
> ```

> [!question]- Q86. How should logging work in microservices? 🟡
> - Write logs to **console** in **JSON** (structured) format.
> - Ship them to a central place (ELK / Elastic, Loki, Splunk) so you can search all services at once.
> - Include `traceId` in every log line (a **correlation id**) to follow one request across services.
> - Use correct levels (ERROR, WARN, INFO, DEBUG) and **never log passwords, tokens or personal data**.
>
> 📖 **Simple words:**
> - **Structured logging** = logs as key-value data (JSON) that tools can search easily.
> - **Correlation id** = an id that links all log lines of one request.
>
> ```yaml
> logging:
>   pattern:
>     level: "%5p [${spring.application.name},%X{traceId},%X{spanId}]"
> ```

> [!question]- Q87. What should you monitor and alert on? 🟡
> Use the **RED** method for services:
> - **R**ate: requests per second
> - **E**rrors: error percentage
> - **D**uration: latency (use p95/p99, not only average)
>
> Also watch **saturation** (CPU, memory, DB connections, thread pool, Kafka consumer lag). Alert on **user-visible symptoms** (high errors, slow responses) and define **SLOs** (for example 99.9% of requests succeed).
>
> 📖 **Simple words:**
> - **p99 latency** = 99% of requests are faster than this value.
> - **SLO (Service Level Objective)** = the reliability target you promise.
> - **Consumer lag** = how far behind a Kafka consumer is.

---
Previous: [[10 - Testing]] | Next: [[12 - Docker Kubernetes and DevOps]]
