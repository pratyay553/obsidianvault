---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Beginner
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Component Scanning & Stereotypes

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Component scanning is how Spring auto-discovers beans by walking the classpath and registering every class annotated with `@Component` or one of its semantic aliases (`@Service`, `@Repository`, `@Controller`), eliminating the need for manual bean registration.

## 🧱 Key Concepts & Mechanics

### How Scanning Works
- `@SpringBootApplication` includes `@ComponentScan` scoped to the main class's package and all sub-packages.
- Spring uses `ClassPathBeanDefinitionScanner` to find annotated classes at startup.
- Custom scan packages: `@ComponentScan(basePackages = "com.example.core")`

### Stereotype Annotations

| Annotation | Extends | Extra Behaviour |
|------------|---------|-----------------|
| `@Component` | — | Generic bean |
| `@Service` | `@Component` | Semantic marker for business logic |
| `@Repository` | `@Component` | Wraps persistence exceptions into `DataAccessException` |
| `@Controller` | `@Component` | Registers with Spring MVC dispatcher |
| `@RestController` | `@Controller` + `@ResponseBody` | Returns JSON/XML directly |
| `@Configuration` | `@Component` | Declares `@Bean` methods; CGLIB-proxied |

### Include / Exclude Filters
```java
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Repository.class
    )
)
```

### `@Configuration` CGLIB Proxying
- `@Configuration` classes are subclassed by CGLIB at runtime.
- Calling `@Bean` methods directly from another `@Bean` method returns the **same singleton** — not a new instance.
- `@Configuration(proxyBeanMethods = false)` disables this for lighter-weight config.

## 💻 Production Code Example

```java
// Main class — scan starts here
@SpringBootApplication   // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Stereotypes in practice
@Repository
public class UserJpaRepository implements UserRepository {
    // Persistence exceptions auto-translated to DataAccessException
}

@Service
public class UserService {
    private final UserRepository repo;
    public UserService(UserRepository repo) { this.repo = repo; }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService svc;
    public UserController(UserService svc) { this.svc = svc; }
}

// Lightweight config — no CGLIB proxy needed
@Configuration(proxyBeanMethods = false)
public class WebClientConfig {
    @Bean
    public WebClient webClient() {
        return WebClient.builder().baseUrl("https://api.example.com").build();
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Placing the main class inside a sub-package (e.g., `com.example.web.Application`) — sibling packages like `com.example.data` are not scanned.
  **Fix:** Put `Application` in the root package (`com.example`).

- **Mistake:** `@Repository` on a class that doesn't use a Spring data abstraction — exception translation won't help if you're not using Spring's `PersistenceExceptionTranslationPostProcessor`.
  **Fix:** Use `@Repository` with Spring Data repos or JPA `EntityManager` usage.

- **Mistake:** Using `@Configuration` for a class that only has `@Bean` methods that don't call each other — unnecessary CGLIB overhead.
  **Fix:** Use `@Configuration(proxyBeanMethods = false)`.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@Component`, `@Service`, and `@Repository`?
   - **A:** Functionally identical for bean registration. The difference is semantic: `@Service` marks business logic, `@Repository` marks persistence and adds exception translation, `@Component` is the generic fallback.

2. **Q:** What does `@SpringBootApplication` do?
   - **A:** It's a composed annotation: `@SpringBootConfiguration` (alias for `@Configuration`) + `@EnableAutoConfiguration` + `@ComponentScan` rooted at the annotated class's package.

3. **Q:** How does `@Configuration` prevent duplicate bean creation when one `@Bean` method calls another?
   - **A:** CGLIB subclasses the `@Configuration` class and intercepts `@Bean` method calls, returning the cached singleton from the container instead of invoking the method body again.

## 🔗 Related Notes
- [[IoC-Container-And-DI]]
- [[Bean-Lifecycle-And-Scopes]]
- [[Configuration-And-Properties]]
