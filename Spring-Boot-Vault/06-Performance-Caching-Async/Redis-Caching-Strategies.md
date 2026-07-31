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

# 📌 Redis Caching Strategies

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring's `@Cacheable` annotation transparently caches method return values in Redis so repeated calls with the same arguments skip the expensive operation — and strategies like cache-aside, write-through, and TTL-based eviction control consistency vs. staleness tradeoffs.

## 🧱 Key Concepts & Mechanics

### Caching Annotations

| Annotation | Behaviour |
|------------|----------|
| `@Cacheable` | Return cached value if present; otherwise invoke method + cache result |
| `@CachePut` | Always invoke method + update cache (no read) |
| `@CacheEvict` | Evict entry or entire cache after method runs |
| `@Caching` | Compose multiple cache annotations on one method |
| `@EnableCaching` | Activates Spring's AOP caching infrastructure |

### Cache Key Strategies
- Default: all method parameters combined
- Custom SpEL: `@Cacheable(key = "#userId + ':' + #type")`
- Custom `KeyGenerator` bean for complex cases

### Caching Patterns

| Pattern | How | When |
|---------|-----|------|
| **Cache-Aside** | App checks cache, misses hit DB, app populates cache | Most common — full control |
| **Write-Through** | Write to cache + DB together | Strong consistency |
| **Write-Behind** | Write to cache, async flush to DB | High write throughput |
| **Refresh-Ahead** | Proactively refresh before TTL expires | Low-latency reads |

### TTL Strategies
- Short TTL (seconds–minutes): frequently changing data (inventory, prices)
- Long TTL (hours): slowly changing data (product catalog, user profiles)
- No TTL (`eternal`): lookup tables, rarely changes — evict on update only

### Cache Stampede Prevention
- Multiple requests miss simultaneously → all hit DB → DB overloaded
- Fix: use Redis `SET NX EX` lock while one request populates the cache
- Or: background refresh before TTL expiry (refresh-ahead)

## 💻 Production Code Example

```yaml
# application.yml
spring:
  cache:
    type: redis
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: 6379
      timeout: 2s
      lettuce:
        pool:
          max-active: 20
          max-idle: 10
```

```java
// ── Cache configuration with custom TTL per cache ─────────────────────
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerCustomizer() {
        return builder -> builder
            .withCacheConfiguration("products",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(30))
                    .disableCachingNullValues()
                    .serializeValuesWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer())))

            .withCacheConfiguration("user-sessions",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofHours(2)))

            .withCacheConfiguration("exchange-rates",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofSeconds(30)));
    }
}

// ── Service with caching ───────────────────────────────────────────────
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    private final ProductRepository repo;

    // Cache-Aside: read from cache first, populate on miss
    @Cacheable(cacheNames = "products", key = "#id", unless = "#result == null")
    public ProductDto findById(UUID id) {
        log.debug("Cache miss for product {}", id);
        return repo.findById(id).map(ProductDto::from)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
    }

    // Write-Through: update DB + refresh cache
    @CachePut(cacheNames = "products", key = "#result.id()")
    @Transactional
    public ProductDto update(UUID id, UpdateProductRequest req) {
        Product product = repo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product", id));
        product.update(req);
        return ProductDto.from(repo.save(product));
    }

    // Evict on delete
    @CacheEvict(cacheNames = "products", key = "#id")
    @Transactional
    public void delete(UUID id) {
        repo.deleteById(id);
    }

    // Evict all entries in a cache (use sparingly — blast eviction)
    @CacheEvict(cacheNames = "products", allEntries = true)
    public void invalidateAll() {}

    // Compose multiple cache operations
    @Caching(
        evict = { @CacheEvict(cacheNames = "products", key = "#id") },
        put   = { @CachePut(cacheNames = "products-v2", key = "#id") }
    )
    public ProductDto migrate(UUID id) { ... }
}

// ── Manual Redis operations (RedisTemplate) ────────────────────────────
@Service
@RequiredArgsConstructor
public class RateLimitService {

    private final StringRedisTemplate redis;

    public boolean isAllowed(String userId, int maxPerMinute) {
        String key = "rate:limit:" + userId;
        Long count = redis.opsForValue().increment(key);
        if (count == 1) {
            redis.expire(key, Duration.ofMinutes(1));  // set TTL on first call
        }
        return count <= maxPerMinute;
    }

    // Distributed lock — prevents cache stampede
    public Optional<ProductDto> getWithLock(UUID id) {
        String lockKey = "lock:product:" + id;
        Boolean acquired = redis.opsForValue()
            .setIfAbsent(lockKey, "1", Duration.ofSeconds(5));
        if (Boolean.TRUE.equals(acquired)) {
            try {
                ProductDto dto = loadFromDb(id);
                redis.opsForValue().set("product:" + id,
                    serialize(dto), Duration.ofMinutes(30));
                return Optional.of(dto);
            } finally {
                redis.delete(lockKey);
            }
        }
        // Another thread is loading — return stale or wait briefly
        return Optional.empty();
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Caching mutable objects — cached reference is shared and mutations affect the cache.
  **Fix:** Cache immutable DTOs (records) or serialize/deserialize to get value copies.

- **Mistake:** `@Cacheable` on a `private` or `final` method — Spring AOP can't proxy it.
  **Fix:** Annotate only `public` methods on Spring-managed beans.

- **Mistake:** Self-invocation — calling `this.findById()` from within the same class bypasses the cache proxy.
  **Fix:** Inject `self` or extract to a separate bean.

- **Mistake:** No TTL on cache entries — cache grows unbounded, stale data persists forever.
  **Fix:** Always set TTL appropriate to the data's change frequency.

- **Mistake:** Caching `Page<T>` or `Stream<T>` — these aren't serializable.
  **Fix:** Cache `List<DTO>` or plain POJOs. Reconstruct pagination from the cached list.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is cache-aside pattern and how does `@Cacheable` implement it?
   - **A:** Cache-aside: application checks cache first; on miss, loads from DB and populates the cache. `@Cacheable` implements this via AOP — on a cache hit it returns the cached value without calling the method; on a miss it invokes the method, caches the result, and returns it.

2. **Q:** What is a cache stampede and how do you prevent it?
   - **A:** When a popular cache entry expires and many concurrent requests simultaneously miss and hit the database. Prevention: distributed Redis lock (only one request rebuilds, others wait/use stale), refresh-ahead (proactively rebuild before TTL expires), or probabilistic early expiration.

3. **Q:** When would you use `@CachePut` instead of `@Cacheable`?
   - **A:** `@CachePut` always executes the method and updates the cache — use it on update operations to keep the cache consistent (write-through pattern). `@Cacheable` skips the method if a cache hit exists — use it only for read operations.

4. **Q:** How do you cache different data with different TTLs in the same app?
   - **A:** Use `RedisCacheManagerBuilderCustomizer` (or configure `spring.cache.redis.time-to-live` per cache name) to give each named cache its own `RedisCacheConfiguration` with an explicit `entryTtl`.

5. **Q:** What is the difference between Redis as a cache vs as a database?
   - **A:** As a cache, Redis holds data that can be regenerated from the primary store (DB) — eviction and expiry are acceptable. As a database, Redis is the source of truth — data must be durable (AOF/RDB persistence), replicated, and never silently evicted.

## 🔗 Related Notes
- [[Async-Processing-Async-CompletableFuture]]
- [[Database-Optimization-NPlus1-Indexing]]
- [[Refresh-Tokens-And-Session-Management]]
