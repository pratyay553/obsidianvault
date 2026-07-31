---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Intermediate
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Bean Lifecycle & Scopes

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring beans go through a precise lifecycle — instantiation, dependency injection, initialization callbacks, active use, and destruction — and their scope (singleton, prototype, request, session) controls how many instances the container creates and for how long they live.

## 🧱 Key Concepts & Mechanics

### Bean Lifecycle Phases
1. **Instantiation** — Spring calls the constructor (or factory method)
2. **Dependency Injection** — fields/setters/constructor args populated
3. **`BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware`** — optional infrastructure callbacks
4. **`BeanPostProcessor#postProcessBeforeInitialization`** — pre-init hooks (e.g., `@PostConstruct` is processed here)
5. **`@PostConstruct` / `InitializingBean#afterPropertiesSet` / `init-method`** — initialization callbacks
6. **`BeanPostProcessor#postProcessAfterInitialization`** — post-init hooks (AOP proxies created here)
7. **Bean is ready** — used by the application
8. **`@PreDestroy` / `DisposableBean#destroy` / `destroy-method`** — shutdown callbacks (singleton only)

### Bean Scopes

| Scope | Instances | Lifecycle | Use Case |
|-------|-----------|-----------|----------|
| `singleton` (default) | 1 per container | Container lifetime | Stateless services, repos, controllers |
| `prototype` | New per request | No destroy callback | Stateful, short-lived objects |
| `request` | 1 per HTTP request | Request lifetime | Web: per-request state |
| `session` | 1 per HTTP session | Session lifetime | Web: user session state |
| `application` | 1 per ServletContext | App lifetime | Web: shared app state |

### `@PostConstruct` vs `InitializingBean`
- `@PostConstruct` — Jakarta annotation, preferred (no Spring coupling)
- `InitializingBean#afterPropertiesSet()` — Spring interface, avoid (couples to Spring)
- Both fire after all dependencies are injected

## 💻 Production Code Example

```java
@Service
@Slf4j
public class ConnectionPoolService implements DisposableBean {

    private final DataSourceProperties props;
    private HikariDataSource pool;

    public ConnectionPoolService(DataSourceProperties props) {
        this.props = props;
    }

    @PostConstruct
    public void init() {
        // Safe to use injected fields here
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(props.getUrl());
        config.setMaximumPoolSize(10);
        this.pool = new HikariDataSource(config);
        log.info("Connection pool initialized");
    }

    @PreDestroy
    public void shutdown() {
        if (pool != null && !pool.isClosed()) {
            pool.close();
            log.info("Connection pool closed");
        }
    }

    // DisposableBean fallback (same as @PreDestroy — pick one)
    @Override
    public void destroy() {
        shutdown();
    }
}

// Prototype scope — new instance per injection point
@Component
@Scope("prototype")
public class ReportGenerator {
    private final List<String> rows = new ArrayList<>();

    public void addRow(String row) { rows.add(row); }
    public String generate() { return String.join("\n", rows); }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Injecting a `prototype`-scoped bean into a `singleton` — the singleton caches the first prototype instance.
  **Fix:** Inject `ApplicationContext` and call `ctx.getBean(ReportGenerator.class)` each time, or use `@Lookup`.

- **Mistake:** Doing heavy work in the constructor instead of `@PostConstruct`.
  **Fix:** Constructors run before injection is complete. Use `@PostConstruct` to safely reference injected fields.

- **Mistake:** Expecting `@PreDestroy` to fire for `prototype` beans.
  **Fix:** Spring does not manage the destruction of prototype beans. Implement `Closeable` and call `destroy()` manually.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the default bean scope in Spring?
   - **A:** `singleton` — one instance per `ApplicationContext`.

2. **Q:** When does `@PostConstruct` fire?
   - **A:** After the constructor runs and all dependencies are injected, but before the bean is made available to the application.

3. **Q:** How do you inject a prototype bean into a singleton?
   - **A:** Use `@Lookup` (Spring overrides the method to call `getBean` each time), or use `ObjectProvider<T>`, or inject `ApplicationContext` and call `getBean()`.

4. **Q:** What is the difference between `@PreDestroy` and `DisposableBean`?
   - **A:** Both fire on container shutdown for singleton beans. `@PreDestroy` is a Jakarta standard annotation (preferred, no Spring coupling). `DisposableBean` is a Spring interface.

## 🔗 Related Notes
- [[IoC-Container-And-DI]]
- [[Component-Scanning-And-Stereotypes]]
