# 56 - Phaser

Tags: #coordination-utilities #section9
Related: [[54 - CountDownLatch]] | [[55 - CyclicBarrier]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `Phaser` — the most flexible synchronization barrier in Java, combining and extending both `CountDownLatch` and `CyclicBarrier`.

---

## Beginner Explanation
`Phaser` is like a `CyclicBarrier` that can dynamically change the number of participating threads. Threads can register and deregister at runtime, and you can have as many phases as you want.

---

## Analogy
A relay race where the number of runners can change between laps. A runner can join mid-race or drop out, and the race continues with whoever's still registered.

---

## Technical Deep Dive

### API
```java
Phaser phaser = new Phaser(N);    // N initial parties
phaser.register();                // add one party dynamically
phaser.bulkRegister(N);           // add N parties
phaser.arrive();                  // arrive but don't wait
phaser.arriveAndAwaitAdvance();   // arrive and wait for all
phaser.arriveAndDeregister();     // arrive then leave permanently
phaser.awaitAdvance(phase);       // wait for phase to complete
phaser.getPhase();                // current phase number
phaser.getRegisteredParties();    // how many registered
phaser.getArrivedParties();       // how many have arrived this phase
```

### Phaser vs CyclicBarrier vs CountDownLatch

| Feature | CountDownLatch | CyclicBarrier | Phaser |
|---|---|---|---|
| Reusable | ❌ | ✅ | ✅ |
| Dynamic parties | ❌ | ❌ | ✅ |
| Multiple phases | ❌ | ✅ (cycles) | ✅ |
| Arrive without waiting | ❌ | ❌ | ✅ |
| Tree structure | ❌ | ❌ | ✅ |
| Termination control | ❌ | ❌ | ✅ (override `onAdvance`) |

### Terminating a Phaser
Override `onAdvance(phase, registeredParties)`:
```java
Phaser phaser = new Phaser(N) {
    @Override
    protected boolean onAdvance(int phase, int registeredParties) {
        return phase >= MAX_PHASES - 1; // return true to terminate
    }
};
```
When `onAdvance` returns `true`, the Phaser enters **termination state** — all threads waiting are released, and future `arrive*` calls return a negative phase number.

### Tiered/Tree Phasers
For very large numbers of parties, chain Phasers:
```java
Phaser root = new Phaser();
Phaser child1 = new Phaser(root, 0);
Phaser child2 = new Phaser(root, 0);
```

---

## Code

```java
import java.util.concurrent.*;

public class PhaserDemo {
    public static void main(String[] args) {
        int workers = 4;
        int phases  = 3;

        Phaser phaser = new Phaser(1); // 1 = main thread registers itself

        for (int i = 0; i < workers; i++) {
            phaser.register(); // register each worker
            final int id = i;
            new Thread(() -> {
                for (int p = 0; p < phases; p++) {
                    // Do phase work
                    try { Thread.sleep((long)(Math.random() * 300)); }
                    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                    System.out.printf("Worker %d done with phase %d%n", id, p);
                    phaser.arriveAndAwaitAdvance(); // sync at each phase
                }
                phaser.arriveAndDeregister(); // leave after all phases
            }).start();
        }

        // Main thread drives the phases
        for (int p = 0; p < phases; p++) {
            phaser.arriveAndAwaitAdvance(); // sync with workers each phase
            System.out.println("=== Phase " + p + " complete ===");
        }
        phaser.arriveAndDeregister(); // main deregisters

        System.out.println("Terminated: " + phaser.isTerminated());
    }
}
```

---

## Interview Questions

**Q: What makes `Phaser` more flexible than `CyclicBarrier`?**
> Dynamic party registration/deregistration, the ability to arrive without waiting (`arrive()`), tree-structured composition for scalability, and controllable termination via `onAdvance()`.

**Q: What does `arriveAndDeregister()` do?**
> The thread arrives at the current phase AND removes itself from future phases. The registered party count drops by one.

**Q: How do you terminate a `Phaser`?**
> Override `onAdvance(phase, registeredParties)` and return `true`. Once terminated, the phase number becomes negative.

---

## Cheat Sheet
| Method | Meaning |
|---|---|
| `arriveAndAwaitAdvance()` | Arrive + block until phase complete |
| `arrive()` | Arrive but don't block |
| `arriveAndDeregister()` | Arrive + leave permanently |
| `register()` | Add one new party |
| `getPhase()` | Current phase (negative = terminated) |
| `onAdvance()` | Override to control termination |
