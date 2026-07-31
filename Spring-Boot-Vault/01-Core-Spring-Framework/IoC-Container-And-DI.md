---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Intermediate
status: Mastered
last_reviewed: 2026-07-23
---

# 📌 IoC Container & Dependency Injection

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring's IoC Container manages object creation and wiring — you declare *what* your class needs, and the container decides *how* to satisfy it, inverting the traditional control flow where objects create their own dependencies.

---

## 🧱 Key Concepts & Mechanics

### What Is Inversion of Control?
- **Traditional:** `UserService us = new UserService(new UserRepository())` — you control instantiation.
- **With IoC:** Spring creates and wires objects; your class just declares what it needs.
- The container reads annotation metadata to build a **dependency graph**, then resolves the whole graph at startup.

### The ApplicationContext
- The central IoC container interface. Spring Boot auto-selects the right implementation:
  - `AnnotationConfigServletWebServerApplicationContext` — standard web app
  - `AnnotationConfigReactiveWebServerApplicationContext` — WebFlux
- **Lifecycle:** instantiate → inject → `@PostConstruct` → ready → `@PreDestroy` → destroy

### Dependency Injection Types

| Type | Mechanism | Verdict |
|------|-----------|---------|
| **Constructor Injection** | `@Autowired` on constructor (implicit if single ctor) | ✅ Always prefer — immutable + testable |
| **Setter Injection** | `@Autowired` on setter method | Use only for optional/reconfigurable deps |
| **Field Injection** | `@Autowired` directly on field | ❌ Avoid — hides deps, blocks unit tests |

### Bean Registration Methods

| Method | When to Use |
|--------|-------------|
| `@Component` + component scan | Classes you own |
| `@Service` / `@Repository` / `@Controller` | Semantic aliases of `@Component` |
| `@Bean` inside `@Configuration` | Third-party classes or complex wiring |
| `@Import(OtherConfig.class)` | Modular config composition |

### Resolving Ambiguity (Multiple Matching Beans)
- `@Primary` — marks the default bean when multiple implementations exist
- `@Qualifier("beanName")` — selects a specific bean by name at injection point
- `@ConditionalOnProperty` — register bean only when a property is set (Boot)

---

## 💻 Production Code Example

```java
// ✅ Constructor Injection — the standard pattern
@Service
public class OrderService {

    private final PaymentGateway paymentGateway;
    private final InventoryClient inventoryClient;

    // Single constructor → Spring 4.3+ infers @Autowired automatically
    public OrderService(PaymentGateway paymentGateway,
                        InventoryClient inventoryClient) {
        this.paymentGateway = paymentGateway;
        this.inventoryClient = inventoryClient;
    }

    public OrderResult placeOrder(Order order) {
        inventoryClient.reserve(order.getItems());
        return paymentGateway.charge(order.getTotal());
    }
}

// ─────────────────────────────────────────────
// Multiple Implementations → @Qualifier
// ─────────────────────────────────────────────
public interface PaymentGateway {
    OrderResult charge(BigDecimal amount);
}

@Component("stripeGateway")
@Primary                            // used when no @Qualifier given
public class StripePaymentGateway implements PaymentGateway { ... }

@Component("paypalGateway")
public class PayPalPaymentGateway implements PaymentGateway { ... }

@Service
public class CheckoutService {
    private final PaymentGateway gateway;

    // Explicit selection overrides @Primary
    public CheckoutService(@Qualifier("paypalGateway") PaymentGateway gateway) {
        this.gateway = gateway;
    }
}

// ─────────────────────────────────────────────
// @Configuration + @Bean — third-party wiring
// ─────────────────────────────────────────────
@Configuration
public class InfraConfig {

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .connectTimeout(Duration.ofSeconds(5))
            .readTimeout(Duration.ofSeconds(10))
            .build();
    }

    @Bean
    @Primary
    public ObjectMapper objectMapper() {
        return JsonMapper.builder()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            .addModule(new JavaTimeModule())
            .build();
    }
}

// ─────────────────────────────────────────────
// Conditional bean registration
// ─────────────────────────────────────────────
@Configuration
public class CacheConfig {

    @Bean
    @ConditionalOnProperty(name = "cache.provider", havingValue = "redis")
    public CacheManager redisCacheManager(RedisConnectionFactory cf) {
        return RedisCacheManager.builder(cf).build();
    }

    @Bean
    @ConditionalOnProperty(name = "cache.provider", havingValue = "caffeine",
                           matchIfMissing = true)
    public CacheManager caffeineCacheManager() {
        return new CaffeineCacheManager();
    }
}
```

---

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Field injection (`@Autowired private Foo foo`).
  **Fix:** Use constructor injection. Field injection hides dependencies and requires a running Spring context for unit tests — you can't just `new MyService(mockFoo)`.

- **Mistake:** Circular dependency with constructor injection (A requires B, B requires A).
  **Fix:** Restructure to eliminate the cycle. If truly unavoidable, extract shared logic to a third component. Setter injection on one side can work, but it's a design smell.

- **Mistake:** `NoUniqueBeanDefinitionException` when two beans implement the same interface.
  **Fix:** Add `@Primary` to the default or `@Qualifier("name")` at the injection point. Don't leave ambiguity for Spring to guess.

- **Mistake:** Manually calling `new MyService()` inside Spring-managed code.
  **Fix:** Inject the bean. `new` bypasses the container — proxies (AOP, `@Transactional`, `@Cacheable`) won't fire.

- **Mistake:** Using `@Autowired` on a constructor that takes a mocked argument in tests.
  **Fix:** Constructor injection needs no mocking framework magic — just pass mock objects directly: `new MyService(mock(Dep.class))`.

- **Mistake:** Over-using `@Lazy` to hide startup problems.
  **Fix:** Use `@Lazy` only for genuinely expensive beans. Eager startup failures are easier to debug than lazy failures at request time.

---

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between IoC and DI?
   - **A:** IoC is the *principle* — control of object creation is inverted from the object to an external container. DI is the *mechanism* Spring uses to implement IoC — dependencies are injected into an object rather than the object creating them itself.

2. **Q:** Why is constructor injection preferred over field injection?
   - **A:** Constructor injection makes dependencies explicit (you can see them in the constructor signature), enforces immutability (`final` fields), and allows unit testing without a Spring context — `new OrderService(mockPayment, mockInventory)`. Field injection hides dependencies and requires reflection to populate them in tests.

3. **Q:** What happens if there are two beans of the same type and you `@Autowired` without a qualifier?
   - **A:** Spring throws `NoUniqueBeanDefinitionException`. Resolve with `@Primary` on the preferred bean, `@Qualifier("beanName")` at the injection point, or by injecting a `List<Interface>` to receive all implementations.

4. **Q:** What is `ApplicationContext` vs `BeanFactory`?
   - **A:** `BeanFactory` is the base IoC container — lazy, minimal. `ApplicationContext` extends it with: eager singleton initialization, event publishing (`ApplicationEventPublisher`), AOP support, `@PostConstruct`/`@PreDestroy`, environment/property abstraction, and i18n. Always use `ApplicationContext` in production code.

5. **Q:** How does Spring break circular dependencies for singletons?
   - **A:** Spring uses a **three-level cache** during singleton creation. When A is being created and needs B, which needs A, Spring injects an *early reference* (partially constructed proxy) of A into B. This only works for setter/field injection — constructor injection circular deps always fail fast at startup, which is correct behavior since they signal a design flaw.

6. **Q:** When would you use `@Bean` over `@Component`?
   - **A:** `@Bean` is required for classes you don't own (third-party libraries like `RestTemplate`, `ObjectMapper`, `DataSource`). It's also better when construction is complex or conditional. `@Component` (and its aliases) is for classes you own and can annotate directly.

7. **Q:** What does `@Lazy` do and when should you use it?
   - **A:** Defers bean instantiation until first requested, rather than at container startup. Useful for expensive beans with slow init, or for breaking `@Autowired`-constructor circular references. Use sparingly — lazy beans hide startup errors until the first HTTP request.

---

## 🔗 Related Notes
- [[Bean-Lifecycle-And-Scopes]] — what happens after injection
- [[Component-Scanning-And-Stereotypes]] — how beans are discovered
- [[Configuration-And-Properties]] — externalising config into beans
