---
tags: [hld]
---

# Consistent Hashing

## 🧠 Problem
SAloon shards its cache across 4 Redis nodes using `hash(key) % 4`. Traffic grows and a 5th node is added. Now `hash(key) % 5` gives a *different* answer for almost every key — nearly the entire cache is suddenly "in the wrong place," and every server floods the DB with cache misses at once.

## 🤔 Why does the problem exist
Plain modulo hashing (`hash(key) % N`) ties every key's location to the *total number of nodes*, `N`. Changing `N` — by adding or removing even one node — changes the denominator for every single key, so nearly all keys remap to a different node simultaneously. That's the opposite of what you want when scaling should be incremental.

## 💡 Idea
**Consistent hashing** arranges both nodes and keys on a conceptual ring (a fixed hash space, e.g. 0 to 2³²−1). A key belongs to the first node found walking clockwise from the key's position. Adding or removing a node only affects the keys between it and its neighbor on the ring — everything else stays exactly where it was.

## 🏗️ Structure
```
              0
        ┌─────┴─────┐
   Node D           Node A
        │             │
   Node C           Node B
        └─────┬─────┘
           (ring, going clockwise)

Key "booking:42" hashes to a point between Node A and Node B → belongs to Node B.
Adding Node E between A and B only steals *some* of B's keys — C, D, A's keys untouched.
```

## ⚙️ Refinement: virtual nodes
Plain consistent hashing can still distribute keys unevenly if node positions land awkwardly on the ring. The fix: give each physical node many **virtual nodes** scattered around the ring (e.g. 100 points per physical server). More points per node smooths out the distribution and makes rebalancing on add/remove proportionally fairer.

## 🏢 SAloon Example
SAloon's Redis cache cluster uses consistent hashing with virtual nodes. When traffic grows and a 5th Redis node is added, only roughly 1/5th of the existing keys need to move (to the new node) — the other 4/5ths keep serving cache hits from the moment the new node joins, instead of the whole cluster taking a simultaneous cache-miss stampede against Postgres.

## 🎯 When should I use it?
Any time nodes in a distributed cache, database shard set, or load-balancer pool need to grow or shrink without a full remap of every key — distributed caches (Redis Cluster, Memcached), CDN routing, and sharded databases all use it.

## ⚠️ When NOT to use it?
Overkill for a fixed, small number of nodes that will basically never change — plain `hash % N` is simpler and fine if you genuinely never plan to resize the cluster.

## 🔥 Interview Question
*"Why does consistent hashing move fewer keys than modulo hashing when a node is added?"*
→ Modulo hashing's mapping depends on the total node count `N`, so changing `N` changes the mapping for essentially every key. Consistent hashing's mapping depends only on a key's fixed position on the ring relative to its neighbors — adding a node only intercepts the keys between the new node and the next one clockwise, leaving every other key's assignment untouched.

## 🔗 Related
[[02 - Load Balancing]]
[[05 - Database Replication and Sharding]]
