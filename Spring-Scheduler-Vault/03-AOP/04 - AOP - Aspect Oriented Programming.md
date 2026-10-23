---
tags: [aop, cross-cutting, interview]
---

# AOP — Aspect-Oriented Programming

Related: [[01 - Spring Scheduling Basics]] · [[14 - Monitoring - Actuator and Logging]] · [[05 - Retry Pattern]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
AOP lets you inject **cross-cutting behavior** (logging, timing, error handling, auditing) into many methods **without editing those methods**. Instead you write it once, in an "aspect", and Spring weaves it in at runtime.

**Analogy**: Instead of putting a security guard inside every single room of a building (editing every method), you put **one guard at the doorway (the pointcut)** who checks everyone who enters/exits any matching room.

## 🔑 Vocabulary

| Term | Meaning |
|---|---|
| **Aspect** | The class containing the cross-cutting logic (`@Aspect`) |
| **Advice** | The actual code that runs (`@Before`, `@After`, `@Around`, etc.) |
| **Pointcut** | An expression describing *which* methods the advice applies to |
| **JoinPoint** | The specific method call currently being intercepted |
| **ProceedingJoinPoint** | A special `JoinPoint` used in `@Around` advice — lets you call `.proceed()` to actually run the original method |

## 🛠️ The Five Advice Types

```java
@Aspect
@Component
public class LoggingAspect {

    @Pointcut("execution(* com.etrans.scheduler.service..*(..))")
    public void serviceLayer() {}

    @Before("serviceLayer()")
    public void logBefore(JoinPoint jp) {
        log.info("Entering: {}", jp.getSignature());
    }

    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint jp, Object result) {
        log.info("{} returned: {}", jp.getSignature(), result);
    }

    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void logAfterThrowing(JoinPoint jp, Exception ex) {
        log.error("{} threw: {}", jp.getSignature(), ex.getMessage());
    }

    @After("serviceLayer()")
    public void logAfter(JoinPoint jp) {
        log.info("Exiting: {}", jp.getSignature());   // runs regardless of outcome (like 'finally')
    }

    @Around("serviceLayer()")
    public Object logExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();        // actually runs the real method
        log.info("{} took {}ms", pjp.getSignature(), System.currentTimeMillis() - start);
        return result;
    }
}
```

| Advice | Runs when | Can it stop/modify the call? |
|---|---|---|
| `@Before` | Before the method | Can throw to prevent execution, but can't change args easily |
| `@After` | Always, after (like `finally`) | No |
| `@AfterReturning` | Only on successful return | Can inspect the return value |
| `@AfterThrowing` | Only if an exception is thrown | Can inspect the exception |
| `@Around` | Wraps the entire call | **Yes** — full control: skip it, retry it, change args/return value |

`@Around` is the most powerful (and most asked about in interviews) because of `ProceedingJoinPoint.proceed()`.

## 🗂️ Aspects in This Module
- **`LoggingAspect`** — generic entry/exit/timing logs for every service call.
- **`MailAspect`** — email-specific concerns (e.g., logging every email sent/failed, maybe retry-triggering — see [[05 - Retry Pattern]]).

## ❓ Interview Q&A

**Q: How is AOP implemented under the hood in Spring?**
A: By default, via **dynamic proxies** — either JDK dynamic proxies (if the target implements an interface) or CGLIB proxies (subclassing, if it doesn't). Spring wraps your bean in a proxy that runs the advice, then delegates to the real object.

**Q: Why won't `@Before`/`@Around` fire when a method calls another method on `this` (self-invocation)?**
A: Because the proxy only intercepts calls that come **from outside the bean, through the proxy**. An internal `this.otherMethod()` call bypasses the proxy entirely — a classic AOP gotcha.

**Q: `@Around` vs `@Before` + `@After`?**
A: `@Around` gives you a single method with full control over whether/how the target method executes (you must call `proceed()`), including the ability to change arguments or the return value, or catch exceptions and retry. `@Before`/`@After` are simpler but can't do those things.

**Q: How would you build a retry mechanism using AOP instead of code duplication?**
A: Write an `@Around` advice with a pointcut matching a custom `@Retryable` annotation; inside, loop calling `pjp.proceed()` up to `MAX_RETRIES` times, catching exceptions and sleeping `RETRY_DELAY_MS` between attempts (see [[05 - Retry Pattern]]). This avoids repeating try/catch/retry code in every scheduler class.

**Q: Pointcut expression breakdown — what does `execution(* com.example..*(..))` mean?**
A: `execution(...)` = match method executions; first `*` = any return type; `com.example..*` = any class in that package or sub-packages; `(..)` = any number/type of arguments.
