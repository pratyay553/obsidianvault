# Collection Framework Overview

## Definition

The **Java Collections Framework** is a unified architecture in `java.util` used to store and manipulate groups of objects. Think of it as a ready-made library that provides three things: **Interfaces** for the blueprint, **Implementations** for the actual data structures, and **Algorithms** for high-performance operations like sorting and searching.

- **Java Collections Framework (JCF)**: unified architecture (interfaces + implementations + algorithms) for storing and manipulating groups of objects.
- Root interfaces: `Collection<E>` (List/Set/Queue lineage) and `Map<K,V>` (separate hierarchy — not a `Collection`).
- Located in `java.util`; concurrent variants in `java.util.concurrent`.
### 1. The Collection Lineage

- **`Collection<E>`** is the root interface for standard groups of objects. It breaks down into three main sub-interfaces:
    
    - **`List`:** For ordered collections that allow duplicates (like `ArrayList`).
        
    - **`Set`:** For unordered collections that guarantee uniqueness (like `HashSet`).
        
    - **`Queue`:** For holding elements prior to processing, usually in a FIFO—First-In, First-Out—manner (like `LinkedList` or `PriorityQueue`).
        

### 2. The Map Lineage

- **`Map<K,V>`** is the outsider. It’s important to note that **`Map` does not extend the `Collection` interface**. It handles key-value pairs (like `HashMap`), where every key must be unique.
    

## ⚡ Thread Safety & Concurrency

"By default, standard collections aren't thread-safe. If I'm working in a multi-threaded environment, I look to the **`java.util.concurrent`** package for high-performance, thread-safe alternatives—like `ConcurrentHashMap` or `CopyOnWriteArrayList`—instead of using old, synchronized legacy classes like `Vector` or `Hashtable`."
## Why It Exists

## ⏳ The Problem: Before Java 1.2

> "Before Java 1.2, there was no unified collections framework. Developers had to use ad-hoc classes like `Vector`, `Hashtable`, or standard arrays."

This caused three major headaches:

- **Inconsistent APIs:** Every class had its own unique way of doing things. For example, to get the size of an array you used `.length`, for a Vector you used `.size()`, and for a Hashtable you used `.size()` but accessed elements differently.
    
- **No Common Interface:** Because they didn't share a common parent, you couldn't write generic code that worked across different data types.
    
- **No Reusable Algorithms:** If you wanted to sort a `Vector` and then sort an array, you had to write two entirely separate sorting algorithms.
    

## 🎯 The Solution: Goals of the Framework

Sun Microsystems designed the Java Collections Framework (JCF) to solve these exact problems. You can break their main goals down into four simple points:

### 1. Less Effort, More Standards

Instead of learning a brand-new API for every data structure, developers only need to learn a few standard interfaces (like `List`, `Set`, and `Map`). This makes code much easier to write and maintain.

### 2. High Performance Out of the Box

The framework provides highly optimized, battle-tested implementations of core data structures (like `ArrayList` and `HashSet`). Developers don't have to waste time reinventing the wheel or writing their own data structures from scratch.

### 3. Better Interoperability

Because all collections follow the same rules, unrelated libraries can seamlessly pass data to one another. A library written by Team A can easily accept a `List` created by Team B.

### 4. Code and Algorithm Reuse

The framework introduces standard utility classes—like `Collections` and `Arrays`—which contain powerful, ready-to-use algorithms for sorting, searching, and shuffling. This fosters massive code reuse across applications.

### 🎙️ Interviewer Check-in

> _"Essentially, JCF took a fragmented, messy system and replaced it with a clean, standardized blueprint that boosted both developer productivity and application performance."_

## Problem It Solves

- Eliminates reinventing dynamic arrays, linked lists, hash tables, trees, and queues per project.
- Provides polymorphic algorithms (`sort`, `binarySearch`, `shuffle`) independent of underlying implementation.
- Enables programming to interfaces (`List<T> l = new ArrayList<>()`), allowing implementation swap without client code change.

## Internal Working ⭐⭐⭐⭐⭐

### Hierarchy

```
                     Iterable<E>
                         |
                    Collection<E>
          ┌──────────────┼──────────────┐
        List<E>        Queue<E>       Set<E>
          |               |              |
  ArrayList        ┌──────┴──────┐   ┌───┴────┐
  LinkedList     Deque<E>    PriorityQueue  HashSet ── LinkedHashSet
  Vector/Stack   ArrayDeque                    |
                 LinkedList              SortedSet<E>
                                              |
                                        NavigableSet<E>
                                              |
                                          TreeSet

                     Map<K,V>  (NOT a Collection)
          ┌──────────────┼──────────────┐
      HashMap       LinkedHashMap    SortedMap<K,V>
        |                                  |
    Hashtable                       NavigableMap<K,V>
                                          |
                                       TreeMap
```

- `List`: ordered, index-based, duplicates allowed.
- `Set`: no duplicates, uniqueness enforced via `equals()`/`hashCode()` (Hash*) or `compareTo()`/`Comparator` (Tree*).
- `Queue`/`Deque`: FIFO / double-ended access, used for BFS, sliding window, scheduling.
- `Map`: key-value association; `HashMap`/`TreeMap`/`LinkedHashMap` internally back most `Set` implementations (`HashSet` = `HashMap` with dummy value).

### Backing Implementation Reality

|Interface impl|Actually backed by|
|---|---|
|`HashSet`|`HashMap<E, Object>` (dummy `PRESENT` value)|
|`LinkedHashSet`|`LinkedHashMap<E, Object>`|
|`TreeSet`|`TreeMap<E, Object>` (via `NavigableMap`)|
|`Stack`|extends `Vector` (legacy, synchronized)|
|`PriorityQueue`|binary heap over `Object[]`|
|`ArrayDeque`|circular resizable array|

### Design Decisions

- `Map` excluded from `Collection` because key-value pair semantics conflict with single-element `add(E)` contract — architects chose correctness over hierarchy uniformity.
- All core implementations are **fail-fast, not thread-safe** by default — performance prioritized; synchronization delegated to wrappers (`Collections.synchronizedList`) or `java.util.concurrent`.
- Growth/resizing strategies differ per structure specifically to amortize cost (see below) — O(1) amortized insertion is a framework-wide guarantee for array-backed structures.

## Internal Data Structure

|Implementation|Underlying structure|
|---|---|
|`ArrayList`|dynamic `Object[]` array|
|`LinkedList`|doubly linked list (`Node<E>` with prev/next)|
|`HashMap`|array of buckets (`Node<K,V>[]`), each bucket = linked list → red-black tree if size ≥ 8|
|`LinkedHashMap`|`HashMap` + doubly linked list threading insertion/access order|
|`TreeMap`|red-black tree (self-balancing BST)|
|`PriorityQueue`|binary min-heap in `Object[]`|
|`ArrayDeque`|circular `Object[]`, head/tail pointers|

## JVM Memory Representation

```
ArrayList (heap object):
┌───────────────────────────┐
│ ArrayList object          │
│  size: int                │
│  elementData: Object[] ───┼──► [e0][e1][e2][null][null]  (capacity > size)
│  modCount: int            │
└───────────────────────────┘

HashMap (heap object):
┌───────────────────────────┐
│ HashMap object             │
│  table: Node[] ────────────┼──► [0]->Node(k,v)->Node(k,v)
│  size: int                 │    [1]->null
│  threshold: int            │    [2]->TreeNode(...)  (treeified bucket)
│  loadFactor: float         │    ...
└───────────────────────────┘
```

- Every element reference in `elementData`/`table` is a **pointer**, not inline storage → poor cache locality for `LinkedList`/`HashMap` chains vs. contiguous `ArrayList`.
- Autoboxing: primitives stored in collections incur `Integer`/`Long` wrapper object overhead (~16 bytes header + value) plus pointer indirection.

## Step-by-Step Execution

**Example: `ArrayList` resize on overflow**

```
capacity = 10, size = 10, add(x) called
1. ensureCapacityInternal(size + 1) → 11 > 10
2. grow(): newCapacity = oldCapacity + (oldCapacity >> 1)  // 1.5x
3. Arrays.copyOf(elementData, newCapacity)  → O(n) copy
4. elementData[size++] = x
```

**Example: `HashMap.put` internal flow**

```
1. hash = key.hashCode() ^ (hash >>> 16)      // spread bits to reduce collisions
2. bucketIndex = hash & (table.length - 1)     // power-of-2 mask, faster than %
3. if bucket empty → insert new Node
4. else traverse chain / tree:
     - if key matches (== or equals) → overwrite value
     - else append to chain (or insert into tree if treeified)
5. if bucket size ≥ TREEIFY_THRESHOLD(8) and table.length ≥ 64 → treeify to Red-Black Tree
6. if size > threshold (capacity * loadFactor) → resize (double capacity, rehash all entries)
```

## ASCII Diagram

**HashMap bucket with collision + treeification:**

```
table[3] ──► [K1,V1] ──► [K2,V2] ──► [K3,V3]   (LinkedList, <8 entries)

after 8th collision AND table.length >= 64:

table[3] ──►      [K4,V4]
                  /       \
             [K2,V2]     [K7,V7]        (Red-Black Tree, O(log n) lookup)
             /    \
         [K1,V1] [K3,V3]
```

**ArrayDeque circular buffer:**

```
      head                     tail
       ▼                        ▼
[ _ | _ | E3 | E4 | E1 | E2 | _ | _ ]
             (wraps around when tail reaches array end)
```

## Complexity Analysis

|Structure|get/access|search/contains|insert|delete|Notes|
|---|---|---|---|---|---|
|`ArrayList`|O(1)|O(n)|O(1) amortized end, O(n) mid|O(n)|shifting on remove|
|`LinkedList`|O(n)|O(n)|O(1) at ends, O(n) mid (search cost)|O(1) once located|no random access|
|`HashMap`/`HashSet`|—|O(1) avg, O(log n) worst (treeified), O(n) worst pre-Java8-style|O(1) avg|O(1) avg|worst-case only under adversarial hashing|
|`TreeMap`/`TreeSet`|—|O(log n)|O(log n)|O(log n)|sorted order maintained|
|`LinkedHashMap`|—|O(1) avg|O(1) avg|O(1) avg|+ insertion/access order overhead|
|`PriorityQueue`|O(1) peek|O(n)|O(log n)|O(log n)|heap property only, not fully sorted|
|`ArrayDeque`|O(1) ends|O(n)|O(1) amortized both ends|O(1) both ends|faster than `LinkedList` for stack/queue use|

**Space overhead**: `ArrayList` ≈ 4 bytes/ref + unused capacity slack; `LinkedList` ≈ 24 bytes/node (2 pointers + object header) — ~3x heavier than `ArrayList`; `HashMap` node ≈ 32+ bytes (hash, key, value, next).

## Operations Table

|Operation|List|Set|Map|Queue/Deque|
|---|---|---|---|---|
|Add|`add(e)`|`add(e)`|`put(k,v)`|`offer(e)`/`push(e)`|
|Remove|`remove(idx/obj)`|`remove(obj)`|`remove(k)`|`poll()`/`pop()`|
|Lookup|`get(idx)`|`contains(obj)`|`get(k)`|`peek()`|
|Iterate order|insertion (List), sorted (via `Collections.sort`)|none (Hash), insertion (Linked), sorted (Tree)|same as Set variants|FIFO/LIFO|

## API Overview

- `Collection`: `add`, `remove`, `contains`, `size`, `isEmpty`, `iterator`, `stream`, `toArray`.
- `List`: + `get(int)`, `set(int,E)`, `indexOf`, `subList`, `ListIterator`.
- `Set`: no new methods — contract change only (uniqueness).
- `Map`: `put`, `get`, `getOrDefault`, `computeIfAbsent`, `merge`, `entrySet`, `keySet`, `values`.
- `Queue`: `offer`/`add`, `poll`/`remove`, `peek`/`element` (offer/poll return null/false on failure vs. throwing).
- `Deque`: `addFirst/Last`, `offerFirst/Last`, `pollFirst/Last`, `push`/`pop` (stack semantics).

## Important Methods

```java
// Java 21 idiomatic usage
Map<String, Integer> freq = new HashMap<>();
freq.merge("a", 1, Integer::sum);                 // atomic-style increment
freq.computeIfAbsent("b", k -> new ArrayList<>()); // lazy init pattern — but wrong return type here, illustrative only

List<Integer> nums = List.of(1, 2, 3);             // immutable factory (Java 9+)
var sorted = nums.stream().sorted().toList();       // Java 16+ toList()

Deque<Integer> stack = new ArrayDeque<>();          // preferred over java.util.Stack
stack.push(1); stack.pop();
```

## Implementation (Java 21)

**Custom generic Stack via `ArrayDeque` (interview-preferred pattern):**

```java
public final class BoundedStack<E> {
    private final Deque<E> data = new ArrayDeque<>();
    private final int capacity;

    public BoundedStack(int capacity) {
        this.capacity = capacity;
    }

    public void push(E item) {
        if (data.size() == capacity) {
            throw new IllegalStateException("Stack overflow");
        }
        data.push(item);
    }

    public E pop() {
        if (data.isEmpty()) {
            throw new NoSuchElementException("Stack empty");
        }
        return data.pop();
    }

    public E peek() {
        return data.peek();
    }
}
```

## Alternative Implementations

- `LinkedList` implements both `List` and `Deque` — rarely optimal for either; `ArrayList` beats it for List use, `ArrayDeque` beats it for Deque use (better cache locality, no per-node allocation).
- `Vector`/`Stack`/`Hashtable`: legacy, synchronized on every method call — avoid; use `ArrayList`/`ArrayDeque`/`HashMap` + explicit synchronization or `java.util.concurrent` equivalents.
- `CopyOnWriteArrayList`, `ConcurrentHashMap`, `ConcurrentLinkedQueue`: concurrent-package alternatives for multi-threaded access without full lock contention.

## Internal Optimizations

- **HashMap treeification** (Java 8+): converts long collision chains (≥8 nodes, table ≥64) to red-black trees, bounding worst-case lookup to O(log n) instead of O(n) — defends against hash-flooding DoS.
- **Power-of-2 table sizing**: enables `hash & (n-1)` instead of `hash % n` — bitmask faster than modulo.
- **Hash spreading** (`h ^ (h >>> 16)`): mixes high bits into low bits since table indexing only uses low bits — reduces collisions for poor `hashCode()` implementations.
- **ArrayList growth factor 1.5x** (vs `Vector`'s 2x): balances memory waste against resize frequency.
- **Immutable collection factories** (`List.of`, `Map.of`): return compact, allocation-optimized implementations (e.g., `ImmutableCollections.ListN`), zero mutation overhead.

## Advantages

- Interface-based programming → implementation swap without API change.
- Reusable, JVM-tuned algorithms (`Collections.sort` uses TimSort, O(n log n) guaranteed).
- Consistent iteration/fail-fast contracts across all structures.

## Disadvantages

- Type erasure limits primitive specialization → autoboxing overhead in generic collections.
- No unified “sorted list” with O(log n) insert — must use `TreeSet`/`TreeMap` and simulate.
- Fail-fast iterators throw at runtime, not compile-time — silent bugs possible under concurrent modification if undetected.

## Trade-offs

|Choice|Trade-off|
|---|---|
|`ArrayList` vs `LinkedList`|fast random access vs fast mid-insertion (rarely wins in practice)|
|`HashMap` vs `TreeMap`|O(1) avg lookup vs sorted iteration order|
|`HashSet` vs `LinkedHashSet`|memory/speed vs predictable iteration order|
|Synchronized wrapper vs `ConcurrentHashMap`|simplicity vs scalability under contention|

## Edge Cases

- Mutating a key object already inserted into a `HashMap`/`HashSet` (if `hashCode()` depends on mutable field) → element becomes unreachable ("lost" in wrong bucket).
- Null handling: `HashMap` allows one null key + multiple null values; `TreeMap` throws `NullPointerException` on null key (no natural ordering); `Hashtable`/`ConcurrentHashMap` disallow null entirely.
- `List.of()`/`Map.of()` throw `UnsupportedOperationException` on mutation attempts — common runtime surprise.
- Autoboxing cache: `Integer` values -128..127 are cached (`==` works), outside range requires `.equals()`.

## Common Pitfalls

- Using `==` instead of `.equals()` for object keys/elements.
- Forgetting to override both `equals()` and `hashCode()` together (contract violation breaks `HashMap`/`HashSet`).
- Modifying a collection while iterating with a for-each loop → `ConcurrentModificationException`; use `Iterator.remove()` or `removeIf()`.
- Using `Vector`/`Hashtable` in new code (synchronization overhead with no real benefit over modern alternatives).
- Assuming `HashMap` iteration order is insertion order (it isn't — use `LinkedHashMap`).

## Thread Safety

|Structure|Thread-safe?|Alternative|
|---|---|---|
|`ArrayList`, `HashMap`, `HashSet`, `ArrayDeque`|No|`Collections.synchronizedX()` or `CopyOnWriteArrayList` / `ConcurrentHashMap`|
|`Vector`, `Hashtable`|Yes (legacy, coarse lock)|prefer `java.util.concurrent`|
|`ConcurrentHashMap`|Yes (segment/bucket-level locking, Java 8+)|—|
|`CopyOnWriteArrayList`|Yes (copy-on-write, read-heavy optimized)|—|

## Fail-Fast Behavior

- Non-concurrent collections track `modCount`; iterators snapshot it at creation.
- Any structural modification outside the iterator increments `modCount`, causing next `next()`/`hasNext()` call to throw `ConcurrentModificationException`.
- **Best-effort only** — not guaranteed, not a synchronization mechanism. `java.util.concurrent` collections are **fail-safe** (weakly consistent iterators, no exception, may not reflect latest state).

## Spliterator / Stream Interaction

- `Collection.stream()` uses `Spliterator` for traversal + optional parallel decomposition (`trySplit()`).
- `ArrayList` spliterator: `SIZED`, `SUBSIZED`, `ORDERED` — splits efficiently (array range split) → good parallel stream candidate.
- `LinkedList`/`HashMap` spliterators split poorly (no O(1) midpoint access) → parallel streams less effective.
- `TreeMap`/`TreeSet` spliterators are `SORTED` + `ORDERED`.

## Comparison with Similar Structures

|Feature|ArrayList|LinkedList|HashMap|TreeMap|ArrayDeque|
|---|---|---|---|---|---|
|Random access|O(1)|O(n)|N/A|N/A|O(1) ends only|
|Ordering|insertion|insertion|none|sorted|insertion (FIFO/LIFO)|
|Null allowed|yes|yes|1 key + nulls|no key|no|
|Memory/element|low|high (node overhead)|medium|medium-high|low|

## DSA Usage ⭐⭐⭐⭐⭐

## Pattern Recognition

|Signal in problem|Structure to reach for|
|---|---|
|"frequency count", "seen before"|`HashMap`/`HashSet`|
|"top K", "Kth largest/smallest"|`PriorityQueue`|
|"sliding window", "next greater element"|`ArrayDeque` (monotonic deque/stack)|
|"sorted order maintained dynamically"|`TreeMap`/`TreeSet`|
|"BFS level order"|`ArrayDeque` as queue|
|"undo/backtrack", "valid parentheses"|`ArrayDeque` as stack|
|"LRU cache"|`LinkedHashMap` (access-order mode)|
|"insertion order + uniqueness"|`LinkedHashSet`|

## Common Interview Problems

1. **Two Sum** — brute O(n²) nested loop → optimal O(n) via `HashMap<value, index>` single pass.
2. **Top K Frequent Elements** — brute O(n log n) full sort → optimal O(n log k) via `PriorityQueue` bounded to size k.
3. **LRU Cache** — brute O(n) array scan per access → optimal O(1) via `LinkedHashMap(accessOrder=true)` or manual HashMap + doubly linked list.
4. **Group Anagrams** — `HashMap<sortedString, List<String>>`, O(n·k log k).
5. **Sliding Window Maximum** — brute O(n·k) recompute per window → optimal O(n) via monotonic `ArrayDeque` of indices.
6. **Valid Parentheses** — `ArrayDeque` as stack, O(n).

## Optimization Strategies

- Replace nested-loop lookups (O(n²)) with `HashMap` for O(1) membership checks → O(n).
- Replace full sort (O(n log n)) with bounded heap (`PriorityQueue`) when only top-K needed → O(n log k).
- Replace linear scan for min/max in window with monotonic deque to achieve amortized O(1) per element, O(n) total.
- Use `TreeMap.ceilingKey`/`floorKey` for O(log n) range queries instead of linear scan.

## Dry Run

**Two Sum, nums = [2,7,11,15], target = 9:**

```
i=0: map={}, complement=9-2=7, not in map → map={2:0}
i=1: map={2:0}, complement=9-7=2, found at index 0 → return [0,1]
```

## Interview Questions ⭐⭐⭐⭐⭐

## Frequently Asked Questions

- **Q: Why is `HashMap` not thread-safe but `ConcurrentHashMap` is, and how?** A: `ConcurrentHashMap` uses CAS operations + bucket-level (node) locking instead of a single global lock (Java 8+), unlike `Hashtable`'s method-level `synchronized`. Follow-up: explain why this avoids the "resize storm" problem under contention.
    
- **Q: Why does `HashMap` require both `equals()` and `hashCode()` overridden?** A: Contract: equal objects must have equal hash codes. Violating it causes duplicate "equal" keys to land in different buckets, breaking `get()`/`containsKey()`. Trap: interviewer may ask what happens if only `equals()` is overridden — answer: object still gets inconsistent bucket placement.
    
- **Q: Difference between `Comparable` and `Comparator`?** A: `Comparable.compareTo()` defines natural ordering (single strategy, implemented by the class itself); `Comparator.compare()` defines external, pluggable ordering (multiple strategies, no source modification needed). Follow-up: how does `TreeMap` use both — natural order default, or supplied `Comparator` at construction.
    
- **Q: Why is `ArrayList` growth 1.5x but `Vector` is 2x?** A: Design trade-off between resize frequency and wasted memory; `ArrayList` (non-synchronized, newer) tunes for lower memory footprint given more frequent expected use.
    
- **Q: How does `HashMap` resize (rehashing) work and what's its cost?** A: Doubles table size when `size > threshold`; every entry rehashed and redistributed — O(n) worst case per resize, but O(1) amortized over n insertions.
    

## Tricky Scenarios

- "Why did my custom object disappear from a `HashSet` after I changed a field used in `hashCode()`?" → mutable hash key placed in wrong bucket relative to its new hash; explain immutability requirement for hash keys.
- "Why does iterating and calling `list.remove(obj)` sometimes skip elements?" → index shifting after remove during manual index-based loop; use `Iterator.remove()` instead.
- "Is `HashMap` iteration order guaranteed across JVM runs?" → no, depends on hash function + bucket layout + insertion history; never rely on it.

## Real-World Backend Usage

- `ConcurrentHashMap` for in-memory caching layers (e.g., session store, config cache) under high concurrency.
- `LinkedHashMap` (access-order) for simple LRU cache implementation in a single-node service.
- `PriorityQueue` for task scheduling / job queues by priority.
- `TreeMap` for maintaining sorted rate-limiting windows or time-series buckets.
- `CopyOnWriteArrayList` for listener/observer registries with rare writes, frequent reads (event bus subscribers).

## Production Best Practices

- Pre-size collections (`new ArrayList<>(expectedSize)`, `new HashMap<>(expectedSize)`) to avoid resize churn in hot paths.
- Prefer immutable collections (`List.of`, `Map.copyOf`) for defensive copies / API return values.
- Never expose internal mutable collections directly from getters — return `Collections.unmodifiableList(...)` or a copy.
- Choose `ConcurrentHashMap` over `synchronizedMap` for read-heavy concurrent workloads — far better throughput.
- Avoid `Vector`/`Hashtable`/`Stack` in new code — legacy synchronization overhead with no upside.
- Watch for memory leaks: `HashMap` entries with keys that are never removed (e.g., using a growing `Map` as an unbounded cache) — use `WeakHashMap` or bounded eviction (`LinkedHashMap` + `removeEldestEntry`, or Caffeine/Guava cache).

## Cheat Sheet

```
Need fast lookup by key           → HashMap
Need sorted key iteration         → TreeMap
Need insertion-order map          → LinkedHashMap
Need unique elements, no order    → HashSet
Need unique + insertion order     → LinkedHashSet
Need unique + sorted order        → TreeSet
Need index-based fast access      → ArrayList
Need frequent head/tail ops       → ArrayDeque
Need priority-based extraction    → PriorityQueue
Need thread-safe map              → ConcurrentHashMap
Need thread-safe read-heavy list  → CopyOnWriteArrayList
```

## Quick Revision

- `Collection` ≠ `Map` (separate hierarchies).
- `HashSet`/`LinkedHashSet`/`TreeSet` are backed by their `Map` counterparts.
- `HashMap`: O(1) avg / O(log n) worst (treeified) / O(n) space; resizes by doubling; not thread-safe.
- Prefer `ArrayDeque` over `LinkedList`/`Stack` for stack/queue use.
- Fail-fast = best-effort `ConcurrentModificationException`; `java.util.concurrent` = fail-safe, weakly consistent.
- Always override `equals()` + `hashCode()` together for custom hash keys; keep hash keys immutable.