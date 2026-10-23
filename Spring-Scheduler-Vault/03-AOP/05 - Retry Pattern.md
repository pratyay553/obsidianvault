---
tags: [resilience, scheduling, interview]
---

# Retry Pattern

Related: [[01 - Spring Scheduling Basics]] · [[04 - AOP - Aspect Oriented Programming]] · [[07 - REST and SOAP Clients]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Scheduled jobs frequently call **external, unreliable systems** (a flaky third-party API, a temporarily-down mail server). The retry pattern says: *if it fails, try again a few times before giving up*, instead of crashing the whole scheduled run on the first hiccup.

## 🛠️ The Classic Manual Implementation

```java
private static final int MAX_RETRIES = 3;
private static final long RETRY_DELAY_MS = 2000L;

public void sendWithRetry(Email email) {
    int attempt = 0;
    while (attempt < MAX_RETRIES) {
        try {
            mailService.send(email);
            return; // success — exit
        } catch (MailException ex) {
            attempt++;
            log.warn("Attempt {} failed: {}", attempt, ex.getMessage());
            if (attempt >= MAX_RETRIES) {
                log.error("Giving up after {} attempts", MAX_RETRIES);
                throw ex; // or send to a dead-letter/error table
            }
            try {
                Thread.sleep(RETRY_DELAY_MS);
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt(); // ⚠️ restore interrupt status!
                throw new RuntimeException(ie);
            }
        }
    }
}
```

### 🔑 The one line everyone forgets: `Thread.currentThread().interrupt()`
When you catch `InterruptedException`, the JVM has already cleared the thread's "interrupted" flag. If you swallow the exception without restoring that flag, code further up the call stack has no way of knowing the thread was asked to stop. **Always re-interrupt** unless you're deliberately and fully handling the interruption right there.

## 📈 Leveling Up: Exponential Backoff
Fixed delays hammer a struggling service at a constant rate. Better:

```java
long delay = RETRY_DELAY_MS * (long) Math.pow(2, attempt); // 2s, 4s, 8s...
```

## 🆚 Manual retry loop vs Spring Retry vs AOP-based retry

| Approach | Pros | Cons |
|---|---|---|
| Manual try/catch loop (shown above) | Simple, explicit, no new dependency | Duplicated across every client/scheduler class |
| `spring-retry` (`@Retryable`/`@Recover`) | Declarative, backoff policies built-in | Extra dependency; less visible control flow |
| Custom `@Around` AOP advice (see [[04 - AOP - Aspect Oriented Programming]]) | DRY — one aspect, reusable annotation | More setup, less obvious to new devs |

## ❓ Interview Q&A

**Q: Why is `Thread.sleep()` inside a loop risky, and what should you always do?**
A: It throws a checked `InterruptedException`; you must catch it, and per Java concurrency best practice, call `Thread.currentThread().interrupt()` to restore the interrupt flag before continuing/rethrowing — otherwise you silently swallow a shutdown/cancellation signal.

**Q: What's wrong with retrying *every* exception type?**
A: Not all failures are transient. Retrying a `400 Bad Request` (a permanent client error) 3 times just wastes time and can mask a real bug — you should retry only on exceptions that indicate a transient condition (timeouts, `503`s, connection resets), not on business/validation errors.

**Q: How would you avoid duplicating this retry loop across `HindalcoClient`, `ODTSClient`, `OmnicommClient`, etc.?**
A: Extract it into a reusable utility method, a generic `RetryTemplate`-style wrapper, or — best — an AOP `@Around` advice keyed to a custom `@Retryable` annotation, so each client method just gets annotated instead of re-implementing the loop.

**Q: What's exponential backoff and why use it?**
A: Increasing the delay between retries exponentially (e.g., 2s, 4s, 8s) instead of a fixed delay — this reduces load on a struggling downstream service and improves the odds it has recovered by the time you retry.
