---
tags: [caching, redis, interview]
---

# Redis Caching

Related: [[09 - Kafka and RabbitMQ Messaging]] · [[01 - Spring Scheduling Basics]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Redis is an in-memory key-value store used here for two purposes visible in the reference sheet:
1. **Declarative caching** of expensive method results (`@Cacheable`/`@CacheEvict`)
2. **Scheduled population** of the cache from source-of-truth data (`RedisPopulate` scheduler class)

## 🔑 Core Pieces

| Component | Role |
|---|---|
| `@Cacheable` | Wraps a method — if the cache already has a result for these arguments, skip the method body and return the cached value |
| `@CacheEvict` | Removes an entry (or clears the whole cache) — used when underlying data changes |
| `RedisTemplate` | Low-level, manual Redis operations (get/set/expire, hash ops, etc.) |
| `Jedis` | A Java client library Spring can use underneath to actually talk to Redis over its wire protocol |

## 🛠️ Code

```java
@Service
public class FuelPriceService {

    @Cacheable(value = "fuelPrices", key = "#region")
    public FuelPrice getPrice(String region) {
        return fuelPriceClient.fetchLatest(region); // only called on cache miss
    }

    @CacheEvict(value = "fuelPrices", allEntries = true)
    public void clearCache() { }
}

@Component
public class RedisPopulate {

    private final RedisTemplate<String, Object> redisTemplate;

    @Scheduled(cron = "0 0 * * * *") // hourly
    public void populate() {
        List<Vehicle> vehicles = vehicleRepository.findAll();
        vehicles.forEach(v ->
            redisTemplate.opsForValue().set("vehicle:" + v.getId(), v, Duration.ofHours(2))
        );
    }
}
```

## 🧠 Why combine a *scheduler* with a cache?
Rather than caching lazily (only on first request), `RedisPopulate` **proactively refreshes** the cache on a schedule — useful when downstream consumers need consistently fast reads and can tolerate data being "as fresh as the last scheduled run" rather than perfectly real-time.

## ❓ Interview Q&A

**Q: `@Cacheable` vs manually using `RedisTemplate`?**
A: `@Cacheable` is declarative — Spring's caching abstraction handles the get-or-compute-and-store logic behind the scenes via a proxy (same AOP mechanism as [[04 - AOP - Aspect Oriented Programming]]). `RedisTemplate` is imperative — you write the get/set logic yourself, useful when you need custom key structures, TTLs per call, or data types `@Cacheable` doesn't handle well (e.g., hashes, sorted sets).

**Q: What's the risk with `@Cacheable` and stale data, and how do you mitigate it?**
A: The cache can return outdated results if the underlying data changes without an eviction. Mitigate with `@CacheEvict` on write operations, a sensible TTL, or (as this module does) a *scheduled refresh* job like `RedisPopulate` that keeps re-populating the cache proactively.

**Q: Why might a team choose to proactively populate a cache via a scheduler instead of relying purely on lazy caching?**
A: To avoid a "cache stampede" — many requests hitting a cold cache simultaneously all missing and hammering the DB/API at once. Pre-warming ensures the cache is always populated before traffic needs it.

**Q: What does `Jedis` do that `RedisTemplate` doesn't?**
A: `Jedis` is a low-level Redis Java client (handles the actual socket/protocol communication); `RedisTemplate` is Spring's higher-level abstraction that can use Jedis (or Lettuce) underneath, adding serialization, exception translation, and Spring-style templating.
