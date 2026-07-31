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

# 📌 Database Optimization — N+1 & Indexing

## 💡 1-Sentence Elevator Pitch (For Interviews)
> The N+1 problem is when fetching N entities triggers N additional queries to load related data — fixed with `JOIN FETCH` or `@EntityGraph` — and database indexing ensures queries execute in O(log n) instead of O(n) by maintaining sorted B-tree structures on frequently queried columns.

## 🧱 Key Concepts & Mechanics

### The N+1 Problem
```
// Bad: 1 query for orders + N queries for each order's customer
List<Order> orders = orderRepo.findAll();   // SELECT * FROM orders   (1 query)
orders.forEach(o -> o.getCustomer().getName()); // SELECT * FROM customers WHERE id=? (×N)
// Total: 1 + N queries
```

### Solutions to N+1

| Solution | Mechanism | Best For |
|----------|-----------|---------|
| `JOIN FETCH` in JPQL | Single SQL JOIN loads both | Known associations needed upfront |
| `@EntityGraph` | Declare fetch plan on repository method | Reusable fetch policies |
| Batch Fetching (`@BatchSize`) | Hibernate loads N relations in one `IN` clause | Legacy code, optional associations |
| DTO Projection | Only fetch needed columns, no entity graph | Read-heavy endpoints |

### Hibernate `show_sql` for Diagnosis
```yaml
spring.jpa.show-sql: true
spring.jpa.properties.hibernate.format_sql: true
logging.level.org.hibernate.SQL: DEBUG
logging.level.org.hibernate.orm.jdbc.bind: TRACE   # log parameter values
```
Or use **Datasource Proxy** / **p6spy** for production-safe query logging.

### Index Types (PostgreSQL)

| Type | Use Case |
|------|---------|
| B-tree (default) | Equality, range queries, ORDER BY |
| Hash | Equality-only queries |
| GIN | Full-text search, JSONB, arrays |
| BRIN | Very large tables with naturally ordered data (timestamps) |
| Partial index | Index only a subset of rows (`WHERE status = 'ACTIVE'`) |
| Composite index | Multi-column queries — column order matters |

### Composite Index Column Order
- Rule: **equality columns first, then range/sort columns**
- `WHERE status = 'ACTIVE' AND created_at > ?` → index on `(status, created_at)`
- A query on `status` alone can still use this index (left-prefix rule)
- A query on `created_at` alone **cannot** use this index

### Explain Plan (PostgreSQL)
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) 
SELECT * FROM orders WHERE customer_id = $1 AND status = 'PENDING';
-- Look for: Seq Scan (bad on large tables) vs Index Scan (good)
```

## 💻 Production Code Example

```java
// ── JOIN FETCH — eliminates N+1 ────────────────────────────────────────
@Query("""
    SELECT DISTINCT o FROM Order o
    JOIN FETCH o.customer c
    JOIN FETCH o.items i
    JOIN FETCH i.product p
    WHERE o.status = :status
    """)
List<Order> findWithDetailsAndStatus(@Param("status") OrderStatus status);

// ── @EntityGraph — declarative fetch strategy ─────────────────────────
@EntityGraph(attributePaths = {"customer", "items", "items.product"})
@Query("SELECT o FROM Order o WHERE o.id = :id")
Optional<Order> findWithDetails(@Param("id") UUID id);

// Or define on the entity:
@Entity
@NamedEntityGraph(
    name = "Order.withCustomerAndItems",
    attributeNodes = {
        @NamedAttributeNode("customer"),
        @NamedAttributeNode(value = "items", subgraph = "items-subgraph")
    },
    subgraphs = @NamedSubgraph(
        name = "items-subgraph",
        attributeNodes = @NamedAttributeNode("product")
    )
)
public class Order { ... }

// ── @BatchSize — reduces N+1 to ceil(N/batchSize) queries ─────────────
@Entity
public class Order {
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    @BatchSize(size = 50)  // fetches 50 orders' items in one IN clause
    private List<OrderItem> items;
}

// ── DTO projection — skips entity hydration entirely ──────────────────
@Query("""
    SELECT new com.example.dto.OrderSummary(
        o.id, o.total, o.status, c.name, c.email)
    FROM Order o
    JOIN o.customer c
    WHERE o.status = :status
    """)
List<OrderSummary> findSummaries(@Param("status") OrderStatus status);

// ── Hibernate statistics for N+1 detection in tests ──────────────────
@SpringBootTest
class OrderRepositoryTest {

    @PersistenceUnit
    EntityManagerFactory emf;

    @Test
    void shouldFetchOrdersWithSingleQuery() {
        Statistics stats = emf.unwrap(SessionFactory.class).getStatistics();
        stats.setStatisticsEnabled(true);
        stats.clear();

        orderService.getOrdersWithDetails(OrderStatus.PENDING);

        assertThat(stats.getQueryExecutionCount())
            .as("Should execute only 1 query (JOIN FETCH)")
            .isEqualTo(1);
    }
}
```

```sql
-- ── Database Indexes ────────────────────────────────────────────────────

-- Single column index
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Composite index: equality first, range second
CREATE INDEX idx_orders_customer_status_date 
    ON orders(customer_id, status, created_at DESC);

-- Partial index: only active orders (smaller, faster for active queries)
CREATE INDEX idx_orders_active 
    ON orders(customer_id, created_at) 
    WHERE status IN ('PENDING', 'PROCESSING');

-- Covering index: includes columns needed by SELECT (avoids table heap access)
CREATE INDEX idx_orders_covering 
    ON orders(customer_id, status) 
    INCLUDE (total, created_at);

-- Full-text search index
CREATE INDEX idx_products_search 
    ON products USING GIN (to_tsvector('english', name || ' ' || description));

-- Check slow queries (pg_stat_statements)
SELECT query, calls, total_exec_time/calls AS avg_ms, rows
FROM pg_stat_statements
ORDER BY avg_ms DESC
LIMIT 20;
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `JOIN FETCH` in a paginated query — Hibernate loads everything into memory.
  **Fix:** Two-query pattern: first page by entity ID, then `JOIN FETCH` by the collected IDs.

- **Mistake:** Adding too many indexes — each index slows writes (INSERT/UPDATE/DELETE must update all indexes).
  **Fix:** Index only columns used in `WHERE`, `JOIN ON`, `ORDER BY`, or foreign keys. Measure write performance impact.

- **Mistake:** `SELECT *` in production — fetches unused columns, wastes network bandwidth, breaks column removal.
  **Fix:** Use DTO projections to select only needed columns.

- **Mistake:** Not having an index on foreign key columns — JOINs and cascade deletes scan the entire child table.
  **Fix:** Always index foreign key columns: `CREATE INDEX idx_order_items_order_id ON order_items(order_id)`.

- **Mistake:** Using `LIKE '%keyword%'` expecting it to use an index — leading wildcard prevents index use.
  **Fix:** Use PostgreSQL full-text search with a GIN index, or `pg_trgm` with a GIN trigram index.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the N+1 problem and how do you fix it?
   - **A:** When loading N entities triggers N additional queries to fetch related associations (lazy loading per entity). Fix with `JOIN FETCH` in JPQL (one SQL JOIN), `@EntityGraph` (declarative fetch plan), or `@BatchSize` (Hibernate loads N relations in one `IN` clause). Use `show_sql` or Hibernate statistics to detect it.

2. **Q:** What is a composite index and what is the left-prefix rule?
   - **A:** A composite index covers multiple columns in a specified order. The left-prefix rule: a query can use the index if its filter starts with the leftmost index columns. `(status, created_at)` helps queries filtering on `status` alone or `status + created_at`, but not `created_at` alone.

3. **Q:** What is a covering index?
   - **A:** An index that includes all columns needed by a query — the database can answer the query entirely from the index without accessing the table heap (Index-Only Scan). Add non-key columns with `INCLUDE (col1, col2)`.

4. **Q:** How do you detect N+1 in production?
   - **A:** Enable `pg_stat_statements` in PostgreSQL to find queries with high call counts and low average rows. In tests, enable Hibernate statistics and assert `getQueryExecutionCount() == 1`. In development, use `datasource-proxy` or `p6spy` to log every query with a stack trace.

5. **Q:** What is `@EntityGraph` and how does it differ from `JOIN FETCH`?
   - **A:** Both eagerly load associations in a single query, but `@EntityGraph` is more reusable — you declare which associations to load as metadata on the repository method without writing JPQL. `JOIN FETCH` is written inline in the JPQL query string. Use `@EntityGraph` when you want the same entity but with different fetch plans for different methods.

## 🔗 Related Notes
- [[Entity-Mapping-And-Relationships]]
- [[JPQL-Native-Queries-Derived-Methods]]
- [[Transaction-Management-Transactional]]
- [[Spring-Data-JPA-Fundamentals]]
