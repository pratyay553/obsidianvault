# 58 - Exchanger

Tags: #coordination-utilities #section9
Related: [[43 - BlockingQueue]] | [[57 - Semaphore]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand `Exchanger<V>` — a synchronization point where exactly two threads swap objects with each other.

---

## Beginner Explanation
`Exchanger` is a rendezvous point. Two threads each call `exchange(myData)` and block until the other thread arrives. When both arrive, they swap their data and both continue.

---

## Analogy
Two couriers meeting at a crossroads. Courier A carries package A for Courier B, and Courier B carries package B for Courier A. They meet, swap, and go their separate ways.

---

## Technical Deep Dive

### API
```java
Exchanger<DataBuffer> exchanger = new Exchanger<>();

// Thread 1
DataBuffer result = exchanger.exchange(myBuffer);       // blocks until Thread 2 arrives
DataBuffer result = exchanger.exchange(myBuffer, 5, TimeUnit.SECONDS); // with timeout
```

### How it works
- Thread 1 calls `exchange(item1)` → blocks
- Thread 2 calls `exchange(item2)` → both are released simultaneously
- Thread 1 receives `item2`, Thread 2 receives `item1`
- **Exactly 2 threads** — not 1, not 3

### Classic use case: pipeline / double-buffer
One thread fills a buffer while another thread drains/processes the previous buffer. When both finish, they exchange buffers and repeat.

---

## Code

```java
import java.util.concurrent.*;
import java.util.*;

public class ExchangerDemo {
    static Exchanger<List<Integer>> exchanger = new Exchanger<>();

    public static void main(String[] args) throws InterruptedException {
        // Producer: fills buffer and hands it to consumer
        Thread producer = new Thread(() -> {
            List<Integer> buffer = new ArrayList<>();
            try {
                for (int cycle = 0; cycle < 3; cycle++) {
                    buffer.clear();
                    for (int i = 0; i < 5; i++) buffer.add(cycle * 10 + i);
                    System.out.println("Producer filled: " + buffer);
                    buffer = exchanger.exchange(buffer); // give full, get empty
                }
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        // Consumer: drains buffer and hands empty back to producer
        Thread consumer = new Thread(() -> {
            List<Integer> buffer = new ArrayList<>();
            try {
                for (int cycle = 0; cycle < 3; cycle++) {
                    buffer = exchanger.exchange(buffer); // get full, give empty
                    System.out.println("Consumer processing: " + buffer);
                    Thread.sleep(200); // simulate processing
                }
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        producer.start(); consumer.start();
        producer.join(); consumer.join();
    }
}
```

---

## Interview Questions

**Q: How many threads can use a single `Exchanger` instance?**
> Exactly 2. If a third thread calls `exchange()`, it will wait until one of the current pair vacates, then pair with the next arriving thread.

**Q: What is the primary use case for `Exchanger`?**
> Pipeline processing with double-buffering: a producer fills one buffer while the consumer drains another; they swap when both are ready.

---

## Cheat Sheet
| Aspect | Detail |
|---|---|
| Threads | Exactly 2 |
| Method | `exchange(V)` — blocks until partner arrives |
| Returns | Partner's object |
| Timeout | `exchange(V, time, unit)` → `TimeoutException` |
| Use for | Double-buffering, pipeline handoffs |
