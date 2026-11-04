---
tags: [microservices, springboot, java, interview, communication]
category: Service Communication and Messaging
questions: 42-50
status: to-revise
---

# Service Communication and Messaging (Q42 to Q50)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q42. What is the difference between synchronous and asynchronous communication? 🟢
> - **Synchronous (REST, gRPC):** caller sends a request and **waits** for the answer. Simple, but if the other service is slow or down, you are affected.
> - **Asynchronous (Kafka, RabbitMQ):** caller sends a message and **continues**. The receiver processes it later. Services are less tied together.
>
> Use sync when you need an immediate answer (check price). Use async for work that can happen later (send email, update stock).
>
> 📖 **Simple words:**
> - **Message broker** = software that safely holds messages between services (Kafka, RabbitMQ).

> [!question]- Q43. RestTemplate, RestClient, WebClient and OpenFeign: which one should I use? 🟡
> | Client | Notes |
> |---|---|
> | `RestTemplate` | Old. In maintenance mode. Avoid in new code. |
> | `RestClient` | Modern, **blocking**, fluent API (Spring 6.1+). Good default. |
> | `WebClient` | **Non-blocking / reactive**. Use with WebFlux or streaming. |
> | OpenFeign | **Declarative**: write an interface, Spring writes the client. Very popular between services. |
>
> 📖 **Simple words:**
> - **Blocking** = the thread waits until the response arrives.
> - **Reactive** = a style where a few threads handle many requests without waiting.

> [!question]- Q44. How do you call another service using OpenFeign? 🟢
> Add `spring-cloud-starter-openfeign`, enable with `@EnableFeignClients`, and declare an interface. **Always set timeouts**, otherwise a slow service can hang your threads.
>
> ```java
> @FeignClient(name = "payment-service", url = "${payment.url}")
> public interface PaymentClient {
>     @PostMapping("/payments")
>     PaymentResponse pay(@RequestBody PaymentRequest request);
> }
> ```
> ```properties
> spring.cloud.openfeign.client.config.default.connect-timeout=2000
> spring.cloud.openfeign.client.config.default.read-timeout=3000
> ```

> [!question]- Q45. What is gRPC and when is it better than REST? 🔴
> gRPC is a fast call protocol from Google. It uses **Protocol Buffers** (compact binary format) over HTTP/2 and creates client/server code from a `.proto` file.
> - **Better for:** internal service-to-service calls, high speed, streaming, strict contracts.
> - **REST is better for:** public APIs, browsers, easy debugging, wide tool support.
>
> 📖 **Simple words:**
> - **Protocol Buffers (protobuf)** = a compact way to describe and send structured data.

> [!question]- Q46. What is Apache Kafka? Explain topic, partition, offset and consumer group. 🟡
> Kafka is a distributed **event log**. Producers write events, consumers read them. Events are kept for a set time, so many services can read them independently.
> - **Topic:** a named stream, like `orders.placed`.
> - **Partition:** a topic is split into parts for speed. Order is guaranteed **inside one partition** (same key = same partition).
> - **Offset:** the position number of a message in a partition. Consumers remember it.
> - **Consumer group:** services in one group share the work; each message goes to one member of the group.
>
> 📖 **Simple words:**
> - **Producer** = sends messages. **Consumer** = reads messages.

> [!question]- Q47. Kafka vs RabbitMQ: what is the difference? 🟡
> | | Kafka | RabbitMQ |
> |---|---|---|
> | Model | Event log (messages stay, can be replayed) | Queue (message removed after it is consumed) |
> | Strength | Huge throughput, streaming, replay | Flexible routing, task queues |
> | Use case | Event-driven systems, analytics | Background jobs, request/reply |
>
> Neither is "better"; choose by need.

> [!question]- Q48. How do you produce and consume Kafka messages in Spring Boot? 🟡
> Add `spring-kafka`. Use `KafkaTemplate` to send and `@KafkaListener` to receive. Use a meaningful **key** (like orderId) so related events stay in order.
>
> ```java
> // Producer
> kafkaTemplate.send("orders.placed", order.id().toString(), event);
>
> // Consumer
> @KafkaListener(topics = "orders.placed", groupId = "inventory-service")
> public void handle(OrderPlaced event) {
>     inventoryService.reserve(event);
> }
> ```
> ```yaml
> spring:
>   kafka:
>     bootstrap-servers: localhost:9092
>     producer:
>       acks: all
>     consumer:
>       group-id: inventory-service
>       auto-offset-reset: earliest
> ```

> [!question]- Q49. What are message delivery guarantees? How do you handle duplicates? 🔴
> - **At-most-once:** may lose messages, never duplicates.
> - **At-least-once:** never loses, but may deliver **twice**. (Most common.)
> - **Exactly-once:** hard and costly; usually limited to Kafka transactions.
>
> Practical solution: use at-least-once and make the consumer **idempotent**. For example, store the processed `eventId` and skip it if it appears again.
>
> 📖 **Simple words:**
> - **Idempotent consumer** = processing the same message twice has no extra effect.

> [!question]- Q50. What is a Dead Letter Queue (DLQ) and how do you handle failing messages? 🔴
> If a message keeps failing (bad data, bug), you must not block the whole topic. Retry a few times, then move the message to a separate **dead letter topic/queue** for investigation. Add an alert when the DLQ has messages.
>
> 📖 **Simple words:**
> - **Poison message** = a message that always fails processing.
>
> ```java
> @Bean
> DefaultErrorHandler errorHandler(KafkaTemplate<Object, Object> template) {
>     var recoverer = new DeadLetterPublishingRecoverer(template);  // sends to <topic>.DLT
>     return new DefaultErrorHandler(recoverer, new FixedBackOff(1000L, 3)); // 3 retries
> }
> ```

---
Previous: [[05 - Data and Transactions]] | Next: [[07 - Discovery Gateway and Configuration]]
