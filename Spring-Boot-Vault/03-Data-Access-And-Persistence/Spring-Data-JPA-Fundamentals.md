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

# 📌 Spring Data JPA Fundamentals

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Data JPA eliminates boilerplate DAO code by generating repository implementations at runtime from interface method signatures — you declare `JpaRepository<Entity, ID>` and Spring creates the SELECT, INSERT, UPDATE, DELETE implementations automatically.

## 🧱 Key Concepts & Mechanics

### Repository Hierarchy

| Interface | Adds |
|-----------|------|
| `Repository<T, ID>` | Marker only |
| `CrudRepository<T, ID>` | `save`, `findById`, `findAll`, `delete`, `count` |
| `PagingAndSortingRepository` | `findAll(Pageable)`, `findAll(Sort)` |
| `JpaRepository<T, ID>` | `flush`, `saveAndFlush`, `deleteInBatch`, `findAllById` |

Always extend `JpaRepository` for full JPA feature access.

### Derived Query Methods
- Method name is parsed into JPQL: `findByEmailAndStatus(String email, Status s)`
- Keywords: `By`, `And`, `Or`, `Not`, `Like`, `Between`, `In`, `OrderBy`, `Top`, `Distinct`

### Pagination & Sorting
- `Pageable pageable` parameter → `Page<T>` return (includes total count — extra `COUNT` query)
- `Pageable pageable` → `Slice<T>` (no total count — faster for infinite scroll)
- `Sort sort` → `List<T>` (no pagination)

### Projections
- **Interface projection:** define a getter interface — Spring creates a proxy returning only those columns
- **DTO projection:** use `@Query` with `new com.example.dto.UserSummary(u.id, u.name)`
- **Class-based (record projection):** works with constructor expressions in JPQL

### Specifications (Dynamic Queries)
- `JpaSpecificationExecutor<T>` + `Specification<T>` — type-safe dynamic predicates
- Compose with `spec1.and(spec2)`, `spec1.or(spec2)`

## 💻 Production Code Example

```java
// ── Entity ──────────────────────────────────────────────────────────
@Entity
@Table(name = "products")
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    private String name;
    private String category;
    @Enumerated(EnumType.STRING)
    private ProductStatus status;
    private BigDecimal price;
    // getters, setters, or Lombok @Data
}

// ── Repository ──────────────────────────────────────────────────────
public interface ProductRepository extends JpaRepository<Product, UUID>,
                                            JpaSpecificationExecutor<Product> {

    // Derived method
    List<Product> findByCategory(String category);

    // Pagination
    Page<Product> findByStatus(ProductStatus status, Pageable pageable);

    // Existence check (no entity loaded)
    boolean existsByName(String name);

    // Interface projection — only id + name columns fetched
    List<ProductSummary> findByCategory(String category, Class<ProductSummary> type);

    // Custom JPQL
    @Query("SELECT p FROM Product p WHERE p.price BETWEEN :min AND :max")
    List<Product> findByPriceRange(@Param("min") BigDecimal min,
                                   @Param("max") BigDecimal max);

    // Modifying query (requires @Transactional on caller)
    @Modifying
    @Query("UPDATE Product p SET p.status = :status WHERE p.category = :category")
    int updateStatusByCategory(@Param("status") ProductStatus status,
                                @Param("category") String category);
}

// ── Projection interface ─────────────────────────────────────────────
public interface ProductSummary {
    UUID getId();
    String getName();
}

// ── Service with pagination ──────────────────────────────────────────
@Service
@Transactional(readOnly = true)
public class ProductService {

    private final ProductRepository repo;

    public Page<ProductDto> list(ProductStatus status, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        return repo.findByStatus(status, pageable).map(ProductDto::from);
    }

    // Dynamic query with Specification
    public List<Product> search(String category, BigDecimal maxPrice) {
        Specification<Product> spec = Specification
            .where(CategorySpec.hasCategory(category))
            .and(PriceSpec.priceLessThan(maxPrice));
        return repo.findAll(spec);
    }
}

// ── Specification ────────────────────────────────────────────────────
public class CategorySpec {
    public static Specification<Product> hasCategory(String category) {
        return (root, query, cb) ->
            category == null ? cb.conjunction()
                             : cb.equal(root.get("category"), category);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `Page<T>` for infinite scroll — triggers an expensive `COUNT(*)` query on every page.
  **Fix:** Use `Slice<T>` — it only fetches the next batch without counting total records.

- **Mistake:** Calling a `@Modifying` query without `@Transactional` — throws `TransactionRequiredException`.
  **Fix:** Annotate the service method with `@Transactional` (not `readOnly = true`).

- **Mistake:** Loading full entities when only a few fields are needed.
  **Fix:** Use interface or DTO projections — Spring Data generates `SELECT id, name` instead of `SELECT *`.

- **Mistake:** Not adding `countQuery` to paginated `@Query` — Spring can't generate the count for `Page<T>`.
  **Fix:** Add `countQuery = "SELECT COUNT(p) FROM Product p WHERE ..."` to the `@Query`.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `Page` and `Slice` in Spring Data?
   - **A:** `Page` runs two queries — one for the data, one `COUNT(*)` for the total. `Slice` only fetches the current batch and checks if a next page exists. `Slice` is faster and is preferred for infinite scroll where total count is irrelevant.

2. **Q:** How does Spring Data generate query implementations at runtime?
   - **A:** Spring Data scans repository interfaces, parses method names using a keyword grammar (findBy, And, Or, Between, etc.), generates JPQL, and creates JDK proxy implementations backed by `SimpleJpaRepository` at application startup.

3. **Q:** What is `@Modifying` and when is it required?
   - **A:** `@Modifying` marks a `@Query` as an UPDATE or DELETE statement (not a SELECT). Without it, Spring Data treats the query as read-only and throws an exception. The calling method must also be `@Transactional`.

4. **Q:** When would you use Specifications vs derived query methods?
   - **A:** Derived methods work for static, known-at-compile-time queries. Specifications are for dynamic queries where filter combinations vary at runtime (e.g., a search endpoint with 10 optional filters).

## 🔗 Related Notes
- [[Entity-Mapping-And-Relationships]]
- [[JPQL-Native-Queries-Derived-Methods]]
- [[Transaction-Management-Transactional]]
