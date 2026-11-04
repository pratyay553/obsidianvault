---
tags: [microservices, springboot, java, interview, java]
category: Java Essentials for Microservices
questions: 9-16
status: to-revise
---

# Java Essentials for Microservices (Q9 to Q16)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q9. What is a Java record and when do you use it? 🟢
> A **record** (Java 16+) is a short way to create an immutable data class. Java generates the constructor, getters, `equals`, `hashCode` and `toString` for you. Perfect for request/response objects and events.
> Do not use records as JPA entities.
>
> 📖 **Simple words:**
> - **DTO (Data Transfer Object)** = a simple object that only carries data in and out of an API.
> - **Immutable** = cannot be changed after it is created.
>
> ```java
> public record OrderResponse(Long id, String status, BigDecimal total) {}
> ```

> [!question]- Q10. What is the Stream API? 🟢
> Streams let you process a collection step by step (filter, transform, collect) in clean code. A stream does not change the original list.
> Tip: keep chains short, and never call a remote service inside a stream loop (it becomes many slow calls).
>
> 📖 **Simple words:**
> - **Lambda** = a short unnamed function, like `o -> o.id()`.
>
> ```java
> List<String> statuses = orders.stream()
>     .filter(o -> o.total().compareTo(BigDecimal.valueOf(100)) > 0)
>     .map(OrderResponse::status)
>     .distinct()
>     .toList();
> ```

> [!question]- Q11. Why use Optional? 🟢
> `Optional<T>` is a box that may or may not hold a value. It makes you handle the "not found" case and reduces `NullPointerException`. Spring Data returns it from `findById`.
> Use it as a **return type**. Do not use it for fields or method parameters.
>
> ```java
> Order order = repository.findById(id)
>     .orElseThrow(() -> new OrderNotFoundException(id));
> ```

> [!question]- Q12. How do you call two services in parallel with CompletableFuture? 🟡
> `CompletableFuture` runs work asynchronously and lets you combine results. If you call two services one after another, time = A + B. In parallel, time = the slower one.
> Always set a timeout and use your own thread pool for production.
>
> 📖 **Simple words:**
> - **Asynchronous** = start a task and continue without waiting for it to finish.
>
> ```java
> CompletableFuture<User> user = CompletableFuture.supplyAsync(() -> userClient.get(id));
> CompletableFuture<List<Order>> orders = CompletableFuture.supplyAsync(() -> orderClient.byUser(id));
>
> Profile profile = user.thenCombine(orders, Profile::new)
>                       .orTimeout(2, TimeUnit.SECONDS)
>                       .join();
> ```

> [!question]- Q13. What are virtual threads (Java 21) and why do they matter for microservices? 🔴
> Virtual threads are very light threads managed by the JVM, not the operating system. You can run thousands of them. When one waits for a database or HTTP call, it does not block an expensive OS thread.
> In Spring Boot 3.2+ you enable them with one property. They help **I/O-bound** services. They do not make CPU-heavy work faster.
> Avoid long `synchronized` blocks, because they can block the underlying thread.
>
> 📖 **Simple words:**
> - **Thread** = a worker that runs your code.
> - **I/O-bound** = mostly waiting for network or disk, not calculating.
>
> ```yaml
> spring:
>   threads:
>     virtual:
>       enabled: true
> ```

> [!question]- Q14. What is the difference between checked and unchecked exceptions? 🟡
> - **Checked** (like `IOException`): the compiler forces you to catch or declare them.
> - **Unchecked** (`RuntimeException`, like `IllegalArgumentException`): no forcing.
>
> In Spring, use unchecked custom exceptions for business errors. Important: `@Transactional` rolls back on unchecked exceptions by default, but **not** on checked ones unless you ask for it.
>
> ```java
> @Transactional(rollbackFor = Exception.class)
> public void placeOrder(Order order) throws IOException { ... }
> ```

> [!question]- Q15. Why must you override equals() and hashCode() together? 🟡
> `equals()` decides when two objects are "the same". `hashCode()` gives a number that `HashMap` and `HashSet` use to find objects fast. Rule: **equal objects must have equal hash codes**. If you change one, change the other, or collections will behave strangely.
> For JPA entities, base equality on a stable business key (like SKU), not on all fields.
>
> ```java
> @Override public boolean equals(Object o) {
>     if (this == o) return true;
>     if (!(o instanceof Product p)) return false;
>     return Objects.equals(sku, p.sku);
> }
> @Override public int hashCode() { return Objects.hash(sku); }
> ```

> [!question]- Q16. How does Java memory work and how do you tune it in containers? 🔴
> Objects live on the **heap**. The **Garbage Collector (GC)** removes objects nobody uses. Common causes of `OutOfMemoryError`: memory leaks, loading huge result sets, unlimited caches.
> In Docker/Kubernetes, tell the JVM to size the heap from the container limit, and watch heap usage in metrics.
>
> 📖 **Simple words:**
> - **Heap** = the memory area where Java objects live.
> - **Garbage Collector** = the JVM part that frees unused memory automatically.
>
> ```bash
> java -XX:MaxRAMPercentage=75 -jar app.jar
> ```

---
Previous: [[01 - Microservices Basics]] | Next: [[03 - Spring Boot Core]]
