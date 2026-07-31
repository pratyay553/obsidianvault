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

# 📌 Method-Level Security & @PreAuthorize

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Method-level security adds authorization checks directly on service methods using SpEL expressions in `@PreAuthorize` — so even if a controller is misconfigured, business logic is still protected at the service layer by Spring AOP.

## 🧱 Key Concepts & Mechanics

### Enabling Method Security
```java
@Configuration
@EnableMethodSecurity   // Spring Security 6+ (replaces @EnableGlobalMethodSecurity)
public class SecurityConfig { ... }
```

### Core Annotations

| Annotation | When | Description |
|------------|------|-------------|
| `@PreAuthorize` | Before method | Evaluates SpEL; blocks if false |
| `@PostAuthorize` | After method | Evaluates SpEL; can access return value via `returnObject` |
| `@PreFilter` | Before method | Filters a collection argument |
| `@PostFilter` | After method | Filters the returned collection |
| `@Secured` | Before method | Simple role-based — no SpEL |

### SpEL Expressions

| Expression | Meaning |
|------------|---------|
| `hasRole('ADMIN')` | Has `ROLE_ADMIN` authority |
| `hasAuthority('READ_ORDERS')` | Has exact authority string |
| `hasAnyRole('ADMIN','MANAGER')` | Has any of the listed roles |
| `isAuthenticated()` | Not anonymous |
| `#userId == authentication.name` | Parameter matches current user |
| `returnObject.ownerId == authentication.name` | Return value field check |
| `@myBean.check(#order)` | Delegate to a Spring bean method |

### Custom Authorization Beans
- Define a `@Component` with check methods, call it in SpEL as `@beanName.method(#param)`
- Keeps complex authorization logic testable and out of annotations

## 💻 Production Code Example

```java
// ── Enable ────────────────────────────────────────────────────────────
@Configuration
@EnableMethodSecurity
public class SecurityConfig { ... }

// ── Service-layer security ────────────────────────────────────────────
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository repo;
    private final OrderAuthChecker authChecker;  // custom bean

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
    public List<OrderDto> getOrdersForUser(String userId) {
        return repo.findByUserId(userId).stream().map(OrderDto::from).toList();
    }

    @PreAuthorize("isAuthenticated()")
    @PostAuthorize("returnObject.ownerId == authentication.name or hasRole('ADMIN')")
    public OrderDto findById(UUID id) {
        return repo.findById(id).map(OrderDto::from)
            .orElseThrow(() -> new ResourceNotFoundException("Order", id));
    }

    // Delegate to custom bean for complex logic
    @PreAuthorize("@orderAuthChecker.canModify(#id, authentication)")
    public OrderDto updateOrder(UUID id, UpdateOrderRequest req) {
        // ...
    }

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(UUID id) {
        repo.deleteById(id);
    }

    // Filter input collection — only process items the user owns
    @PreFilter("filterObject.ownerId == authentication.name")
    public void bulkCancel(List<Order> orders) {
        orders.forEach(o -> o.setStatus(OrderStatus.CANCELLED));
        repo.saveAll(orders);
    }
}

// ── Custom authorization component ────────────────────────────────────
@Component("orderAuthChecker")
@RequiredArgsConstructor
public class OrderAuthChecker {

    private final OrderRepository repo;

    public boolean canModify(UUID orderId, Authentication auth) {
        if (auth.getAuthorities().stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"))) {
            return true;
        }
        return repo.findById(orderId)
            .map(o -> o.getOwnerId().equals(auth.getName()))
            .orElse(false);
    }
}

// ── Testing method security ───────────────────────────────────────────
@SpringBootTest
class OrderServiceSecurityTest {

    @Autowired OrderService orderService;

    @Test
    @WithMockUser(username = "user1", roles = "USER")
    void userCanViewOwnOrders() {
        assertDoesNotThrow(() -> orderService.getOrdersForUser("user1"));
    }

    @Test
    @WithMockUser(username = "user2", roles = "USER")
    void userCannotViewOthersOrders() {
        assertThrows(AccessDeniedException.class,
            () -> orderService.getOrdersForUser("user1"));
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void adminCanViewAnyOrders() {
        assertDoesNotThrow(() -> orderService.getOrdersForUser("user1"));
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Forgetting `@EnableMethodSecurity` — all `@PreAuthorize` annotations are silently ignored.
  **Fix:** Add `@EnableMethodSecurity` to a `@Configuration` class.

- **Mistake:** `@PreAuthorize` on a `private` method — Spring AOP can't proxy private methods.
  **Fix:** Put security annotations only on `public` methods.

- **Mistake:** Putting authorization logic only in the controller — internal service calls bypass controller-layer checks.
  **Fix:** Annotate at the service layer where the business logic lives.

- **Mistake:** `hasRole('ROLE_ADMIN')` — double `ROLE_` prefix.
  **Fix:** `hasRole()` automatically prepends `ROLE_`. Use `hasRole('ADMIN')` or `hasAuthority('ROLE_ADMIN')`.

- **Mistake:** Calling a `@PreAuthorize`-annotated method from within the same bean — Spring AOP self-invocation bypass.
  **Fix:** Same as `@Transactional` — inject `self` or extract to a separate bean.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@PreAuthorize` and `@Secured`?
   - **A:** `@Secured` only supports simple role names — no SpEL. `@PreAuthorize` uses Spring Expression Language so you can express complex conditions: parameter-based checks, bean method calls, method return value inspection. Always prefer `@PreAuthorize`.

2. **Q:** How does `@PostAuthorize` differ from `@PreAuthorize`?
   - **A:** `@PreAuthorize` runs before the method — if it fails, the method is never called. `@PostAuthorize` runs after — the method executes, and then the return value is checked (accessible as `returnObject`). Use `@PostAuthorize` when you need to inspect the returned object to determine access.

3. **Q:** How do you test `@PreAuthorize`-secured methods?
   - **A:** Use `@WithMockUser(username = "alice", roles = "USER")` from `spring-security-test`. This populates the `SecurityContextHolder` with a mock authentication so Spring AOP evaluates the security expression as if a real user is logged in.

4. **Q:** Why put `@PreAuthorize` on the service layer instead of just the controller?
   - **A:** Controllers only secure HTTP entry points. If the service is called from a batch job, message consumer, internal admin endpoint, or another service, controller-level security is bypassed. Service-layer annotations protect the business logic regardless of how it's invoked.

## 🔗 Related Notes
- [[Spring-Security-Architecture-Filters]]
- [[Asymmetric-JWT-RS256-Implementation]]
- [[Unit-And-Integration-Testing-Testcontainers]]
