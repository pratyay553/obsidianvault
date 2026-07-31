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

# 📌 Entity Mapping & Relationships

## 💡 1-Sentence Elevator Pitch (For Interviews)
> JPA entity mapping translates Java object graphs into relational tables using annotations like `@OneToMany` and `@ManyToOne`, with fetch type controlling whether related data is loaded eagerly (always) or lazily (on demand) — and choosing wrong causes either N+1 queries or unnecessary data loading.

## 🧱 Key Concepts & Mechanics

### Core Entity Annotations

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks class as a JPA-managed entity |
| `@Table(name = "...")` | Overrides default table name |
| `@Id` | Marks the primary key field |
| `@GeneratedValue` | Strategy: `IDENTITY`, `SEQUENCE`, `UUID`, `AUTO` |
| `@Column(nullable = false, length = 255)` | Column constraints |
| `@Enumerated(EnumType.STRING)` | Store enum as string (not ordinal) |
| `@Embedded` / `@Embeddable` | Map a value object into the same table |
| `@CreationTimestamp` / `@UpdateTimestamp` | Hibernate auto-timestamps |

### Relationship Types & Ownership

| Annotation | Table Side | Owner |
|------------|-----------|-------|
| `@ManyToOne` | Child table holds FK | Child (always owns) |
| `@OneToMany(mappedBy = "field")` | Parent side | Parent delegates to child |
| `@OneToOne` | Either side | Side without `mappedBy` |
| `@ManyToMany` | Join table | Either (define on one side) |

### Fetch Types (Defaults)

| Association | Default Fetch |
|-------------|--------------|
| `@ManyToOne` | `EAGER` |
| `@OneToOne` | `EAGER` |
| `@OneToMany` | `LAZY` |
| `@ManyToMany` | `LAZY` |

**Rule of thumb:** Always use `LAZY` on all associations. Load eagerly only via explicit JOIN FETCH in queries.

### Cascade Types
- `CascadeType.PERSIST` — save parent → save children
- `CascadeType.MERGE` — merge parent → merge children
- `CascadeType.REMOVE` — delete parent → delete children ⚠️ dangerous on `@ManyToMany`
- `CascadeType.ALL` — all of the above
- `orphanRemoval = true` — delete child when removed from parent's collection

## 💻 Production Code Example

```java
// ── Parent entity ─────────────────────────────────────────────────────
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @ManyToOne(fetch = FetchType.LAZY)      // override EAGER default
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    @OneToMany(mappedBy = "order",          // mappedBy = field name in OrderItem
               cascade = CascadeType.ALL,
               orphanRemoval = true,
               fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();

    @Embedded
    private ShippingAddress shippingAddress;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;

    @CreationTimestamp
    private Instant createdAt;

    // Convenience methods maintain bidirectional sync
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
    }
}

// ── Child entity ───────────────────────────────────────────────────────
@Entity
@Table(name = "order_items")
public class OrderItem {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;             // owns the FK

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;

    @Column(nullable = false)
    private int quantity;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal unitPrice;
}

// ── Embeddable value object ────────────────────────────────────────────
@Embeddable
public class ShippingAddress {
    private String street;
    private String city;
    @Column(name = "postal_code")
    private String postalCode;
    private String country;
}

// ── ManyToMany with join table metadata ───────────────────────────────
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `FetchType.EAGER` on `@ManyToOne` (the default) — every query loads the related entity, even when not needed.
  **Fix:** Set `fetch = FetchType.LAZY` on all associations. Fetch eagerly only in queries that explicitly need the data via `JOIN FETCH`.

- **Mistake:** `@Enumerated(EnumType.ORDINAL)` — enum ordinal shifts if enum order changes, corrupting data.
  **Fix:** Always use `@Enumerated(EnumType.STRING)`.

- **Mistake:** Not initializing collection fields — `items.add(...)` throws `NullPointerException`.
  **Fix:** Initialize: `private List<OrderItem> items = new ArrayList<>()`.

- **Mistake:** `CascadeType.REMOVE` on `@ManyToMany` — deletes all related entities shared with other parents.
  **Fix:** Never use `CASCADE_REMOVE` or `orphanRemoval` on `@ManyToMany`.

- **Mistake:** Bidirectional relationship not synchronized — adding to parent collection without setting the child's back-reference.
  **Fix:** Write convenience `addX()` / `removeX()` methods that set both sides.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@ManyToOne` and `@OneToMany`?
   - **A:** `@ManyToOne` lives on the child (FK owner) — many orders belong to one customer. `@OneToMany(mappedBy = "customer")` lives on the parent and references the field in the child that owns the FK. Always define the owning side with the FK column.

2. **Q:** What does `mappedBy` mean?
   - **A:** It tells JPA that the other side owns the foreign key. The side with `mappedBy` is the inverse — changes to it are not persisted unless the owning side is also updated.

3. **Q:** What is `orphanRemoval` and when would you use it?
   - **A:** `orphanRemoval = true` deletes a child entity from the database when it's removed from the parent's collection. Use it for owned child entities (e.g., `OrderItem` can't exist without an `Order`) but never on `@ManyToMany`.

4. **Q:** Why should you always use `LAZY` fetch type?
   - **A:** Eager loading always executes a JOIN regardless of whether the related data is needed, causing unnecessary database load. With lazy loading, the related entity is fetched only when accessed — and with `JOIN FETCH` in queries, you can still load it eagerly when you explicitly need it.

## 🔗 Related Notes
- [[Spring-Data-JPA-Fundamentals]]
- [[JPQL-Native-Queries-Derived-Methods]]
- [[Database-Optimization-NPlus1-Indexing]]
- [[Transaction-Management-Transactional]]
