# Race Condition

## Objective

By the end of this note, you will understand:

- The precise, formal definition of a race condition
- The different categories of race conditions (read-modify-write, check-then-act)
- Why race conditions are so notoriously hard to detect and reproduce
- How to systematically identify and fix them in real code

This note formalizes the problem first introduced conceptually in [[22 - Synchronization]] and builds directly on [[23 - Monitor Lock]] through [[25 - synchronized Keyword]] for the fix.

---

## Why should I learn this?

"Race condition" is the term you'll use constantly, in code reviews, incident postmortems, and interviews, to describe the single most common category of concurrency bug. Being able to precisely identify a race condition in code — not just recognize "this feels unsafe" — is a core professional skill. This note gives you a systematic framework for spotting them.

---

## Beginner Explanation

A **race condition** occurs when the correctness of a program depends on the **relative timing** or **interleaving** of multiple threads — meaning the outcome can change depending on which thread happens to "win the race" to execute first, or how their operations happen to interleave.

Think of two people racing to be the last one to write their name on a sign-up sheet with only one remaining slot. If both see the same "1 slot left" status at the same moment and both try to grab it, the actual outcome (who gets the slot, or whether the sheet ends up corrupted with both names crammed into one line) depends entirely on the exact **timing** of their actions — not on any predictable, well-defined logic.

---

## Real World Analogy

**Airport Analogy**

Imagine two gate agents, working independently, both checking "how many seats are left on this flight" before assigning the very last seat to two different standby passengers:

1. Agent A checks the system: "1 seat left." Decides to assign it to Passenger A.
2. Agent B checks the system, **before** Agent A has finished the assignment: also sees "1 seat left." Decides to assign it to Passenger B.
3. Both agents proceed to assign the same single seat to two different passengers.

The airline now has **two passengers assigned to the same seat** — an overbooking bug caused entirely by the **timing** of the two "check, then act" sequences overlapping. If Agent B had checked even a fraction of a second later (after Agent A's assignment was fully recorded), they would have correctly seen "0 seats left" and avoided the conflict. This exact "check-then-act" pattern is one of the two classic categories of race condition, explained formally below.

---

## Technical Explanation

### Formal Definition

A **race condition** is a situation where the behavior of a program depends on the relative timing of events (typically, the interleaving of operations from multiple threads) in a way that can produce incorrect or inconsistent results.

### Category 1: Read-Modify-Write Race

This is the pattern already covered in [[22 - Synchronization]]'s `count++` example:

```java
count++; // Read count, compute count+1, write it back -- 3 separate steps
```

If two threads interleave these three steps badly, one thread's update can be **lost**, as demonstrated in that note's dry run.

### Category 2: Check-Then-Act Race

This is a different, equally common pattern, illustrated by the airport analogy above:

```java
if (map.get(key) == null) {   // CHECK
    map.put(key, computeExpensiveValue()); // ACT
}
```

Even if `map` itself is a thread-safe collection, this **combination** of "check if absent, then put" is **not** atomic as a whole — two threads can both see `null` during their respective checks (before either has performed its `put`), and both proceed to compute and insert a value, potentially doing redundant work or, worse, one thread's `put` silently overwriting the other's in ways that violate the intended "only insert if absent" logic.

**The Fix**: Use an atomic, purpose-built operation instead of separate check-then-act steps:

```java
map.putIfAbsent(key, computeExpensiveValue()); // ATOMIC check-and-act, single operation
```

(See [[52 - ConcurrentHashMap]] for more on `putIfAbsent()` and similar atomic compound operations.)

### Why Race Conditions Are Hard to Detect

1. **They're timing-dependent**: The exact interleaving that triggers the bug might occur only 1 in 10,000 executions (or far less often) — meaning code can pass extensive testing and still fail unpredictably in production.
2. **They often depend on system load**: A race condition might never manifest on a fast, lightly-loaded development machine, but appear frequently under heavy production load where context switches (see [[12 - Context Switching]]) happen more often at "inconvenient" moments.
3. **Debuggers can mask them**: Stepping through code with a debugger changes the timing entirely, often making the race condition disappear while you're actively trying to observe it (sometimes called a "Heisenbug").
4. **They don't always crash the program**: Unlike a `NullPointerException`, a race condition might just silently produce a **slightly wrong** result (like a counter being off by a small amount), which can go unnoticed for a long time.

---

## Internal Working

Using the check-then-act example, here's the precise interleaving that causes the bug:

1. Thread A calls `map.get(key)`, receives `null` (key isn't present yet).
2. **Context switch** — Thread A is paused before it can call `put()`.
3. Thread B calls `map.get(key)`, **also** receives `null` (Thread A hasn't inserted anything yet).
4. Thread B computes a value and calls `map.put(key, valueB)`.
5. Thread A resumes, computes its own value (redundant work already done by B), and calls `map.put(key, valueA)`, **overwriting** Thread B's just-inserted value.
6. **Result**: redundant computation occurred, and the final stored value depends entirely on which thread's `put()` happened to execute last — a non-deterministic, incorrect outcome from the perspective of "only insert if truly absent."

---

## Memory Diagram

```
        Shared Map: { }  (empty initially)

   Thread A                          Thread B
   --------                          --------
   get(key) -> null                  
   [context switch]
                                     get(key) -> null  (still empty!)
                                     compute valueB
                                     put(key, valueB)  -> Map: { key: valueB }
   compute valueA (REDUNDANT WORK)
   put(key, valueA)                  -> Map: { key: valueA }  (overwrote B's insert!)
```

---

## Execution Flow

```
Two threads race to "check if absent, then insert"
        |
        v
Both threads independently CHECK -- both see "absent"
        |
        v
Both threads independently ACT -- both compute and insert
        |
        v
Whichever ACT happens last "wins" -- the other's work
is silently discarded, AND redundant computation occurred
        |
        v
Bug: "only insert if absent" logic was violated,
even though each individual operation "looked" correct in isolation
```

---

## Code Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

public class RaceConditionDemo {

    public static void main(String[] args) throws InterruptedException {

        // --- Demonstrating check-then-act race ---
        Map<String, Integer> unsafeMap = new HashMap<>(); // not even thread-safe for basic ops!
        AtomicInteger computeCallCount = new AtomicInteger(0);

        Runnable checkThenActTask = () -> {
            if (!unsafeMap.containsKey("key")) {          // CHECK
                computeCallCount.incrementAndGet();          // simulate "expensive computation"
                unsafeMap.put("key", computeCallCount.get()); // ACT
            }
        };

        Thread t1 = new Thread(checkThenActTask);
        Thread t2 = new Thread(checkThenActTask);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Expensive computation ran " + computeCallCount.get() +
                " times (ideally should be 1, since 'key' should only need computing once)");

        // --- Demonstrating the FIX using an atomic compound operation ---
        Map<String, Integer> safeMap = new ConcurrentHashMap<>();
        AtomicInteger safeComputeCallCount = new AtomicInteger(0);

        Runnable atomicTask = () -> {
            safeMap.computeIfAbsent("key", k -> safeComputeCallCount.incrementAndGet());
        };

        Thread t3 = new Thread(atomicTask);
        Thread t4 = new Thread(atomicTask);
        t3.start();
        t4.start();
        t3.join();
        t4.join();

        System.out.println("Safe version: expensive computation ran " +
                safeComputeCallCount.get() + " times (should reliably be 1)");
    }
}
```

### Line-by-line explanation

1. `checkThenActTask` demonstrates the classic check-then-act race: two threads might both see `"key"` as absent and both perform the "expensive computation" — the printed count can (though won't always, due to timing variability) show `2` instead of the intended `1`.
2. `atomicTask` fixes this using `ConcurrentHashMap.computeIfAbsent()`, which performs the check-and-insert as a **single atomic operation** — guaranteeing the computation function runs **at most once** per key, no matter how many threads race to call it concurrently.
3. This example demonstrates both the bug **and** its idiomatic fix side by side.

---

## Dry Run

Running the unsafe version multiple times might show:

```
Expensive computation ran 1 times (ideally should be 1, since 'key' should only need computing once)
```

(sometimes)

or:

```
Expensive computation ran 2 times (ideally should be 1, since 'key' should only need computing once)
```

(other times — the race condition doesn't trigger every single run, which is exactly what makes it dangerous)

The safe version, however, **reliably** shows:

```
Safe version: expensive computation ran 1 times (should reliably be 1)
```

every single time you run it.

---

## Common Mistakes

1. **Only thinking about read-modify-write races, and missing check-then-act races.**
    
    - Many developers correctly synchronize simple counter increments but miss the equally common (and equally dangerous) "check if X, then do Y" pattern scattered throughout typical business logic (e.g., "if user doesn't exist, create them").
2. **Assuming a thread-safe collection makes ALL operations on it safe.**
    
    - `ConcurrentHashMap` guarantees individual operations (like a single `get()` or `put()`) are thread-safe, but a **sequence** of operations (like `get()` followed by a separate `put()`) is **not** automatically atomic as a combined unit — you must use purpose-built atomic compound methods (`putIfAbsent()`, `computeIfAbsent()`, etc.) for that.
3. **Testing on a lightly-loaded development machine and concluding "no race condition exists" because it never reproduced.**
    
    - Absence of observed failure is not proof of absence of a race condition — proper code review and static reasoning about shared mutable state matters more than relying purely on testing to catch these bugs.
4. **Fixing a race condition by "just adding `synchronized` somewhere," without carefully identifying the exact shared state and the exact critical section boundaries.**
    
    - Partial or misplaced synchronization can leave the race condition partially unfixed, or introduce new problems (like unnecessary contention or deadlock risk) without fully solving the original issue.

---

## Production Usage

- **The airport overbooking scenario is directly analogous to real e-commerce "last item in stock" bugs** — two customers both see "1 item left," both complete checkout, and the store oversells by one unit — a classic, real check-then-act race condition with direct business impact.
- **User registration systems** commonly have check-then-act races around "does this username/email already exist?" checks followed by account creation — properly handled via database-level unique constraints and/or atomic operations, not just application-level checks.
- **Caching layers** are a very common source of check-then-act races (`if not cached, compute and cache`), which is exactly why `ConcurrentHashMap.computeIfAbsent()` and similar atomic compound methods exist and are so widely used in production caching code.
- **Distributed systems** face an even broader version of this problem (races across multiple machines, not just threads), typically requiring distributed locks or consensus protocols — a topic beyond a single-JVM vault like this one, but conceptually rooted in the exact same "timing-dependent correctness" issue.

---

## Interview Questions

**Beginner:**

1. What is a race condition?
2. What makes race conditions particularly difficult to detect through testing?
3. Give an everyday, non-code example of a race condition.

**Intermediate:** 4. What is the difference between a "read-modify-write" race and a "check-then-act" race? 5. Why doesn't using a thread-safe collection automatically eliminate all race conditions involving that collection? 6. What atomic compound method would you use to fix a "check if absent, then insert" race on a `ConcurrentHashMap`?

**Advanced:** 7. Why can a race condition pass extensive automated testing and still fail in production? 8. Explain, using a step-by-step interleaving, how two threads performing "check if absent, then insert" can both proceed to insert, defeating the intended logic.

**Production-Level:** 9. Describe a real e-commerce or business scenario where a check-then-act race condition could cause a costly bug, and how you'd fix it. 10. How would you approach code review specifically to catch potential race conditions that automated tests might miss?

---

## Follow-up Questions

- Is every unsynchronized access to shared mutable data automatically a race condition? _(In the broad sense, yes — any unsynchronized concurrent access with at least one writer creates the _potential_ for a race condition, even if a specific test run happens not to trigger a visibly incorrect result.)_
- Can a race condition exist even if every individual method involved is itself thread-safe? _(Yes — this is exactly the check-then-act problem: individually safe operations, combined unsafely, still create a race.)_
- Does adding `synchronized` around the entire check-then-act sequence fix the problem? _(Yes, if done correctly — wrapping the whole check-and-act sequence in a single synchronized block, using a consistent lock, restores atomicity for the combined operation; alternatively, purpose-built atomic methods like `computeIfAbsent()` achieve the same result more idiomatically for collections.)_

---

## Best Practices

- When reviewing code, explicitly look for **any** sequence of "read/check" followed by a separate "write/act" on shared mutable state — this pattern is a race condition red flag even if each step individually looks safe.
- Prefer purpose-built atomic compound operations (`putIfAbsent()`, `computeIfAbsent()`, `AtomicInteger.compareAndSet()`, etc.) over manually composed check-then-act logic wherever available.
- Don't rely on testing alone to catch race conditions — reason carefully about shared mutable state during design and code review.
- When you must manually synchronize a check-then-act sequence, ensure the **entire** sequence (both the check and the act) is covered by the **same** lock, held continuously across both steps.

---

## Performance Notes

- Fixing a check-then-act race by wrapping it in a broad `synchronized` block can introduce more contention than using a purpose-built atomic method (like `computeIfAbsent()`), which is often internally optimized for exactly this pattern.
- The performance cost of properly fixing a race condition is almost always dramatically smaller than the cost (in debugging time, data corruption, and lost trust) of a race condition reaching production undetected.

---

## Comparison Table

|Race Type|Example Pattern|Typical Fix|
|---|---|---|
|Read-Modify-Write|`count++`|`synchronized`, or `AtomicInteger` (see [[30 - Atomic Variables]])|
|Check-Then-Act|`if (!map.containsKey(k)) map.put(k, v)`|`map.putIfAbsent(k, v)` / `computeIfAbsent()`|

---

## Summary

- A race condition occurs when program correctness depends on the timing/interleaving of concurrent operations.
- Two classic categories: **read-modify-write** races (like `count++`) and **check-then-act** races (like "if absent, then insert").
- Race conditions are notoriously hard to detect because they're timing-dependent, often load-dependent, and can silently produce slightly wrong results rather than obvious crashes.
- Fixing them requires ensuring the **entire** relevant operation (not just individual steps) is atomic — via proper synchronization or purpose-built atomic compound methods.
- Testing alone is insufficient to rule out race conditions — careful reasoning about shared mutable state during design and review is essential.

---

## Cheat Sheet

|Symptom|Likely Race Condition Type|
|---|---|
|Counter/sum lower than expected total|Read-modify-write race|
|Duplicate work, or "should be unique" data corrupted|Check-then-act race|
|Bug appears rarely, inconsistently, under load|Classic race condition signature|
|Fix: wrap entire sequence in one lock|Manual synchronization fix|
|Fix: use `putIfAbsent`/`computeIfAbsent`/CAS methods|Idiomatic atomic compound operation fix|

---

## Keywords

Race Condition, Read-Modify-Write, Check-Then-Act, Atomicity, computeIfAbsent, putIfAbsent, Non-Deterministic Bug, Heisenbug

---

## Related Notes

[[22 - Synchronization]] [[25 - synchronized Keyword]] [[30 - Atomic Variables]] [[31 - CAS]] [[52 - ConcurrentHashMap]]

---

## Practice Exercises

**Easy:**

1. Reproduce the read-modify-write race from [[22 - Synchronization]] and confirm you can explain, step by step, why it occurs.
2. Reproduce the check-then-act race from this note's Code Example, and observe the "expensive computation ran 2 times" result on at least one run.

**Medium:** 3. Fix the check-then-act example using a manually synchronized block (instead of `computeIfAbsent()`), ensuring correctness, and compare the code complexity to the atomic-method version. 4. Write a check-then-act race involving a simple `if (list.isEmpty()) list.add(item)` pattern, and demonstrate the bug with concurrent threads.

**Hard:** 5. Design and implement a simulated "last item in stock" e-commerce scenario with a shared inventory count, demonstrate the overselling bug under concurrent "purchase" threads, then fix it properly using appropriate synchronization or atomic operations.

---

## Mini Project

**Project: Race Condition Bug Bounty Simulator**

Build a small console application that:

1. Implements 3 different intentionally-buggy scenarios: a read-modify-write counter race, a check-then-act cache race, and a "last item in stock" inventory race.
2. Runs each scenario many times (e.g., 50 runs) under concurrent load, recording how often each produces an incorrect result versus a correct one.
3. Implements the correct fix for each scenario, and re-runs the same 50-trial experiment, confirming 100% correctness after the fix.
4. Prints a summary report comparing "before fix" failure rate versus "after fix" failure rate for all three scenarios.

**Goal:** Build direct, statistical, hands-on evidence of race conditions' unpredictable nature, and concrete confirmation that proper fixes eliminate them entirely — turning an abstract warning into measured proof.

---

## Common Interview Trap

**Trap Question:** _"If I use `ConcurrentHashMap` instead of a regular `HashMap`, does that automatically make my `if (!map.containsKey(k)) map.put(k, v)` code thread-safe?"_

The trap is assuming "thread-safe collection" means "every possible sequence of operations on it is automatically atomic." The correct answer: **no** — `ConcurrentHashMap` guarantees each **individual** operation (`containsKey()`, `put()`, etc.) is internally thread-safe and won't corrupt the map's internal structure. But the **combination** of a separate `containsKey()` check followed by a separate `put()` call is still a check-then-act race condition, because another thread can execute its own `containsKey()`/`put()` sequence in between your two calls. The fix requires using `ConcurrentHashMap`'s purpose-built atomic compound methods, like `putIfAbsent()` or `computeIfAbsent()`, which perform the check-and-act as a single, truly atomic operation.

---

## Frequently Asked Questions

**Q: Is a race condition always a bug, or can it sometimes be intentional/harmless?** A: In the vast majority of real-world cases, race conditions are unintentional bugs; genuinely "benign" races (where any possible interleaving still produces an acceptable result) exist in theory but require extremely careful, expert-level reasoning to design safely — as a general rule, treat any identified race condition as a bug to fix.

**Q: Can race conditions occur even in single-core systems?** A: Yes — even on a single core, preemptive context switching (see [[12 - Context Switching]]) can interleave two threads' operations at nearly any point, producing the same race condition risks as on multi-core systems.

**Q: Are race conditions unique to Java?** A: No — race conditions are a fundamental concept in concurrent programming across all languages and platforms; Java's specific tools for preventing them (`synchronized`, atomic classes, concurrent collections) are Java-specific, but the underlying problem is universal.

**Q: Can automated tools detect race conditions?** A: Some static analysis tools and specialized dynamic analysis tools (like thread sanitizers in other ecosystems, or certain Java-specific race detectors) can catch some race conditions, but they're not comprehensive — careful design and code review remain essential complements to tooling.

---

## Revision Checklist

- [ ] I can precisely define a race condition, not just describe it vaguely.
- [ ] I can distinguish between read-modify-write races and check-then-act races.
- [ ] I understand why a thread-safe collection doesn't automatically make multi-step sequences on it safe.
- [ ] I know the idiomatic atomic compound methods (`putIfAbsent`, `computeIfAbsent`) for fixing common check-then-act races.
- [ ] I understand why race conditions are so hard to catch through testing alone.

---

## Flashcards

**Q:** What is a race condition? **A:** A situation where program correctness depends on the timing/interleaving of concurrent operations.

**Q:** What are the two classic categories of race conditions? **A:** Read-modify-write races and check-then-act races.

**Q:** Does using `ConcurrentHashMap` automatically make a `containsKey()` + `put()` sequence thread-safe? **A:** No — use `putIfAbsent()` or `computeIfAbsent()` for an atomic combined operation instead.

**Q:** Why are race conditions hard to catch through testing? **A:** They're timing-dependent and often only manifest rarely or under specific load conditions, unlike a typical crash-on-every-run bug.

**Q:** What's the general fix strategy for a race condition? **A:** Ensure the entire relevant sequence of operations is atomic — via proper synchronization covering the whole sequence, or a purpose-built atomic compound method.

---

## References

- Oracle Java Tutorials — Concurrency: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- `java.util.concurrent.ConcurrentHashMap` API Documentation (`putIfAbsent`, `computeIfAbsent`)
- `java.util.concurrent.atomic.AtomicInteger` API Documentation