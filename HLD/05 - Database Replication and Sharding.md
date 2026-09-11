---
tags: [hld]
---

# Database Replication & Sharding

## 🧠 Problem
SAloon has one Postgres instance. Two separate problems show up as it grows: (1) it's a single point of failure — if it dies, the whole app is down — and (2) it's a single point of *capacity* — read and write load both funnel through one machine, and eventually one disk can't hold all the bookings in the country.

## 🤔 Why does the problem exist
A single database server has one copy of the data (no redundancy) and one machine's worth of CPU/disk/RAM (a capacity ceiling) — the exact same problem [[01 - Scalability]] describes for app servers, but harder here because *data* can't just be duplicated across servers the way stateless app logic can — copies must stay correct.

## 💡 Idea
Two different techniques, solving two different problems:
- **Replication** — copy the *entire* dataset onto multiple machines, to survive failure and to spread out *read* load.
- **Sharding (partitioning)** — split the dataset into pieces, each living on a different machine, to spread out *storage and write* load.

## 🏗️ Structure
```
Replication (same data, many copies)      Sharding (different data, many machines)
        Primary (writes)                   Shard 1: salons A–M
        /      \                           Shard 2: salons N–Z
  Replica 1   Replica 2 (reads)            Each shard: independent DB, own capacity
```

## ⚙️ How replication works
- **Primary-replica (master-slave):** writes go to the primary; the primary streams changes to replicas; reads can be served by any replica. Replication is usually *asynchronous* (fast writes, brief risk of replica lag) or *synchronous* (writes wait for replica confirmation — safer, slower).
- Failover: if the primary dies, a replica is promoted to primary. This needs a mechanism to detect the failure and avoid two primaries at once ("split brain").

## ⚙️ How sharding works
- Pick a **shard key** (e.g. `salon_id`, `user_id`) and a strategy to map keys to shards: range-based (A–M / N–Z), hash-based (`hash(key) % num_shards`), or directory-based (a lookup table of key → shard).
- The hard part: queries that need data from multiple shards (e.g. "top 10 salons nationwide") now require querying every shard and merging — this is why shard-key choice matters enormously, it should match your most common query pattern.
- Adding a new shard later means re-distributing keys — this is exactly the re-balancing problem [[07 - Consistent Hashing]] exists to minimize.

## 🏢 SAloon Example
SAloon shards bookings by `region_id` — a Mumbai booking and a Delhi booking live on entirely different DB instances, since almost every real query ("show today's slots") is naturally scoped to one region anyway. Within each regional shard, there's a primary (handles booking writes) and two read replicas (serve the much higher volume of "browse available slots" reads).

## 🎯 When should I use it?
Replication: as soon as you need either read-scaling or failure tolerance — almost always, even at modest scale. Sharding: once total data size or write throughput exceeds what one machine can hold/handle — a much higher bar, don't reach for it prematurely.

## ⚠️ When NOT to use it?
Don't shard before you need to — cross-shard queries, joins, and transactions get significantly harder, and re-sharding later is a painful migration. Exhaust vertical scaling and replication first.

## 🔥 Interview Question
*"Replication vs sharding — when would you use each?"*
→ Replication solves availability and read-scaling by copying the whole dataset; every replica can answer any read query. Sharding solves storage and write-scaling by splitting the dataset; each shard only knows its own slice of the data. Large systems typically use both: shard for scale, replicate each shard for durability and read throughput.

## 🔗 Related
[[01 - Scalability]]
[[06 - CAP Theorem and Consistency Patterns]]
[[07 - Consistent Hashing]]
