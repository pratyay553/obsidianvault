# 80 - Reactive vs Multithreading

Tags: #frameworks #section13
Related: [[78 - Virtual Threads (Project Loom)]] | [[49 - CompletableFuture]] | [[77 - Async in Spring]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand the difference between reactive programming and traditional multithreading — when to use each, and how virtual threads change the trade-offs.

---

## Beginner Explanation
Traditional multithreading uses **one thread per task** — simple code, but threads are expensive. Reactive programming uses **callbacks and event loops** — complex code, but very few threads handle thousands of tasks. Virtual threads (Java 21) offer a third path: one thread per task, but threads are cheap.

---

## Analogy
- **Traditional (one thread per request)**: One waiter per table — simple for the waiter, but expensive if you have 10,000 tables.
- **Reactive (event loop)**: One super-efficient waiter manages all tables by never standing idle — complex choreography, but scales to 10,000 tables.
- **Virtual threads**: One waiter per table, but waiters are practically free — simple code, massive scale.

---

## Technical Deep Dive

### Traditional blocking model (one thread per request)

```java
// Blocking, simple code
String result = httpClient.get(url); // thread blocks here waiting for response
process(result);
```

**Pros**: Simple, readable, debuggable, familiar  
**Cons**: Each blocked thread consumes ~1MB stack; 10k concurrent requests = 10GB just for stacks

### Reactive model (Project Reactor / RxJava / WebFlux)

```java
// Non-blocking, callback-based
Mono<String> result = webClient.get().uri(url).retrieve().bodyToMono(String.class)
    .map(this::process)
    .flatMap(this::persist)
    .onErrorReturn("default");
```

**Pros**: Tiny thread count; extreme throughput for I/O-bound workloads  
**Cons**: Complex code; hard to debug; callback hell; can't use `ThreadLocal`; stacktraces are useless; requires reactive drivers for everything (DB, HTTP, etc.)

### Reactive threading model (WebFlux / Netty)
```
N event-loop threads (N = CPU cores)
Each thread runs an event loop — never blocks
When I/O completes → callback is scheduled on an event loop thread
```

### Virtual threads model (Java 21+)

```java
// Looks like blocking, but isn't (for the OS)
String result = httpClient.get(url); // virtual thread parks, carrier thread free
process(result);
```

**Pros**: Simple blocking code; scales like reactive; readable stacktraces; ThreadLocal works (but prefer ScopedValues)  
**Cons**: Must avoid pinning (`synchronized` + blocking); doesn't help CPU-bound work; ecosystem still catching up

### Feature comparison

| Aspect | Blocking Threads | Reactive | Virtual Threads |
|---|---|---|---|
| Code style | Imperative (simple) | Declarative/callbacks (complex) | Imperative (simple) |
| Thread count | High (1 per task) | Very low (N = cores) | Medium (N carriers) |
| Stack memory | High | Low | Very low (~1KB per VT) |
| Debugging | Easy | Hard | Easy |
| Throughput | Limited by pool | Excellent | Excellent |
| CPU-bound | Good | No benefit | No benefit |
| Spring support | MVC (servlet stack) | WebFlux | MVC + virtual threads |
| Java version | All | All | 21+ |

### When to use what

| Use case | Recommendation |
|---|---|
| New Java 21+ I/O-bound service | Virtual threads (simplest) |
| Extreme throughput, existing reactive team | Reactive (WebFlux) |
| CPU-bound computation | Platform threads + ForkJoinPool |
| Legacy code migration | Virtual threads (drop-in) |
| Mixed CPU + I/O | Virtual threads or separate pools |

---

## Code

```java
// Spring MVC with virtual threads (Java 21 + Spring Boot 3.2)
// application.yml: spring.threads.virtual.enabled=true

@RestController
public class OrderController {
    // This blocks, but runs on a virtual thread — scales like reactive
    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        User user = userService.fetchUser(id);    // blocks virtual thread, not OS thread
        Order order = orderService.fetch(id);     // same
        return enrich(user, order);               // CPU work
    }
}

// Spring WebFlux (reactive)
@RestController
public class ReactiveOrderController {
    // Non-blocking, callback chain — harder to read
    @GetMapping("/orders/{id}")
    public Mono<Order> getOrder(@PathVariable Long id) {
        return userService.fetchUser(id)
            .zipWith(orderService.fetch(id))
            .map(tuple -> enrich(tuple.getT1(), tuple.getT2()));
    }
}
```

---

## Interview Questions

**Q: What is the main trade-off between reactive and virtual thread programming?**
> Reactive: complex non-blocking code with very low thread count. Virtual threads: simple blocking code with the same I/O scalability. Virtual threads win on simplicity; reactive wins on ecosystem maturity and CPU control.

**Q: Can reactive and virtual threads be used together?**
> Yes, but it's unusual. WebFlux can run on virtual threads, but the event-loop model and virtual-thread model overlap in purpose. Typically you choose one model per service.

**Q: Why don't virtual threads help with CPU-bound work?**
> CPU-bound work doesn't block — the virtual thread stays mounted on its carrier. Having more virtual threads than CPU cores doesn't add parallelism; it just adds scheduling overhead.

---

## Cheat Sheet
| Model | Thread cost | Code style | Best for |
|---|---|---|---|
| Blocking | High (~1MB/thread) | Simple | Legacy, small-scale |
| Reactive | Very low | Complex | High-throughput, reactive teams |
| Virtual threads | Very low (~1KB) | Simple | Java 21+ I/O-heavy services |
