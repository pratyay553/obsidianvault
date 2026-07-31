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

# 📌 MongoDB Integration

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Data MongoDB maps Java classes to MongoDB documents with `@Document`, provides `MongoRepository` for common CRUD operations, and exposes `MongoTemplate` for complex aggregation pipelines — all backed by the reactive-capable MongoDB Java driver.

## 🧱 Key Concepts & Mechanics

### Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@Document(collection = "...")` | Marks class as a MongoDB document |
| `@Id` | Maps to MongoDB `_id` field |
| `@Field("field_name")` | Overrides default field name |
| `@Indexed` | Creates a single-field index |
| `@CompoundIndex` | Creates multi-field index |
| `@TextIndexed` | Enables text search on field |
| `@DBRef` | Reference to another document (lazy-loadable) — avoid; embed instead |
| `@Transient` | Excludes field from serialization |

### MongoRepository vs MongoTemplate

| | MongoRepository | MongoTemplate |
|-|----------------|---------------|
| Use for | CRUD, derived queries | Aggregations, complex updates, upserts |
| Return type | Entity objects | Flexible (Document, projection, scalar) |
| Learning curve | Low | Medium |

### Aggregation Pipeline
- `Aggregation.newAggregation()` → `$match`, `$group`, `$sort`, `$project`, `$lookup`
- `MongoTemplate.aggregate()` runs the pipeline
- Strongly prefer over multiple round trips

### Indexes
- Always define indexes for fields used in queries
- `@CompoundIndex` for queries filtering/sorting by multiple fields
- TTL index: `@Indexed(expireAfterSeconds = 3600)` — auto-deletes documents

## 💻 Production Code Example

```java
// ── Document class ──────────────────────────────────────────────────
@Document(collection = "events")
@CompoundIndex(name = "user_type_idx", def = "{'userId': 1, 'type': 1}")
public class UserEvent {

    @Id
    private String id;          // String → MongoDB ObjectId as string

    @Field("user_id")
    @Indexed
    private String userId;

    @Field("event_type")
    private String type;

    private Map<String, Object> payload;

    @Indexed(expireAfterSeconds = 7_776_000)  // 90-day TTL
    private Instant occurredAt;
}

// ── Repository ──────────────────────────────────────────────────────
public interface UserEventRepository extends MongoRepository<UserEvent, String> {

    List<UserEvent> findByUserIdAndTypeOrderByOccurredAtDesc(String userId, String type);

    // Pagination
    Page<UserEvent> findByUserId(String userId, Pageable pageable);

    // Custom query
    @Query("{ 'user_id': ?0, 'occurred_at': { $gte: ?1 } }")
    List<UserEvent> findRecentByUser(String userId, Instant since);
}

// ── Complex aggregation with MongoTemplate ──────────────────────────
@Service
@RequiredArgsConstructor
public class EventAnalyticsService {

    private final MongoTemplate mongoTemplate;

    public List<EventTypeSummary> summarizeEvents(String userId, Instant since) {
        Aggregation agg = Aggregation.newAggregation(
            Aggregation.match(
                Criteria.where("user_id").is(userId)
                        .and("occurred_at").gte(since)
            ),
            Aggregation.group("event_type")
                .count().as("count")
                .max("occurred_at").as("lastSeen"),
            Aggregation.sort(Sort.by(Sort.Direction.DESC, "count")),
            Aggregation.project("count", "lastSeen")
                .and("_id").as("eventType"),
            Aggregation.limit(10)
        );

        return mongoTemplate
            .aggregate(agg, "events", EventTypeSummary.class)
            .getMappedResults();
    }

    // Upsert — update or insert
    public void recordEvent(UserEvent event) {
        Query query = new Query(
            Criteria.where("user_id").is(event.getUserId())
                    .and("event_type").is(event.getType())
        );
        Update update = new Update()
            .set("occurred_at", event.getOccurredAt())
            .inc("count", 1)
            .setOnInsert("user_id", event.getUserId());
        mongoTemplate.upsert(query, update, UserEvent.class);
    }
}

public record EventTypeSummary(String eventType, long count, Instant lastSeen) {}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `@DBRef` for relationships — causes extra round trips (N+1 at the document level).
  **Fix:** Embed child documents directly in the parent when they're always read together. Store only IDs for loose references and resolve in the application layer.

- **Mistake:** No indexes on query fields — collection scans become catastrophic at scale.
  **Fix:** Add `@Indexed` or `@CompoundIndex` matching your query filter and sort patterns.

- **Mistake:** Using MongoDB for data that needs ACID transactions across multiple documents.
  **Fix:** MongoDB supports multi-document transactions (4.0+) but with overhead. If your domain requires strong consistency across entities, consider PostgreSQL.

- **Mistake:** Mapping `@Id` to a primitive `long` — MongoDB uses `ObjectId` by default; mismatches cause mapping errors.
  **Fix:** Use `String` for `@Id` and let MongoDB auto-generate ObjectIds, or use `ObjectId` type directly.

## ❓ Frequently Asked Interview Questions

1. **Q:** When would you choose MongoDB over PostgreSQL for a Spring Boot app?
   - **A:** MongoDB excels at storing variable-schema documents (event streams, user activity logs, product catalogs with varying attributes), high write-throughput scenarios, and when horizontal sharding is required. PostgreSQL is better for relational data, complex joins, and strict ACID requirements.

2. **Q:** What is the difference between `MongoRepository` and `MongoTemplate`?
   - **A:** `MongoRepository` is the high-level abstraction — derived queries, CRUD, pagination. `MongoTemplate` gives full control: aggregation pipelines, upserts, atomic field updates (`$inc`, `$push`), and operations that don't map cleanly to repository methods.

3. **Q:** How do you implement a TTL-based document expiry in Spring Data MongoDB?
   - **A:** Annotate a `Date` or `Instant` field with `@Indexed(expireAfterSeconds = N)`. MongoDB's background thread deletes documents where that field's value is older than N seconds.

4. **Q:** How does Spring Data MongoDB handle schema evolution?
   - **A:** MongoDB is schemaless so adding new fields to a document class is backward compatible. Removed fields are silently ignored on read. Renamed fields require a migration script or `@Field` alias mapping. Unlike JPA, there's no flyway equivalent — schema changes must be managed manually.

## 🔗 Related Notes
- [[Spring-Data-JPA-Fundamentals]]
- [[Database-Optimization-NPlus1-Indexing]]
- [[Transaction-Management-Transactional]]
