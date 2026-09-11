---
tags: [hld]
---

# Caching

## 🧠 Problem
SAloon's "view salon profile" page hits Postgres on every single request, even though a salon's profile changes maybe once a week. 90% of the DB's load is the same handful of queries run over and over.

## 🤔 Why does the problem exist
Databases are the slowest, most contended part of most systems — disk I/O, lock contention, and query planning all cost time. Re-running an identical, rarely-changing query for every request wastes that cost on data that hasn't moved.

## 💡 Idea
**Caching** stores the result of an expensive operation (a DB query, a computation, an API call) in a fast store — usually in-memory — so repeat requests for the same thing return instantly without redoing the work.

## 🏗️ Structure
```
Request ──▶ Check Cache ──▶ HIT: return cached value (fast)
                 │
                 └──▶ MISS: query DB ──▶ store result in cache ──▶ return value
```

## ⚙️ Key decisions
- **Where:** client-side (browser), CDN edge ([[04 - CDN]]), in-process (local memory), or a shared cache like Redis/Memcached. Shared caches are needed once you have multiple app servers, so every server sees the same cached value.
- **Eviction policy:** cache can't grow forever — LRU (Least Recently Used) is the default; LFU (Least Frequently Used) suits skewed access patterns.
- **Invalidation** — the genuinely hard part ("there are only two hard things in computer science: cache invalidation and naming things"):
  - **TTL (time-to-live):** value expires after N seconds regardless of correctness — simple, but can serve stale data.
  - **Write-through:** update cache and DB together on every write — always fresh, but every write pays the cache-write cost.
  - **Write-behind:** write to cache immediately, flush to DB asynchronously — fast writes, risk of data loss if the cache dies before flushing.
  - **Explicit invalidation:** delete/update the cache key the moment the underlying data changes.

## 🏢 SAloon Example
A salon's profile is cached in Redis with a 10-minute TTL, keyed by `salon:{id}`. When a salon owner edits their profile, `SalonService` explicitly deletes `salon:{id}` from Redis right after the DB write succeeds — so edits show up immediately instead of waiting out the TTL, while reads for unedited profiles stay cheap.

## 🎯 When should I use it?
Data that's read far more often than it's written, and where a little staleness is tolerable — profiles, product catalogs, computed aggregates, session lookups.

## ⚠️ When NOT to use it?
Don't cache data that must always be perfectly fresh and consistent — e.g. "is this slot still available right now" during checkout, where a stale cache hit could double-book a slot. That needs a real DB read (or a lock) at the moment of truth, not a cache.

## 🔥 Interview Question
*"What's the difference between write-through and write-behind caching?"*
→ Write-through updates the cache and the database synchronously on every write, guaranteeing consistency at the cost of write latency. Write-behind updates the cache immediately and writes to the database asynchronously later, giving faster writes but risking data loss if the cache fails before the flush happens.

## 🔗 Related
[[04 - CDN]]
[[06 - CAP Theorem and Consistency Patterns]]
[[00 - Introduction to HLD]]
