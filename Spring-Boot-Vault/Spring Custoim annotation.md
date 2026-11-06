When designing custom annotations in a Spring Boot application, a senior backend engineer should treat them as part of the application's architecture—not just as a way to reduce repeated code. A poorly designed annotation can hide business logic, make debugging difficult, and hurt maintainability.

Here are the key rules and best practices.

---

# 1. Create an annotation only when it represents a concept

Good examples:

```java
@Auditable
@Retryable
@RateLimited
@RequiresPermission
@MaskSensitiveData
```

Bad examples:

```java
@DoEverything
@CommonLogic
@ExecuteMethod
```

**Rule:**  
An annotation should express **intent**, not implementation.

---

# 2. Hide cross-cutting concerns, not business logic

Good candidates:

- Logging
    
- Metrics
    
- Validation
    
- Security
    
- Transactions
    
- Caching
    
- Auditingx
    
- Retry
    
- Rate limiting
    

Bad candidate:

```java
@CalculateSalary
```

Salary calculation belongs in a service.

---

# 3. One annotation = one responsibility

Bad:

```java
@SecureAndLogAndAuditAndRetry
```

Good:

```java
@Secured
@Auditable
@Retryable
```

Each annotation should solve one concern.

---

# 4. Don't surprise developers

Bad:

```java
@Validate
```

Hidden behavior:

- starts transaction
    
- publishes Kafka
    
- sends email
    
- writes DB
    

A developer reading

```java
@Validate
```

should not have to inspect ten classes to understand its effects.

---

# 5. Keep annotations declarative

Good:

```java
@Retryable(
    maxAttempts = 3,
    delay = 500
)
```

Bad:

```java
@Retryable(
    strategy = new RetryAlgorithm()
)
```

Annotations should contain metadata, not executable objects.

---

# 6. Use enums instead of strings

Bad

```java
@Permission("ADMIN")
```

Good

```java
@Permission(Role.ADMIN)
```

Benefits:

- compile-time safety
    
- IDE auto-complete
    
- easier refactoring
    

---

# 7. Always specify RetentionPolicy correctly

Most Spring annotations need

```java
@Retention(RetentionPolicy.RUNTIME)
```

because Spring uses reflection.

Wrong:

```java
RetentionPolicy.CLASS
```

Spring cannot see it at runtime.

---

# 8. Choose Target carefully

Instead of

```java
@Target(ElementType.TYPE)
```

use the smallest applicable target.

Example

```java
@Target(ElementType.METHOD)
```

or

```java
@Target({
    ElementType.TYPE,
    ElementType.METHOD
})
```

Don't allow invalid usage.

---

# 9. Document every attribute

Instead of

```java
int value();
```

prefer

```java
int maxRetries();
```

Much clearer.

---

# 10. Give sensible defaults

Good

```java
public @interface Retryable {

    int maxAttempts() default 3;

    long delay() default 1000;
}
```

Developers shouldn't have to specify common values every time.

---

# 11. Don't overuse annotation parameters

Bad

```java
@MyAnnotation(
    a="",
    b="",
    c="",
    d="",
    e="",
    f=""
)
```

Too many attributes usually indicate multiple responsibilities.

---

# 12. Compose annotations

Spring encourages meta-annotations.

Example

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)

@Service
@Transactional
public @interface DomainService {
}
```

Now developers write

```java
@DomainService
```

instead of

```java
@Service
@Transactional
```

---

# 13. Avoid annotation inheritance confusion

Understand

```java
@Inherited
```

It only works on class inheritance.

It does **not** work for:

- methods
    
- interfaces
    

Many developers misuse it.

---

# 14. Keep annotation processing lightweight

Bad

```text
Every API call

↓

Reflection

↓

Database query

↓

Network call

↓

Kafka

↓

Redis
```

Annotation processing should remain efficient. Heavy work should be delegated to services or asynchronous processing when appropriate.

---

# 15. Don't hide exceptions

Bad

```java
@Retryable
```

internally

```java
catch(Exception e){
}
```

Developers should know what failures can occur.

---

# 16. Name annotations like English

Good

```java
@Auditable
@Logged
@Cacheable
@Retryable
@Encrypted
```

Bad

```java
@Process
@Execute
@Handler
```

---

# 17. Avoid creating annotations for one class

If only one class uses

```java
@MySpecialAnnotation
```

it probably shouldn't exist.

Annotations should represent reusable concepts.

---

# 18. Test annotation behavior

Test:

- annotation detection
    
- AOP interception
    
- edge cases
    
- invalid configurations
    
- nested annotations
    
- inheritance
    
- proxy behavior
    

---

# 19. Consider proxy limitations

Spring AOP works through proxies.

This won't trigger advice:

```java
class UserService {

    @Auditable
    public void save(){}

    public void update(){
        save();   // self-invocation
    }
}
```

The call to `save()` bypasses the Spring proxy, so `@Auditable` won't be applied. Design annotations with this limitation in mind, or use alternatives such as separating the method into another bean or using AspectJ when appropriate.

---

# 20. Don't replace good APIs with annotations

Bad

```java
@SendEmail
```

Good

```java
emailService.send(...)
```

Annotations are better suited for cross-cutting concerns than for core business operations.

---

# 21. Keep annotations independent of infrastructure

Bad

```java
@KafkaRetry
```

Better

```java
@Retryable
```

Your annotation should express **behavior**, not a specific technology, unless the annotation is explicitly tied to that technology.

---

# 22. Make them discoverable

Provide:

- JavaDoc
    
- examples
    
- expected behavior
    
- limitations
    
- thread-safety considerations
    
- performance impact
    

A custom annotation becomes part of your framework and should be documented accordingly.

---

# 23. Prefer existing Spring annotations first

Before creating:

```java
@MyTransactional
```

ask whether

```java
@Transactional
```

already satisfies the requirement.

Likewise, consider existing annotations such as:

- `@Cacheable`
    
- `@Async`
    
- `@Validated`
    
- `@Retryable` (from Spring Retry)
    
- `@Scheduled`
    

Only create a custom annotation when it adds domain-specific meaning or combines behavior in a way that improves your application's design.

## A Senior Engineer's Checklist

Before introducing a custom annotation, ask:

- **Does it represent a clear architectural or domain concept?**
    
- **Is it handling a cross-cutting concern rather than business logic?**
    
- **Does it have a single, well-defined responsibility?**
    
- **Will another developer understand its behavior from its name and documentation?**
    
- **Can I reuse an existing Spring annotation instead?**
    
- **Will it work correctly with Spring's proxy model (AOP, transactions, etc.)?**
    
- **Are its defaults sensible and its parameters type-safe?**
    
- **Is it easy to test, debug, and maintain?**
    

If the answer to these questions is consistently "yes," the annotation is much more likely to remain a valuable architectural tool rather than becoming a source of hidden complexity.