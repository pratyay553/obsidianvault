---
tags: [lld-problem]
---

# LRU Cache

## 🧠 Problem
Design a fixed-capacity cache that evicts the **Least Recently Used** entry when full, with O(1) `get` and `put`.

## 🤔 Why does the problem exist
A plain `HashMap` gives O(1) lookup but no ordering; a plain linked list gives ordering but O(n) lookup/removal. Neither alone hits O(1) for both operations — you need to combine them.

## 💡 Idea
Combine a `HashMap<Key, Node>` for O(1) lookup with a doubly linked list for O(1) reordering (move-to-front on access) and O(1) eviction (remove from tail).

## 🏗️ Structure
```
HashMap<K, Node<K,V>>   — O(1) lookup
Doubly Linked List      — head = most recently used, tail = least recently used
get(k)  → move node to head
put(k,v)→ insert at head; if over capacity, evict tail
```

## ☕ Java Implementation
```java
class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node> map = new HashMap<>();
    private final Node head = new Node(), tail = new Node(); // sentinels

    LRUCache(int capacity) { this.capacity = capacity; head.next = tail; tail.prev = head; }

    V get(K key) {
        Node n = map.get(key);
        if (n == null) return null;
        moveToHead(n);
        return (V) n.value;
    }

    void put(K key, V value) {
        Node n = map.get(key);
        if (n != null) { n.value = value; moveToHead(n); return; }
        if (map.size() == capacity) { Node lru = tail.prev; remove(lru); map.remove(lru.key); }
        Node fresh = new Node(key, value);
        map.put(key, fresh);
        addToHead(fresh);
    }
    // remove(), addToHead(), moveToHead() are standard O(1) doubly-linked-list ops
}
```

## 🏢 SAloon Tie-in
SAloon's availability cache (see [[../07 - Concurrency/02 - Locks]]) uses exactly this eviction policy so the in-memory cache doesn't grow unbounded across thousands of stylists/slots.

## Patterns Applied
This one is more data-structure than design-pattern, but it pairs directly with [[../07 - Concurrency/02 - Locks]] once made thread-safe (wrap `get`/`put` with a lock).

## 🔥 Interview Question
*"How would you make this thread-safe without killing performance?"*
→ A single `synchronized`/`ReentrantLock` around both `get` and `put` is the simplest correct answer (note: `get` in LRU is also a *write* since it reorders the list, so `ReadWriteLock` doesn't actually help here) — for higher throughput, sharding the cache into N independently-locked segments by key hash is the next step.

## 🔗 Related
[[../07 - Concurrency/02 - Locks]]

#lld-problem
