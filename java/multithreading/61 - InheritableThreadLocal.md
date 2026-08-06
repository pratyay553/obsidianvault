# 61 - InheritableThreadLocal

Tags: #thread-local-storage #section10
Related: [[60 - ThreadLocal]] | [[13 - Daemon Thread]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand `InheritableThreadLocal` — a `ThreadLocal` variant where child threads automatically inherit the parent thread's values.

---

## Beginner Explanation
Normal `ThreadLocal` values are invisible to child threads. `InheritableThreadLocal` automatically copies the parent thread's values into any child thread created from it.

---

## Analogy
A family business: when a new employee (child thread) is hired by the owner (parent thread), they automatically inherit the company culture and settings (thread-local values). But after hiring, changes in either direction don't affect the other.

---

## Technical Deep Dive

```java
InheritableThreadLocal<String> itl = new InheritableThreadLocal<>();
```

### How it works
When `Thread.start()` is called, the JVM copies the parent's `InheritableThreadLocal` entries into the child's `ThreadLocalMap`. After the copy, changes are **independent** — parent and child have separate copies.

### Customizing inheritance
Override `childValue(T parentValue)` to transform the value:
```java
InheritableThreadLocal<List<String>> audit = new InheritableThreadLocal<>() {
    @Override
    protected List<String> childValue(List<String> parentValue) {
        return new ArrayList<>(parentValue); // deep copy for independence
    }
};
```

### ⚠️ Does NOT work with thread pools
Thread pool threads are created once and reused. The child thread was created long before your request — it inherits the values from whatever thread created the pool, not from the request-handling thread calling `submit()`.

**Fix**: Use `TransmittableThreadLocal` (TTL, from Alibaba open source) or pass context explicitly.

---

## Code

```java
public class InheritableThreadLocalDemo {
    static InheritableThreadLocal<String> context = new InheritableThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        context.set("parent-value");
        System.out.println("Parent: " + context.get()); // parent-value

        Thread child = new Thread(() -> {
            System.out.println("Child inherited: " + context.get()); // parent-value
            context.set("child-value"); // only affects child
            System.out.println("Child after set: " + context.get()); // child-value
        });
        child.start();
        child.join();

        System.out.println("Parent after child set: " + context.get()); // still parent-value
    }
}
```

---

## Interview Questions

**Q: How does `InheritableThreadLocal` differ from `ThreadLocal`?**
> `InheritableThreadLocal` copies the parent thread's values into child threads at creation time. After the copy, parent and child values are independent.

**Q: Does `InheritableThreadLocal` work with thread pools?**
> No. Pool threads are pre-created and don't inherit values from the thread that submits a task. Use `TransmittableThreadLocal` or explicit context propagation.

---

## Cheat Sheet
| Aspect | ThreadLocal | InheritableThreadLocal |
|---|---|---|
| Child inherits | ❌ No | ✅ Yes (at creation) |
| Pool-safe | N/A | ❌ No |
| After inheritance | Independent | Independent |
| Customize copy | N/A | Override `childValue()` |
