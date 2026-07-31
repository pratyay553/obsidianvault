---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Advanced
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Transaction Management & @Transactional

## 💡 1-Sentence Elevator Pitch (For Interviews)
> `@Transactional` is a Spring AOP proxy that wraps your method in a database transaction — if the method completes normally the transaction commits; if a `RuntimeException` is thrown it rolls back — and its `propagation` and `isolation` attributes control how it interacts with existing transactions.

## 🧱 Key Concepts & Mechanics

### How It Works (AOP Proxy)
1. Spring creates a CGLIB/JDK proxy around the `@Transactional` bean
2. On method entry, the proxy calls `TransactionManager.begin()`
3. Method body executes with the transaction bound to the current thread
4. On normal return → `commit()`; on `RuntimeException` → `rollback()`
5. Checked exceptions do **not** trigger rollback by default

### Propagation Types

| Propagation | Behaviour |
|-------------|----------|
| `REQUIRED` (default) | Join existing tx; create new if none |
| `REQUIRES_NEW` | Suspend existing tx; always start a new one |
| `NESTED` | Execute in a savepoint within the current tx |
| `SUPPORTS` | Join if exists; run without tx if not |
| `NOT_SUPPORTED` | Suspend existing tx; run without tx |
| `MANDATORY` | Must run within an existing tx; throw if not |
| `NEVER` | Must NOT run within a tx; throw if one exists |

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| `READ_UNCOMMITTED` | ✅ | ✅ | ✅ |
| `READ_COMMITTED` (default PG) | ❌ | ✅ | ✅ |
| `REPEATABLE_READ` | ❌ | ❌ | ✅ |
| `SERIALIZABLE` | ❌ | ❌ | ❌ |

### Rollback Rules
- Default: rollback on `RuntimeException` and `Error` only
- Checked exceptions → commit (historical Spring choice — override with `rollbackFor`)
- `rollbackFor = Exception.class` — rollback on any exception
- `noRollbackFor = InventoryException.class` — exclude specific types

### `readOnly = true`
- Hint to the persistence provider and connection pool
- Hibernate skips dirty checking, PostgreSQL uses read replica routing in some setups
- Use on all query-only service methods

## 💻 Production Code Example

```java
@Service
@Transactional(readOnly = true)   // default for all methods in this service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepo;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final AuditLogRepository auditRepo;

    public OrderDto findById(UUID id) {
        return orderRepo.findById(id)
            .map(OrderDto::from)
            .orElseThrow(() -> new ResourceNotFoundException("Order", id));
    }

    // ── Core write method ────────────────────────────────────────────
    @Transactional                          // overrides class-level readOnly
    @Retryable(retryFor = OptimisticLockingFailureException.class, maxAttempts = 3)
    public OrderDto placeOrder(PlaceOrderRequest req) {
        inventoryService.reserve(req.items()); // participates in same tx (REQUIRED)
        Order order = Order.create(req);
        orderRepo.save(order);
        paymentService.charge(order);          // must succeed or whole tx rolls back
        return OrderDto.from(order);
    }

    // ── REQUIRES_NEW: audit log must persist even if outer tx rolls back ──
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logEvent(UUID orderId, String event) {
        auditRepo.save(AuditLog.of(orderId, event));
    }

    // ── Checked exception: must explicitly opt into rollback ─────────
    @Transactional(rollbackFor = PaymentException.class)
    public void processRefund(UUID orderId) throws PaymentException {
        // rolls back on PaymentException (checked) and all RuntimeExceptions
        paymentService.refund(orderId);
        orderRepo.updateStatus(orderId, OrderStatus.REFUNDED);
    }

    // ── High-isolation: prevent phantom reads for inventory check ────
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public boolean canFulfil(UUID productId, int quantity) {
        int available = inventoryRepo.lockAndGetStock(productId); // SELECT FOR UPDATE
        return available >= quantity;
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Calling `@Transactional` methods from within the same class — the AOP proxy is bypassed, no transaction is created.
  **Fix:** Extract the method to a different Spring bean, or inject `self` via `ApplicationContext.getBean()`.

- **Mistake:** `@Transactional` on a `private` or `final` method — CGLIB cannot proxy these.
  **Fix:** Make transactional methods `public` and non-final.

- **Mistake:** Catching `RuntimeException` inside a `@Transactional` method without rethrowing — transaction commits despite the error.
  **Fix:** Let exceptions propagate, or call `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`.

- **Mistake:** Using `@Transactional` on the `@Repository` layer when it's already managed by the service — nested commits cause unexpected behavior.
  **Fix:** Put `@Transactional` on the `@Service` layer. Repositories should be REQUIRED participants, not transaction owners.

- **Mistake:** `REQUIRES_NEW` in the same thread as the outer transaction — the outer tx is suspended but shares the same DB connection pool. Can deadlock if pool size is too small.
  **Fix:** Ensure pool size > 1 + max nesting depth, or route `REQUIRES_NEW` calls to a separate service.

## ❓ Frequently Asked Interview Questions

1. **Q:** Why doesn't `@Transactional` work when called from the same class?
   - **A:** Spring transactions use AOP proxies — the proxy wraps the bean and intercepts external calls. When you call `this.method()`, you bypass the proxy and call the raw object directly — no transaction. The fix is to inject the bean into itself or extract the method to a separate Spring-managed class.

2. **Q:** What is the difference between `REQUIRED` and `REQUIRES_NEW`?
   - **A:** `REQUIRED` (default) joins an existing transaction if one is active, or creates a new one. `REQUIRES_NEW` always creates a new transaction, suspending any existing one — the inner transaction commits or rolls back independently of the outer one. Use `REQUIRES_NEW` for audit logs or outbox events that must persist even if the main operation rolls back.

3. **Q:** Does `@Transactional` roll back on checked exceptions?
   - **A:** No, by default Spring only rolls back on `RuntimeException` and `Error`. For checked exceptions, add `rollbackFor = MyCheckedException.class`.

4. **Q:** What does `readOnly = true` actually do?
   - **A:** It's a hint: Hibernate skips dirty-checking (no snapshot comparison at flush time), some JDBC drivers route to read replicas, and HikariCP may use a different connection pool. It doesn't enforce immutability — you can still accidentally save.

5. **Q:** What is optimistic locking and how does Spring/JPA support it?
   - **A:** Optimistic locking assumes conflicts are rare — no DB lock is held. JPA uses `@Version` (an integer or timestamp column). On update, JPA checks the version matches; if another transaction updated it first, it throws `OptimisticLockingFailureException`. Use `@Retryable` to retry the operation transparently.

## 🔗 Related Notes
- [[Spring-Data-JPA-Fundamentals]]
- [[Entity-Mapping-And-Relationships]]
- [[Database-Optimization-NPlus1-Indexing]]
