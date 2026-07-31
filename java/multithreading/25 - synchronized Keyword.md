# synchronized Keyword

## Objective

By the end of this note, you will understand:

- Every syntactic form the `synchronized` keyword can take
- Precisely what each form locks on
- Practical guidance on choosing between synchronized methods and synchronized blocks
- Common real-world patterns and anti-patterns

This note is the practical, syntax-focused companion to [[23 - Monitor Lock]] and [[24 - Intrinsic Lock]] — those notes explained the underlying mechanism; this note is your complete reference for actually **writing** `synchronized` code correctly.

---

## Why should I learn this?

`synchronized` is the single most commonly used concurrency keyword in Java, and it has several syntactic forms that behave subtly differently. Many bugs and interview mistakes come not from misunderstanding "what synchronization is" conceptually, but from **getting the syntax wrong** — synchronizing on the wrong object, using a static method when an instance method was needed, or vice versa. This note is your complete, practical reference for getting the syntax right every time.

---

## Beginner Explanation

`synchronized` can be used in four main ways:

1. **Synchronized instance method** — locks the whole method body using the calling object's monitor.
2. **Synchronized static method** — locks the whole method body using the class's monitor.
3. **Synchronized block on `this`** — locks just part of a method, using the calling object's monitor.
4. **Synchronized block on a custom object** — locks just part of a method (or any code), using some other, specific object's monitor.

Think of these as four different-sized "padlocks" you can attach to your code, all doing the same fundamental job (only one thread through at a time) but at different levels of granularity and using different "keys" (monitor objects).

---

## Real World Analogy

**Shopping Mall Analogy**

Imagine four different levels of access control in a shopping mall:

1. **Synchronized instance method** = locking an entire individual store's front door — only one customer (thread) can be inside that specific store at a time.
2. **Synchronized static method** = locking the entire mall's main entrance — only one customer can be inside the whole mall complex at a time, affecting every store.
3. **Synchronized block on `this`** = locking only the fitting room inside a specific store, not the whole store — customers can browse the store freely, but only one at a time in the fitting room.
4. **Synchronized block on a custom object** = using a dedicated, separate lock (like a shared vault key) that could even be used to coordinate access between multiple different stores, independent of any one store's own door.

---

## Technical Explanation

### Form 1: Synchronized Instance Method

```java
class Account {
    private double balance;

    public synchronized void withdraw(double amount) {
        balance -= amount;
    }
}
```

- Locks on `this` — the specific `Account` instance the method is called on.
- Equivalent to wrapping the entire method body in `synchronized(this) { ... }`.
- Different instances have independent locks — `account1.withdraw()` and `account2.withdraw()` can run concurrently on different threads without blocking each other.

### Form 2: Synchronized Static Method

```java
class Account {
    private static int totalAccountsCreated = 0;

    public static synchronized void incrementAccountCount() {
        totalAccountsCreated++;
    }
}
```

- Locks on the **`Class` object** (`Account.class`) — shared across **all** instances and static contexts of this class.
- Only one thread can execute **any** synchronized static method of this class at a time, regardless of how many `Account` instances exist.

### Form 3: Synchronized Block on `this` (Partial Method Locking)

```java
class Account {
    private double balance;
    private String lastTransactionLog;

    public void withdraw(double amount) {
        // Some non-critical setup work here, NOT synchronized
        String description = "Withdrawal of " + amount;

        synchronized (this) {
            balance -= amount; // only THIS part needs protection
            lastTransactionLog = description;
        }
    }
}
```

- Locks only the specific block, not the entire method — useful for keeping the synchronized (and therefore potentially contended) section as **small** as possible (see [[27 - Critical Section]]).
- Still uses `this` as the lock, so it's equivalent in **scope** (which monitor) to a synchronized instance method, just smaller in **duration**.

### Form 4: Synchronized Block on a Custom, Dedicated Lock Object

```java
class Account {
    private double balance;
    private final Object balanceLock = new Object();

    public void withdraw(double amount) {
        synchronized (balanceLock) {
            balance -= amount;
        }
    }
}
```

- Uses a **dedicated** lock object (`balanceLock`), completely separate from `this`.
- This is often considered the **best practice** approach for library/production code: it avoids accidentally exposing your internal lock to external code that might also try to synchronize on `this` (potentially causing unexpected contention or, in the worst case, deadlocks — see [[36 - Deadlock]]) — since nobody outside this class has access to `balanceLock`.

### Choosing Between These Forms

|Situation|Recommended Form|
|---|---|
|Simple case, entire method needs protection, no external exposure concerns|Synchronized instance method|
|Need to protect only PART of a method (rest can run unsynchronized)|Synchronized block, ideally on a dedicated lock object|
|Need to protect static/class-level shared state|Synchronized static method (or block on `SomeClass.class`)|
|Writing library code where `this` might be exposed/synchronized on externally|Synchronized block on a dedicated, private lock object|

---

## Internal Working

Regardless of which syntactic form is used, the underlying mechanism is identical (see [[23 - Monitor Lock]]):

1. The JVM determines which object's monitor is targeted (`this`, `SomeClass.class`, or a custom object).
2. It attempts to acquire that specific monitor.
3. If successful, the protected code runs; if not, the thread blocks until the monitor becomes available.
4. Upon completion (or an exception propagating out), the monitor is automatically released.

The **only** difference between the forms is **which object's monitor gets used**, and **how much code** is covered by the lock (the whole method vs. just a block).

---

## Memory Diagram

```
   Account instance #1          Account instance #2
   ----------------------       ----------------------
   | monitor (this)      |      | monitor (this)      |
   | balanceLock monitor |      | balanceLock monitor |
   ----------------------       ----------------------
        (independent!)               (independent!)

   Account.class (shared, ONE per class, JVM-wide)
   ----------------------
   | monitor (class-level)|  <- used by synchronized STATIC methods
   ----------------------
```

---

## Execution Flow

```
Thread calls account1.withdraw(50)  [synchronized instance method]
        |
        v
Acquires account1's monitor (this)
        |
        v
Executes withdraw() body
        |
        v
Releases account1's monitor

   MEANWHILE, concurrently and WITHOUT blocking:

Thread calls account2.withdraw(30)  [different instance!]
        |
        v
Acquires account2's monitor (this) -- DIFFERENT monitor, no conflict
        |
        v
Executes withdraw() body
        |
        v
Releases account2's monitor
```

---

## Code Example

```java
public class SynchronizedKeywordDemo {

    static class Account {
        private double balance = 1000;
        private final Object balanceLock = new Object();
        private static int operationCount = 0;

        // Form 1: synchronized instance method
        public synchronized void withdrawViaMethod(double amount) {
            balance -= amount;
        }

        // Form 3/4: synchronized block on a dedicated lock object
        public void withdrawViaBlock(double amount) {
            synchronized (balanceLock) {
                balance -= amount;
            }
        }

        // Form 2: synchronized static method
        public static synchronized void recordOperation() {
            operationCount++;
        }

        public double getBalance() {
            return balance;
        }

        public static int getOperationCount() {
            return operationCount;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Account acc1 = new Account();
        Account acc2 = new Account();

        Runnable task1 = () -> {
            for (int i = 0; i < 1000; i++) {
                acc1.withdrawViaMethod(1);
                Account.recordOperation();
            }
        };

        Runnable task2 = () -> {
            for (int i = 0; i < 1000; i++) {
                acc2.withdrawViaBlock(1);
                Account.recordOperation();
            }
        };

        Thread t1 = new Thread(task1);
        Thread t2 = new Thread(task2);

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Account 1 final balance: " + acc1.getBalance() + " (expected 0)");
        System.out.println("Account 2 final balance: " + acc2.getBalance() + " (expected 0)");
        System.out.println("Total operations recorded: " + Account.getOperationCount() + " (expected 2000)");
    }
}
```

### Line-by-line explanation

1. `withdrawViaMethod` demonstrates Form 1 (synchronized instance method) — protects `acc1`'s own balance updates.
2. `withdrawViaBlock` demonstrates Form 3/4 (synchronized block on a dedicated lock) — protects `acc2`'s own balance updates via a separate lock object, `balanceLock`.
3. `recordOperation` demonstrates Form 2 (synchronized static method) — protects the shared `operationCount`, coordinating across **both** `acc1` and `acc2`'s operations, since it's static and shared class-wide.
4. Both accounts correctly reach a balance of `0` (started at 1000, withdrew 1000 total via 1000 operations of 1 each), and `operationCount` correctly totals `2000` (1000 from each thread) — demonstrating that all three synchronization forms correctly protect their respective shared data.

---

## Dry Run

|Check|Expected|Why|
|---|---|---|
|`acc1.getBalance()`|0|1000 - (1000 × 1), protected by instance method sync|
|`acc2.getBalance()`|0|1000 - (1000 × 1), protected by block-level sync on dedicated lock|
|`Account.getOperationCount()`|2000|1000 + 1000, protected by static method sync (shared monitor across both accounts)|

---

## Common Mistakes

1. **Synchronizing on `this` in library/public-facing classes without realizing external code could also synchronize on the same object.**
    
    - If external code does `synchronized(myLibraryObject) { ... indefinitely ... }`, it could unexpectedly block your own internal `synchronized` methods (which also use `this`), or even contribute to deadlocks. Using a dedicated, private lock object avoids this risk entirely.
2. **Mixing instance-level and static-level synchronization when protecting related data, expecting them to coordinate.**
    
    - As covered in [[23 - Monitor Lock]], these lock different monitors entirely — mixing them for logically related data provides **no** actual mutual exclusion between the two access paths.
3. **Synchronizing an entire large method when only a small part actually touches shared state.**
    
    - This unnecessarily increases contention and reduces throughput — extract just the necessary section into a synchronized block instead (see [[27 - Critical Section]]).
4. **Using a new object as the lock each time, instead of a stable, shared reference.**
    
    - E.g., `synchronized(new Object()) { ... }` provides **zero** actual mutual exclusion, since a brand new, unique object is created (and thus "locked") on every single call — no two threads would ever actually contend for the same monitor.

---

## Production Usage

- **Synchronized instance methods** are common for simple, self-contained thread-safe classes (e.g., a basic thread-safe counter or cache wrapper).
- **Synchronized blocks on dedicated lock objects** are the standard, recommended pattern in library and framework code, specifically to avoid exposing internal locking to external callers.
- **Synchronized static methods** are less common but appear in singleton initialization patterns, shared class-level counters/metrics, and similar globally-shared state scenarios.
- **Modern codebases increasingly prefer** `java.util.concurrent` utilities (`AtomicInteger`, `ConcurrentHashMap`, explicit `Lock`s) over hand-written `synchronized` blocks for anything beyond simple cases — but `synchronized` remains extremely common for straightforward critical sections throughout the Java ecosystem.

---

## Interview Questions

**Beginner:**

1. What are the four main syntactic forms of `synchronized` in Java?
2. What does a synchronized instance method lock on?
3. What does a synchronized static method lock on?

**Intermediate:** 4. Why might you prefer a synchronized block over a synchronized method? 5. Why is synchronizing on a dedicated, private lock object often considered better practice than synchronizing on `this`? 6. What's wrong with `synchronized(new Object()) { ... }`?

**Advanced:** 7. Explain a realistic scenario where synchronizing on `this` in a public class could lead to unexpected contention or deadlock risk from external code. 8. Why do synchronized static methods provide no mutual exclusion against synchronized instance methods, even on related data?

**Production-Level:** 9. In a library you're designing for external consumers, which synchronized form would you choose for internal thread safety, and why? 10. How would you refactor an overly broad synchronized method (locking the entire method) to reduce contention while preserving correctness?

---

## Follow-up Questions

- Can you synchronize on a primitive value, like an `int`? _(No — primitives don't have monitors; synchronized requires a reference to an actual object.)_
- Is it valid syntax to synchronize on `null`? _(No — attempting `synchronized(null) { ... }` throws a `NullPointerException` at runtime when that line executes.)_
- Can a synchronized block appear inside a loop, synchronizing and releasing on each iteration? _(Yes — this is a valid and sometimes useful pattern, e.g., minimizing how long the lock is held per iteration rather than holding it for the entire loop's duration.)_

---

## Best Practices

- Prefer synchronized blocks over synchronized methods when only part of the method needs protection — minimize the critical section (see [[27 - Critical Section]]).
- Use a dedicated, `private final Object lock = new Object();` for library/production code, rather than synchronizing on `this`, to avoid external interference.
- Never synchronize on a newly created object per call — this provides no actual mutual exclusion.
- Be deliberate about instance-level versus class-level (static) locking based on whether the protected data is per-instance or shared class-wide.

---

## Performance Notes

- Minimizing the size of synchronized sections (using blocks instead of whole methods where appropriate) directly reduces lock contention and improves throughput under concurrent load.
- Synchronizing an entire large method unnecessarily can serialize unrelated work that didn't actually need protection, needlessly hurting performance.

---

## Comparison Table

|Form|Locks On|Scope|
|---|---|---|
|`synchronized` instance method|`this`|Entire method body|
|`synchronized` static method|`SomeClass.class`|Entire method body|
|`synchronized(this) { }`|`this`|Just the block|
|`synchronized(customLock) { }`|`customLock` (dedicated object)|Just the block|

---

## Summary

- `synchronized` has four main syntactic forms: instance methods, static methods, blocks on `this`, and blocks on a dedicated custom lock object.
- Synchronized instance methods/blocks-on-`this` lock the specific object instance; synchronized static methods lock the shared `Class` object.
- Using a dedicated, private lock object (rather than `this`) is generally considered best practice for library/production code, avoiding external interference.
- Minimizing synchronized section size (blocks over whole methods) reduces contention and improves performance.
- All forms rely on the exact same underlying monitor mechanism (see [[23 - Monitor Lock]]) — the differences are purely about **which** monitor is used and **how much** code is protected.

---

## Cheat Sheet

|Code|Locks On|
|---|---|
|`public synchronized void foo() {}`|`this`|
|`public static synchronized void foo() {}`|`SomeClass.class`|
|`synchronized(this) { }`|`this`|
|`synchronized(SomeClass.class) { }`|The `Class` object|
|`synchronized(customLockObject) { }`|`customLockObject`|

---

## Keywords

synchronized, Synchronized Method, Synchronized Block, Dedicated Lock Object, Class Object Lock, Critical Section

---

## Related Notes

[[23 - Monitor Lock]] [[24 - Intrinsic Lock]] [[27 - Critical Section]] [[36 - Deadlock]]

---

## Practice Exercises

**Easy:**

1. Write a class with a synchronized instance method protecting a counter, and confirm correct results under concurrent access.
2. Rewrite the same class using a synchronized block on a dedicated lock object instead, and confirm identical correctness.

**Medium:** 3. Write a class with both a synchronized instance method and a synchronized static method, and empirically demonstrate (via timing/logging) that they don't block each other. 4. Refactor a method that unnecessarily synchronizes its entire body down to a minimal synchronized block around just the shared-state access.

**Hard:** 5. Design a small class where synchronizing on `this` could plausibly cause a deadlock risk from external code (e.g., simulate external code also synchronizing on the same instance), then fix it using a dedicated private lock object.

---

## Mini Project

**Project: Thread-Safe Bank Account Library**

Build a small `BankAccount` class intended to be used as a reusable library component:

1. Implement `deposit()` and `withdraw()` using a dedicated, private lock object (not `this`).
2. Add a static `totalAccountsCreated` counter, protected via a synchronized static method or block on the `Class` object.
3. Write a test harness with multiple threads performing concurrent deposits/withdrawals across multiple account instances, plus account creation, confirming all final values are correct.
4. Add comments explaining why each synchronization choice was made (block vs. method, instance vs. static, dedicated lock vs. `this`).

**Goal:** Practice making deliberate, justified synchronization design choices — exactly the kind of reasoning expected in real code reviews and system design interviews.

---

## Common Interview Trap

**Trap Question:** _"If I refactor a `synchronized` instance method into a `synchronized(this) { ... }` block wrapping the exact same code, does anything about the locking behavior change?"_

The trap is expecting some subtle difference. The correct answer: **no** — a synchronized instance method is, by definition, exactly equivalent to wrapping its entire body in `synchronized(this) { ... }`. There is no behavioral difference whatsoever between these two forms when they cover identical code; the only reason to prefer the block form is when you want to synchronize **less** than the entire method (i.e., a genuinely smaller critical section) or synchronize on a **different** object than `this`.

---

## Frequently Asked Questions

**Q: Can a synchronized block be nested inside another synchronized block?** A: Yes — as long as they're on the same object (reentrancy applies, see [[23 - Monitor Lock]]) or different objects (each acquired independently), nested synchronized blocks are valid and common.

**Q: Does `synchronized` work on interface methods?** A: You cannot mark an interface method itself as `synchronized` (interfaces don't have method bodies to synchronize, prior to default methods) — but a class's default or implementing method can certainly be `synchronized`.

**Q: Can constructors be synchronized?** A: No — the `synchronized` modifier is not allowed on constructors in Java; if you need to protect object construction-related shared state, you'd use a synchronized block inside the constructor instead, synchronizing on an appropriate object.

**Q: Is there a performance difference between synchronizing on `this` versus a dedicated custom lock object?** A: No inherent performance difference — the choice is about design safety (avoiding external interference) rather than raw speed.

---

## Revision Checklist

- [ ] I can list and correctly use all four syntactic forms of `synchronized`.
- [ ] I understand exactly what each form locks on.
- [ ] I understand why a dedicated lock object is often preferred over `this` in library code.
- [ ] I can identify and fix an overly broad synchronized method by refactoring to a minimal synchronized block.
- [ ] I understand why `synchronized(new Object()) {}` provides no real protection.

---

## Flashcards

**Q:** What does a synchronized instance method lock on? **A:** `this` — the specific object instance.

**Q:** What does a synchronized static method lock on? **A:** The `Class` object, shared across all instances.

**Q:** Why is synchronizing on a dedicated lock object often preferred over `this`? **A:** It prevents external code from accidentally interfering by also synchronizing on the same object.

**Q:** Is a synchronized instance method behaviorally different from `synchronized(this) {}` wrapping the whole method body? **A:** No — they are exactly equivalent.

**Q:** What's wrong with `synchronized(new Object()) { ... }`? **A:** A new, unique object is locked every call, so no two threads ever actually contend for the same monitor — it provides no real mutual exclusion.

---

## References

- Oracle Java Tutorials — Synchronized Methods: https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html
- Oracle Java Tutorials — Intrinsic Locks and Synchronization: https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html
- JLS (Java Language Specification) — Chapter 17: Threads and Locks