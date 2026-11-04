---
tags: [microservices, springboot, java, interview, resilience]
category: Resilience and Distributed Data
questions: 59-68
status: to-revise
---

# Resilience and Distributed Data (Q59 to Q68)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q59. What is a Circuit Breaker? 🟢
> Like an electric fuse. If a service keeps failing, the circuit breaker **stops calling it** for a while, so your service does not waste threads waiting and fail too.
> States:
> - **Closed:** normal, calls pass through.
> - **Open:** too many failures; calls are rejected immediately (or sent to a fallback).
> - **Half-open:** after a wait, a few test calls are allowed. If they succeed, it closes again.
>
> 📖 **Simple words:**
> - **Fail fast** = return an error quickly instead of waiting for a long timeout.

> [!question]- Q60. How do you use Resilience4j in Spring Boot? 🟡
> Resilience4j is the standard library (Hystrix is retired). Add `spring-cloud-starter-circuitbreaker-resilience4j` (or the `resilience4j-spring-boot3` starter), configure instances in YAML, and use annotations.
>
> ```yaml
> resilience4j:
>   circuitbreaker:
>     instances:
>       payment:
>         sliding-window-size: 10
>         failure-rate-threshold: 50
>         wait-duration-in-open-state: 10s
>   retry:
>     instances:
>       payment:
>         max-attempts: 3
>         wait-duration: 500ms
>         enable-exponential-backoff: true
> ```
> ```java
> @CircuitBreaker(name = "payment", fallbackMethod = "fallback")
> @Retry(name = "payment")
> public PaymentResponse pay(PaymentRequest r) { return paymentClient.pay(r); }
>
> PaymentResponse fallback(PaymentRequest r, Throwable t) { return PaymentResponse.pending(); }
> ```

> [!question]- Q61. How should retries be done safely? 🟡
> Retry only for **temporary** problems (timeouts, 503), never for 400 or business errors.
> - Retry only **idempotent** operations (or use an idempotency key).
> - Use **exponential backoff** (wait 1s, 2s, 4s) plus **jitter** (small random delay).
> - Limit attempts (for example 3).
> Without limits, retries from many clients create a **retry storm** that kills a recovering service.
>
> 📖 **Simple words:**
> - **Backoff** = waiting longer after each failed attempt.
> - **Jitter** = a random extra delay so clients do not retry at the same moment.

> [!question]- Q62. What are timeouts, bulkheads and rate limiters in resilience? 🟡
> - **Timeout:** stop waiting after X seconds. Always set one on every remote call.
> - **Bulkhead:** limit concurrent calls to one dependency, so one slow service cannot use all your threads (like ship compartments).
> - **Rate limiter:** limit how many calls you make or accept per second.
>
> 📖 **Simple words:**
> - **Thread pool** = a fixed group of worker threads.

> [!question]- Q63. What is a fallback and graceful degradation? 🟡
> A **fallback** is Plan B when a call fails: return cached data, a default value, or a "pending" status. **Graceful degradation** means the app keeps working with reduced features instead of crashing. Example: if the recommendation service is down, show popular products.
> Always log and alert on fallbacks, otherwise problems stay hidden.

> [!question]- Q64. What is a cascading failure and how do you prevent it? 🔴
> Service C becomes slow -> B waits for C and fills its threads -> A waits for B -> the whole system is down.
> Prevention: **timeouts**, **circuit breakers**, **bulkheads**, **rate limits**, async messaging where possible, and load tests that include failure scenarios (**chaos testing**).
>
> 📖 **Simple words:**
> - **Chaos testing** = deliberately breaking things in a safe way to see how the system reacts.

> [!question]- Q65. What is idempotency and how do you implement it? 🔴
> An operation is idempotent when repeating it does not change the result. It is vital for payments: a retry must not charge the customer twice.
> Implementation: the client sends a unique **Idempotency-Key**. The server stores key + result; if the same key comes again, it returns the stored result without doing the work again.
>
> ```java
> @PostMapping("/payments")
> ResponseEntity<Payment> pay(@RequestHeader("Idempotency-Key") String key,
>                             @RequestBody PayRequest req) {
>     return ResponseEntity.ok(paymentService.payOnce(key, req)); // same key = same result
> }
> ```

> [!question]- Q66. What is the Saga pattern? 🔴
> A **saga** is a business transaction split into **local transactions** across services. If one step fails, earlier steps are undone with **compensating actions**.
> Example: Create order -> Reserve stock -> Charge payment. If payment fails -> release stock -> cancel order.
> - **Choreography:** services react to each other's events (simple, harder to follow).
> - **Orchestration:** one coordinator tells each service what to do (clearer; tools: Temporal, Camunda).
>
> 📖 **Simple words:**
> - **Compensating action** = a step that cancels the effect of an earlier step (refund, cancel).

> [!question]- Q67. What is the Outbox pattern? 🔴
> Problem: you save to the database **and** publish a Kafka event. If the app crashes between them, data and event disagree (the **dual write** problem).
> Solution: in **one local transaction**, save the data and also an event row in an `outbox` table. A separate process (a poller or **Debezium**) reads the outbox and publishes to Kafka, so no event is lost.
>
> 📖 **Simple words:**
> - **Debezium** = a tool that watches database changes and publishes them as events (CDC).
>
> ```java
> @Transactional
> public void placeOrder(Order order) {
>     orderRepository.save(order);
>     outboxRepository.save(new OutboxEvent("OrderPlaced", order.getId(), toJson(order)));
> }
> ```

> [!question]- Q68. What are CQRS and Event Sourcing? 🔴
> - **CQRS:** use one model to **write** (commands) and another, optimised model to **read** (queries). Gives fast reads and independent scaling.
> - **Event Sourcing:** do not store the current state; store **every change as an event** (`MoneyDeposited`, `MoneyWithdrawn`). Current state = replay of events. Gives a perfect audit trail, but is more complex.
>
> They are often used together but you can use CQRS alone.
>
> 📖 **Simple words:**
> - **Audit trail** = a complete history of who did what and when.

---
Previous: [[07 - Discovery Gateway and Configuration]] | Next: [[09 - Security]]
