# Queue

## Part 1 — The Queue Interface Itself

### 1. Problem Before Invention

- Programs constantly need to process items **in the order they arrived**, not the order convenient to access — a print spooler, a task scheduler, a BFS traversal, a request buffer between a producer and a slower consumer.
- Before a `Queue` abstraction, developers reused `List` or raw arrays: `list.add(x)` to enqueue, `list.remove(0)` to dequeue.
- The mental model — "first in, first out" — had no dedicated type. Every developer re-derived the same discipline manually, with no compiler-enforced contract to prevent misuse (e.g., someone accidentally calling `list.remove(list.size()-1)` and silently turning a queue into a stack).

### 2. Why Previous Approaches Failed

```
ArrayList as a queue:
enqueue → list.add(x)          // O(1) amortized — fine
dequeue → list.remove(0)       // O(n) — must shift ALL remaining elements left

[A][B][C][D][E]  remove(0)
 ↓
[B][C][D][E]     ← every element shifted one slot left, O(n) per dequeue
```

- `ArrayList.remove(0)` is O(n) because deleting the head of a contiguous array forces a shift of every remaining element — a queue that dequeues n items this way costs **O(n²)** total, unacceptable for schedulers/BFS on large inputs.
- `LinkedList` as a raw `List` avoided the shift cost but exposed no queue-specific vocabulary — callers used `add()`/`remove(0)`, communicating nothing about intended FIFO discipline; nothing stopped `add(index, e)` from violating that discipline.
- No interface meant no polymorphism: code written against "a FIFO structure" couldn't be swapped between array-backed and heap-backed (priority) implementations without rewriting call sites.

### 3. Why Java Designers Created Queue

- Introduced in **Java 5** (JSR-166 area, alongside `java.util.concurrent`) specifically to:
    - Give FIFO (and priority-order) semantics a **first-class type**, separate from the general-purpose `List` contract.
    - Standardize a **non-throwing** API variant (`offer`/`poll`/`peek`) alongside the throwing variant (`add`/`remove`/`element`) — because "queue full" or "queue empty" are often _expected, recoverable_ conditions, not exceptional ones.
    - Provide a common supertype so `LinkedList`, `ArrayDeque`, `PriorityQueue`, and later the entire `java.util.concurrent` blocking-queue family could be used interchangeably by algorithms that only care about "produce/consume in order."

### 4. Internal Working (Contract Level)

```
Iterable
   │
Collection
   │
 Queue<E>
   │
 Deque<E>   (double-ended: queue + stack in one)
```

- `Queue` does **not** mandate a backing structure — it's a behavioral contract:
    - `offer(e)` — insert at tail, return `false` if capacity-bounded and full (non-throwing).
    - `poll()` — remove+return head, return `null` if empty (non-throwing).
    - `peek()` — return head without removing, `null` if empty.
    - `add(e)` / `remove()` / `element()` — throwing equivalents (from `Collection`), used when queue-full/empty is a programming error, not an expected state.
- `Deque<E>` extends `Queue<E>`, adding `addFirst/Last`, `offerFirst/Last`, `pollFirst/Last` — a `Deque` can simulate **both** a queue (FIFO) and a stack (LIFO) through one structure, which is why `ArrayDeque` deprecated the need for `Stack`.

### 5. Memory Layout (Abstract)

- `Queue` itself has none — it's an interface. Layout depends entirely on implementation (`ArrayDeque` = circular array, `LinkedList` = doubly linked nodes, `PriorityQueue` = binary heap array). Each is covered below.

### 6–7. Step-by-Step / Complexity — N/A at interface level (see implementations)

### 8. Hidden Implementation Details

- `poll()`/`peek()` returning `null` on empty creates ambiguity if the queue itself legitimately holds `null` elements — this is **why most `Queue` implementations (`ArrayDeque`, `PriorityQueue`) explicitly disallow `null` elements**, removing the ambiguity at the type level rather than documenting around it.
- `LinkedList` implements `Queue` too, but is rarely the right choice — `ArrayDeque` is faster for nearly every queue/stack workload (see Part 2).

### 9. Trade-offs

|Choice|Trade-off|
|---|---|
|Non-throwing (`offer`/`poll`) vs throwing (`add`/`remove`) API|recoverable-state ergonomics vs. fail-fast bug detection|
|Interface with no complexity guarantee|flexibility across array/linked/heap backing vs. caller must know the implementation to reason about performance|
|Disallowing `null`|removes `poll()` ambiguity vs. loses a "no value" sentinel developers might expect from `List` habits|

### 10. Alternatives

- `List` + manual discipline — rejected (O(n) head removal, no FIFO vocabulary).
- `Stack`/`Vector` — legacy, synchronized, wrong semantics (LIFO) for most queue use cases.
- `java.util.concurrent` blocking queues (`LinkedBlockingQueue`, `ArrayBlockingQueue`, `SynchronousQueue`) — for producer-consumer across threads; `Queue` itself is single-threaded, non-blocking.

### 11. Common Misconceptions

- "`Queue` is always FIFO." False — `PriorityQueue` implements `Queue` but orders by priority, not arrival time.
- "`poll()` throws if empty." False — `poll()`/`peek()` return `null`; `remove()`/`element()` throw. Interviewers deliberately test this pair.
- "`LinkedList` is the natural queue implementation because 'linked' sounds queue-like." False — `ArrayDeque` outperforms it in virtually every metric (see below).

### 12. Production Use Cases

- Task/job queues before handing off to a thread pool.
- BFS traversal frontier.
- Event buffering between producer and consumer stages of a pipeline (single-threaded case; multithreaded → `BlockingQueue`).

### 13. Interview Expectations

- Must explain FIFO contract, throwing vs. non-throwing methods, and that `Queue` doesn't fix an implementation.
- Must know `Deque` unifies queue+stack and is preferred over legacy `Stack`.
- Trap: asking "is `Queue` thread-safe?" — no, plain `Queue` implementations are not; only `java.util.concurrent` variants are.

---

## Part 2 — ArrayDeque (The Default Queue/Stack Implementation)

### 1. Problem Before Invention

- Need an implementation of `Queue`/`Deque` that supports **O(1) insertion and removal at both ends**, without the per-node memory overhead and pointer-chasing of a linked list.

### 2. Why Previous Approaches Failed

- `LinkedList` achieves O(1) at both ends too — so why replace it?
    - Every node is a **separate heap allocation** (object header ~16 bytes + 2 pointers ~16 bytes + payload) — for a queue of `Integer`, the node overhead can be 3–4x the actual data size.
    - Nodes are scattered across the heap → **poor cache locality**; each `next`/`prev` dereference is a potential cache miss, whereas array access strides through contiguous memory.
    - `Stack` (extends `Vector`) is synchronized on every call — needless lock overhead for single-threaded stack use.
- A plain (non-circular) array fails too: removing from the front requires shifting everything (same O(n) problem as `ArrayList.remove(0)`), unless you make it circular.

### 3. Why Java Designers Created ArrayDeque

- Introduced in **Java 6** as the **circular array** implementation of `Deque` — designed explicitly to replace both `Stack` (LIFO) and `LinkedList` (as `Queue`) with a single, faster, allocation-light structure.
- Official Javadoc explicitly recommends `ArrayDeque` over `Stack` when used as a stack, and over `LinkedList` when used as a queue.

### 4. Internal Working

- Backing store: `Object[] elements`, always kept at power-of-2 capacity (enables fast modulo via bitmask).
- Two cursors: `head` (index of first element) and `tail` (index of next free slot after last element).
- The array **wraps around** — "circular buffer" — so removing from the front never requires shifting; it just advances `head`.

### 5. Memory Layout

```
capacity = 8 (power of 2)

        head                  tail
         ▼                     ▼
elements: [ _ | _ | E1 | E2 | E3 | _ | _ | _ ]
                index 2   3    4

addFirst(E0):
        head             tail
         ▼                 ▼
elements: [ _ | E0 | E1 | E2 | E3 | _ | _ | _ ]
              index 1

After wraparound (tail exceeds array length, wraps to index 0):
        tail        head
         ▼            ▼
elements: [ E5 | E6 | _ | _ | E2 | E3 | E4 | _ ]
           0    1         4    5    6
```

- `head`/`tail` computed with bitmask: `newIndex = (index + 1) & (elements.length - 1)` — works only because capacity is a power of 2 (equivalent to `% capacity` but faster, no branch for negative results if handled correctly).

### 6. Step-by-Step Execution

**`offerLast(x)` (enqueue):**

```
1. elements[tail] = x
2. tail = (tail + 1) & (elements.length - 1)
3. if tail == head → array is full → resize (double capacity), re-layout elements linearly, reset head=0
```

**`pollFirst()` (dequeue):**

```
1. if head == tail → empty → return null
2. x = elements[head]
3. elements[head] = null              // avoid memory leak (GC can't reclaim referenced object)
4. head = (head + 1) & (elements.length - 1)
5. return x
```

**Resize (doubling), dry run:**

```
Before (full, capacity 8):
head=2                tail=2 (wrapped, full)
[E5|E6|_|_|E2|E3|E4|_]   → detect full when tail catches head after increment

Resize: allocate new array of capacity 16,
copy elements in logical order starting from head:
[E2|E3|E4|E5|E6|_|_|_|_|_|_|_|_|_|_|_]
head=0, tail=5
```

### 7. Complexity Derivation

|Operation|Complexity|Why|
|---|---|---|
|`addFirst`/`addLast`|O(1) amortized|pointer/index update only; resize is O(n) but amortizes to O(1) over n ops (doubling strategy — same amortized argument as `ArrayList`)|
|`pollFirst`/`pollLast`|O(1)|index update + nulling reference|
|`peekFirst`/`peekLast`|O(1)|direct array access|
|`get(index)` (not supported directly — no random access API on Deque, but internal access is O(1) via offset)|—|`ArrayDeque` deliberately doesn't expose index-based `get` — it's not a `List`|
|Iteration|O(n)|contiguous-ish array walk, excellent cache locality vs. LinkedList's pointer chasing|

**Amortized proof sketch (same as ArrayList):** each resize costs O(n) but doubles capacity, so resizes happen at sizes 1,2,4,8,...,n → total resize cost = O(n) (geometric series sums to ~2n) spread over n insertions = O(1) amortized per insertion.

### 8. Hidden Implementation Details

- Capacity is **always rounded up to the next power of 2** (e.g., request 10 → actual capacity 16) — silent memory overhead if not anticipated.
- `null` elements are **explicitly disallowed** — `addFirst(null)` throws `NullPointerException` — because `null` is used internally as the sentinel to detect "array not full" vs ambiguity with `poll()` returning `null` for "empty."
- Not thread-safe — no locking; concurrent structural modification produces undefined behavior (not even reliably fail-fast, unlike `ArrayList`/`LinkedList`).
- `iterator()` is fail-fast for structural modifications.

### 9. Trade-offs

|Aspect|ArrayDeque|LinkedList|
|---|---|---|
|Memory/element|low (contiguous array, no per-node overhead)|high (node object + 2 pointers)|
|Cache locality|good|poor|
|Random access|none (by design)|none (but O(n) via traversal)|
|Null elements|disallowed|allowed|
|Resize cost|occasional O(n), amortized O(1)|never (grows one node at a time)|

### 10. Alternatives

- `LinkedList` (as `Deque`) — only sensible if `null` elements are required, or extremely frequent mid-list mutation combined with queue-like access is needed (rare).
- `Stack`/`Vector` — legacy, avoid entirely.
- `ConcurrentLinkedDeque` — lock-free, thread-safe alternative for concurrent producer-consumer.

### 11. Common Misconceptions

- "ArrayDeque is a fixed-size array." False — it resizes (doubles) like `ArrayList`.
- "ArrayDeque can't be used as a queue, only a stack." False — it implements both `Deque` (stack via `push`/`pop`) and `Queue` (FIFO via `offer`/`poll`).
- "LinkedList is faster for queue operations since it's 'linked'." False in practice — allocation and cache-miss overhead usually make `ArrayDeque` faster despite both being O(1) asymptotically.

### 12. Production Use Cases

- BFS/level-order traversal frontier — `ArrayDeque` as queue.
- Monotonic stack/queue problems (sliding window maximum, next greater element).
- Undo/redo stacks, expression evaluation (stack via `push`/`pop`).
- Work-stealing / task deque patterns (single-threaded local queue before handing to executor).

### 13. Interview Expectations

- Must explain **why** ArrayDeque beats LinkedList (memory density, cache locality, no allocation per element) — not just "it's faster."
- Must know it disallows `null` and why (disambiguates `poll()`'s empty-signal).
- Must be able to derive amortized O(1) via the doubling argument, same reasoning as `ArrayList`.
- Trap: "Is `ArrayDeque` a `List`?" — No. It has no index-based `get`/`set` — it's a `Deque`, a different contract entirely, despite being array-backed.

---

## Part 3 — PriorityQueue (Order Beyond Arrival Time)

### 1. Problem Before Invention

- Many problems need "always process the currently smallest/largest/most urgent item next," not "first arrived, first processed" — Dijkstra's algorithm, task scheduling by priority, top-K streaming problems, event simulation (process events in timestamp order as they're discovered, not as they arrive).
- A FIFO queue cannot express this — the _n_-th arrival might need to be processed before the 1st.

### 2. Why Previous Approaches Failed

- **Sorting the whole collection every time a new element arrives**: O(n log n) per insertion — unacceptable for streaming/repeated extraction.
- **Keeping a sorted `List`/`TreeSet` and always taking the front**: insertion into a sorted array is O(n) (shifting); `TreeSet` gives O(log n) insert AND O(log n) min — sounds equivalent to a heap, but carries much heavier per-node memory overhead (red-black tree node: parent/left/right pointers + color bit) versus a heap's flat array, and doesn't support duplicates without extra wrapping (`Set` uniqueness).
- **Unsorted list + linear scan for min on each extraction**: O(1) insert, but O(n) extraction — bad when extraction is frequent (e.g., Dijkstra pops a vertex per iteration).

### 3. Why Java Designers Created PriorityQueue

- Introduced in **Java 5** specifically to provide **O(log n) insertion AND O(log n) extraction of the minimum (or max, via comparator)** using a **binary heap** — the theoretically optimal balance for repeated insert/extract-min workloads, with minimal memory overhead (backed by a plain array, no node objects).

### 4. Internal Working

- Backing structure: `Object[] queue`, representing a **complete binary tree** implicitly (no explicit child/parent pointers — computed via index arithmetic):
    - `parent(i) = (i - 1) / 2`
    - `left(i) = 2*i + 1`
    - `right(i) = 2*i + 2`
- **Min-heap property** (default, natural ordering or supplied `Comparator`): every parent ≤ both children — **not** a fully sorted structure, only the root is guaranteed to be the minimum.

### 5. Memory Layout

```
Logical tree:                Array representation:
        3
      /   \                  index: 0   1   2   3   4
     5     8                 value: 3 | 5 | 8 | 9 | 7
    / \
   9   7

parent(3) = (3-1)/2 = 1 → queue[1] = 5   ✓ (parent of index 3 is index 1)
left(0)   = 1             → queue[1] = 5 (left child of root)
right(0)  = 2             → queue[2] = 8 (right child of root)
```

### 6. Step-by-Step Execution

**`offer(x)` — insert with "sift-up" (bubble-up):**

```
Insert 2 into [3,5,8,9,7]:

1. Append at end (maintains "complete tree" shape):
   [3,5,8,9,7,2]         index 5, parent = (5-1)/2 = 2 → queue[2]=8

2. Compare with parent: 2 < 8 → swap
   [3,5,2,9,7,8]         new index 2, parent = (2-1)/2=0 → queue[0]=3

3. Compare with parent: 2 < 3 → swap
   [2,5,3,9,7,8]         new index 0 = root, stop (no parent)

Result: [2,5,3,9,7,8]  — heap property restored, O(log n) swaps (tree height)
```

**`poll()` — extract-min with "sift-down" (bubble-down):**

```
Extract from [2,5,3,9,7,8]:

1. Save root (2) as result to return
2. Move LAST element to root:
   [8,5,3,9,7]            (removed last, placed at index 0)

3. Sift down: compare index 0 (8) with children (5, 3) → pick smaller child (3, index 2)
   8 > 3 → swap
   [3,5,8,9,7]

4. Check index 2's children: none (out of bounds) → stop

Return 2, heap becomes [3,5,8,9,7]  — O(log n) swaps
```

### 7. Complexity Derivation

|Operation|Complexity|Derivation|
|---|---|---|
|`offer`|O(log n)|sift-up traverses at most tree height = ⌊log₂ n⌋ levels|
|`poll`|O(log n)|sift-down traverses at most tree height|
|`peek`|O(1)|root is always `queue[0]`|
|`contains`/`remove(Object)`|O(n)|must linear-scan (heap gives no shortcut for arbitrary element lookup — only root is known)|
|Build heap from n elements (`heapify`)|O(n), not O(n log n)|bottom-up sift-down: most nodes are near leaves and sift down only 0–1 levels; sum of work = Σ(n/2^h)·h converges to O(n), not the naive O(n log n) from n individual inserts|

**Space**: O(n) — flat array, no per-node pointer overhead, unlike `TreeMap`/`TreeSet`.

### 8. Hidden Implementation Details

- Only the **minimum** (root) is guaranteed sorted-position — iterating a `PriorityQueue` directly (`for (x : pq)`) does **NOT** yield sorted order; only repeated `poll()` does. This is the single most common interview/production bug.
- Growth strategy: capacity increases by 50% if small, or doubling behavior varies by JDK version internals — same amortized O(1) resize reasoning as `ArrayList`.
- `remove(Object)` and `contains` are O(n) — `PriorityQueue` is optimized for min-extraction, not membership testing; don't use it as a "sorted set."
- Not thread-safe; `PriorityBlockingQueue` is the concurrent equivalent.
- Ties: no guaranteed order among equal-priority elements (not stable).

### 9. Trade-offs

|Structure|Insert|Extract-min|Peek-min|Arbitrary lookup|Memory/element|
|---|---|---|---|---|---|
|`PriorityQueue` (heap)|O(log n)|O(log n)|O(1)|O(n)|low (array)|
|`TreeSet`/`TreeMap`|O(log n)|O(log n)|O(log n)|O(log n)|high (tree nodes)|
|Sorted `ArrayList`|O(n)|O(1) (remove end) or O(n) (remove front)|O(1)|O(log n) via binary search|low|
|Unsorted list|O(1)|O(n)|O(n)|O(n)|low|

- Heap wins when the workload is "insert + extract-min repeatedly," loses when "frequent arbitrary lookup/removal" or "need full sorted iteration" is required (use `TreeSet` instead).

### 10. Alternatives

- `TreeSet`/`TreeMap` — when full sorted-order traversal or O(log n) arbitrary removal/lookup is also needed.
- Indexed heap (custom, tracks element→index map) — when O(log n) `decreaseKey`/arbitrary removal is needed (classic Dijkstra optimization; Java's `PriorityQueue` doesn't support this natively).
- `PriorityBlockingQueue` — thread-safe priority queue for concurrent producer-consumer.

### 11. Common Misconceptions

- "Iterating a `PriorityQueue` gives sorted order." False — only guarantees heap property, not full order; must `poll()` repeatedly for sorted output.
- "`PriorityQueue` is a `Deque`/supports both ends." False — it only exposes head (min/max) access; no `addFirst`/`addLast`.
- "`remove(Object)` is fast because it's a heap." False — O(n), no index shortcut for arbitrary elements.

### 12. Production Use Cases

- Task schedulers ordering jobs by priority/deadline.
- Dijkstra's shortest path, Prim's MST — repeatedly extract-min vertex.
- Top-K streaming (bounded min-heap of size K) — merge K sorted streams/lists.
- Event-driven simulation — process events ordered by timestamp as discovered.

### 13. Interview Expectations

- Must be able to draw the array↔tree correspondence and derive parent/child index formulas from scratch.
- Must correctly perform sift-up/sift-down dry runs.
- Must explain why `heapify` is O(n) not O(n log n) — a frequently probed "gotcha."
- Must know: bounded top-K pattern uses a **min-heap of size K** (not max-heap) — counter-intuitive on first exposure; explain why (evict the smallest when a larger candidate arrives, keeping the K largest).
- Trap: "Can PriorityQueue efficiently support `decreaseKey`?" — No, not natively; requires a custom indexed heap; expected to know this limitation (relevant when Dijkstra with heap is discussed).

---

## Part 4 — Comparison Across Queue Family

|Implementation|Backing|Order|Insert|Extract head|Random access|Thread-safe|
|---|---|---|---|---|---|---|
|`ArrayDeque`|circular array|FIFO/LIFO (insertion)|O(1) amortized|O(1)|no|no|
|`LinkedList`|doubly linked nodes|FIFO/LIFO (insertion)|O(1)|O(1)|O(n)|no|
|`PriorityQueue`|binary heap array|priority order|O(log n)|O(log n)|no|no|
|`LinkedBlockingQueue`|linked nodes + locks|FIFO|O(1) (blocks if bounded+full)|O(1) (blocks if empty)|no|yes|
|`ArrayBlockingQueue`|fixed circular array + lock|FIFO|O(1) (blocks if full)|O(1) (blocks if empty)|no|yes|
|`PriorityBlockingQueue`|binary heap + lock|priority order|O(log n)|O(log n)|no|yes|

## Cheat Sheet

```
Need plain FIFO/LIFO, single-threaded    → ArrayDeque
Need FIFO with allowed nulls             → LinkedList (rare justification)
Need "always smallest/largest next"      → PriorityQueue (min-heap default)
Need top-K streaming                     → bounded min-heap (size K) via PriorityQueue
Need thread-safe producer-consumer FIFO  → LinkedBlockingQueue / ArrayBlockingQueue
Need thread-safe priority queue          → PriorityBlockingQueue
Need decreaseKey (Dijkstra optimization) → custom indexed heap, not java.util.PriorityQueue
```

## Quick Revision

- `Queue` = behavioral contract (FIFO or priority-order), not a performance guarantee — implementation decides complexity.
- `offer/poll/peek` (non-throwing) vs `add/remove/element` (throwing) — pick based on whether empty/full is expected or exceptional.
- `ArrayDeque`: circular array, O(1) amortized both ends, beats `LinkedList` on memory density and cache locality — default choice for stack/queue.
- `PriorityQueue`: binary heap in a flat array, O(log n) insert/extract-min, O(1) peek, O(n) arbitrary lookup — iteration ≠ sorted order.
- `heapify` from n elements is O(n), not O(n log n) — memorize the bottom-up sift-down argument.
- None of these are thread-safe by default — use `java.util.concurrent`'s blocking variants for cross-thread producer-consumer.
- Classic bug: assuming `PriorityQueue`'s iterator yields sorted output — it doesn't; only sequential `poll()` does.