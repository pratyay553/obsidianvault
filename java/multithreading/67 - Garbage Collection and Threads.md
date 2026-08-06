# 67 - Garbage Collection and Threads

Tags: #java-memory-model #section11 #jvm
Related: [[68 - JVM Thread Management]] | [[72 - Performance Tuning]] | [[62 - Java Memory Model]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand how Java's Garbage Collector interacts with threads — including STW pauses, safepoints, and GC thread impact on latency.

---

## Beginner Explanation
The GC runs on its own threads in the background. Periodically it must pause all your application threads (Stop-The-World) to safely move objects. These pauses are the main way GC affects multithreaded application performance.

---

## Analogy
A janitor cleaning a shared office. For light cleaning, the janitor works around the staff. But for deep cleaning (moving furniture), everyone must leave the building (Stop-The-World pause) so the janitor can safely move everything without things being used.

---

## Technical Deep Dive

### GC threads
Modern GC algorithms use multiple GC threads:
- **Concurrent threads** — run alongside your application threads (G1, ZGC, Shenandoah)
- **Stop-The-World (STW) threads** — require all application threads to pause

### Stop-The-World (STW) pauses
During an STW pause:
- All application threads are frozen at a **safepoint**
- GC threads do their work (mark, sweep, compact, copy)
- Application threads resume

A 200ms STW pause on a request-handling server = 200ms added to every request in flight during that pause.

### Safepoints
A safepoint is a point in code where the JVM knows the exact state of all objects and the stack — making it safe to inspect/move them. Common safepoint locations:
- Method returns
- Loop back edges (JIT-compiled code)
- Object allocation sites

When the JVM needs a safepoint (STW), it sets a flag. Each thread reaches its next safepoint and pauses. The GC starts only when **all** threads have reached safepoints.

### Time To Safepoint (TTSP)
If one thread is in a tight loop with few safepoints, it delays the STW pause for all other threads. This is called high TTSP. JVM flag `-XX:+PrintGCApplicationStoppedTime` shows total pause including TTSP.

### GC algorithms and threading

| GC | STW phases | Concurrent phases | Java version |
|---|---|---|---|
| Serial GC | All phases | None | All |
| Parallel GC | All phases | None | All |
| G1 GC | Young GC, mixed GC | Marking, cleanup | Java 9+ default |
| ZGC | Very short (<1ms) | Almost everything | Java 15+ |
| Shenandoah | Very short | Almost everything | Java 12+ |

### How threads affect GC
- **More live objects** = more GC work
- **More threads** = more allocation → more frequent GC
- **Large thread stacks** = more memory pressure (each thread stack is ~512KB–2MB)
- **Thread-local allocation buffers (TLABs)** — each thread has a small private heap buffer to allocate from without lock contention

### GC tuning for threaded apps
```
-Xms4g -Xmx4g               # fix heap size — avoid expansion pauses
-XX:+UseG1GC                 # G1 for most apps
-XX:MaxGCPauseMillis=200     # target max pause
-XX:+UseZGC                  # sub-ms pauses — Java 15+
-XX:ParallelGCThreads=N      # STW GC threads
-XX:ConcGCThreads=N          # concurrent GC threads
```

---

## Code

```java
public class GCThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        // Print GC thread info
        java.lang.management.ManagementFactory.getThreadMXBean()
            .dumpAllThreads(false, false).stream()
            .filter(ti -> ti.getThreadName().contains("GC") ||
                          ti.getThreadName().contains("Concurrent"))
            .forEach(ti -> System.out.println("GC Thread: " + ti.getThreadName()));

        // Trigger allocation to observe GC
        System.out.println("Allocating...");
        for (int i = 0; i < 1_000_000; i++) {
            byte[] b = new byte[1024]; // 1KB — triggers GC
        }
        System.gc(); // request GC (not guaranteed)
        Thread.sleep(1000);

        // Check GC stats
        java.lang.management.ManagementFactory.getGarbageCollectorMXBeans()
            .forEach(gc -> System.out.printf("GC: %-30s  count=%d  time=%dms%n",
                gc.getName(), gc.getCollectionCount(), gc.getCollectionTime()));
    }
}
```

---

## Interview Questions

**Q: What is a Stop-The-World (STW) pause?**
> A period where all application threads are frozen at safepoints so the GC can safely inspect and move objects. All requests in flight are delayed for the duration.

**Q: What is a safepoint?**
> A location in code where the JVM knows the complete state of all objects and references, making it safe to perform GC work. Threads must reach a safepoint before an STW pause begins.

**Q: How does ZGC improve on G1?**
> ZGC performs almost all its work concurrently (alongside application threads), keeping STW pauses under 1ms regardless of heap size. G1 still has multi-millisecond STW pauses for young/mixed collections.

**Q: What is a TLAB?**
> Thread-Local Allocation Buffer — a small private chunk of heap assigned to each thread. Allocation within a TLAB requires no locking. When full, the thread requests a new TLAB from the heap (which does require locking).

---

## Cheat Sheet
| Concept | Detail |
|---|---|
| STW pause | All threads frozen; GC runs |
| Safepoint | Safe state for GC inspection |
| TTSP | Time for all threads to reach safepoint |
| TLAB | Per-thread allocation buffer — no lock needed |
| Low-pause GC | ZGC, Shenandoah |
| Flag | `-XX:+UseZGC`, `-XX:MaxGCPauseMillis=N` |
