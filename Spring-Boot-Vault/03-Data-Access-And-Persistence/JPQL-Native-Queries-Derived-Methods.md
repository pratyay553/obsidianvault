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

# 📌 JPQL, Native Queries & Derived Methods

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Data JPA gives you three query strategies — auto-generated derived methods from method names, JPQL `@Query` annotations for object-level queries, and native SQL `@Query(nativeQuery=true)` for database-specific features — each trading portability for power.

## 🧱 Key Concepts & Mechanics

### Query Strategy Comparison

| Strategy | Portability | Power | Best For |
|----------|------------|-------|----------|
| Derived methods | ✅ High | Limited | Simple lookups by 1-3 fields |
| JPQL `@Query` | ✅ High | Medium | Joins, aggregations on entity graph |
| Native `@Query` | ❌ DB-specific | Full SQL | DB functions, CTEs, bulk ops |
| Specifications | ✅ High | Dynamic | Runtime filter combinations |
| Criteria API | ✅ High | Full | Fully type-safe dynamic queries |

### JPQL vs SQL
- JPQL operates on **entity classes and fields**, not table/column names
- `FROM Product p` not `FROM products p`
- `p.orderItems` traverses the relationship — no explicit JOIN needed for simple nav
- Explicit JOIN: `JOIN p.orderItems oi WHERE oi.quantity > 1`
- **JOIN FETCH** — eagerly loads association in one query (fixes N+1)

### Named Parameters
- `@Param("name")` + `:name` in query string
- Positional `?1`, `?2` — avoid, breaks on refactoring

### Projections with @Query
- Interface projection: `SELECT p.id AS id, p.name AS name FROM ...` + interface with `getId()`, `getName()`
- Constructor expression: `SELECT NEW com.example.dto.ProductDto(p.id, p.name) FROM ...`
- Spring Boot 3: record-based projections work with interface projection syntax

## 💻 Production Code Example

```java
public interface OrderRepository extends JpaRepository<Order, UUID> {

    // ── Derived methods ──────────────────────────────────────────────
    List<Order> findByCustomerEmailAndStatus(String email, OrderStatus status);
    Optional<Order> findTopByCustomerIdOrderByCreatedAtDesc(UUID customerId);
    long countByStatus(OrderStatus status);
    boolean existsByCustomerIdAndStatus(UUID customerId, OrderStatus status);

    // ── JPQL with JOIN FETCH (prevents N+1) ──────────────────────────
    @Query("""
        SELECT DISTINCT o FROM Order o
        JOIN FETCH o.customer c
        JOIN FETCH o.items i
        JOIN FETCH i.product p
        WHERE o.status = :status
        ORDER BY o.createdAt DESC
        """)
    List<Order> findWithDetailsAndStatus(@Param("status") OrderStatus status);

    // ── JPQL aggregation ─────────────────────────────────────────────
    @Query("""
        SELECT o.status AS status, COUNT(o) AS count, SUM(o.total) AS total
        FROM Order o
        WHERE o.createdAt >= :since
        GROUP BY o.status
        """)
    List<OrderStatRow> getStatsSince(@Param("since") Instant since);

    // ── Pagination with custom JPQL (must supply countQuery) ─────────
    @Query(
        value = "SELECT o FROM Order o WHERE o.customerId = :id ORDER BY o.createdAt DESC",
        countQuery = "SELECT COUNT(o) FROM Order o WHERE o.customerId = :id"
    )
    Page<Order> findByCustomerId(@Param("id") UUID id, Pageable pageable);

    // ── Interface projection ─────────────────────────────────────────
    @Query("SELECT o.id AS id, o.total AS total, o.status AS status FROM Order o WHERE o.customerId = :id")
    List<OrderSummaryView> findSummariesByCustomer(@Param("id") UUID customerId);

    // ── Constructor projection (DTO) ─────────────────────────────────
    @Query("""
        SELECT NEW com.example.dto.OrderDto(o.id, o.total, o.status, c.name)
        FROM Order o JOIN o.customer c
        WHERE o.id = :id
        """)
    Optional<OrderDto> findDtoById(@Param("id") UUID id);

    // ── Native SQL (DB-specific function) ────────────────────────────
    @Query(
        value = """
            SELECT * FROM orders
            WHERE customer_id = :customerId
              AND created_at AT TIME ZONE 'UTC' >= NOW() - INTERVAL '30 days'
            ORDER BY created_at DESC
            LIMIT :limit
            """,
        nativeQuery = true
    )
    List<Order> findRecentNative(@Param("customerId") UUID id,
                                  @Param("limit") int limit);

    // ── Bulk update ──────────────────────────────────────────────────
    @Modifying
    @Query("UPDATE Order o SET o.status = 'CANCELLED' WHERE o.id IN :ids")
    int cancelOrders(@Param("ids") List<UUID> ids);
}

// ── Projection interfaces ────────────────────────────────────────────
public interface OrderSummaryView {
    UUID getId();
    BigDecimal getTotal();
    OrderStatus getStatus();
}

public interface OrderStatRow {
    OrderStatus getStatus();
    Long getCount();
    BigDecimal getTotal();
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Missing `countQuery` on paginated `@Query` — Spring can't auto-derive the count from complex JPQL.
  **Fix:** Always supply `countQuery` when using `@Query` with `Page<T>` return type.

- **Mistake:** Using `JOIN FETCH` in a paginated query — Hibernate warns and loads everything into memory for pagination.
  **Fix:** Separate the concern: first fetch IDs with pagination, then fetch with `JOIN FETCH` by ID list (two-query pattern).

- **Mistake:** `DISTINCT` missing from `JOIN FETCH` on collections — entity appears once per collection item.
  **Fix:** Add `SELECT DISTINCT o` when joining a collection association.

- **Mistake:** Native query returning `List<Order>` when the entity has lazy associations — triggers lazy loading per row.
  **Fix:** Use native queries only for scalar projections (not full entity hydration) or DTOs.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between JPQL and SQL?
   - **A:** JPQL is object-oriented — it queries entity classes and their fields/relationships, not database tables and columns. `FROM Order o JOIN o.items` traverses the Java object graph; JPA translates this to the correct SQL JOIN with the right foreign key. SQL queries tables directly.

2. **Q:** Why does `JOIN FETCH` with pagination cause a HibernateJpaDialect warning?
   - **A:** When fetching a collection with `JOIN FETCH`, the JOIN produces more rows than entities (one per collection item). Hibernate can't apply SQL `LIMIT`/`OFFSET` on this result because it would cut through entity boundaries. It falls back to loading all matching rows into memory and paginating there — a potential OOM for large datasets.

3. **Q:** When should you use native queries over JPQL?
   - **A:** When you need database-specific features unavailable in JPQL: window functions, CTEs (`WITH`), `JSONB` operators (Postgres), full-text search, `AT TIME ZONE`, `RETURNING`, bulk updates with complex expressions, or anything requiring a database-specific dialect.

4. **Q:** What is a DTO projection and why is it better than fetching full entities?
   - **A:** A DTO projection (`SELECT NEW ...` or interface projection) fetches only the requested columns instead of the full entity. This reduces data transfer, eliminates lazy-loading surprises, and avoids putting managed entities in the service/API layer where accidental modifications could be flushed to the database.

## 🔗 Related Notes
- [[Spring-Data-JPA-Fundamentals]]
- [[Entity-Mapping-And-Relationships]]
- [[Database-Optimization-NPlus1-Indexing]]
- [[Transaction-Management-Transactional]]
