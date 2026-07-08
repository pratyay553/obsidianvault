
# List Interface

## Definition

- `List<E>` — ordered `Collection` that permits duplicates, supports positional (index-based) access, and preserves insertion order.
- Sits between `Collection` (contract) and concrete implementations (`ArrayList`, `LinkedList`, `Vector`) — pure abstraction, zero implementation of its own.
- Package: `java.util`. Generic since Java 5.

## Why It Exists

- Pre-generics/pre-JCF Java had no standard ordered, index-accessible, duplicate-permitting container contract — only raw arrays (fixed size) and `Vector` (concrete, synchronized, no abstraction).
- `List` decouples **what** (ordered, indexed, duplicate-tolerant sequence) from **how** (array vs linked nodes) — enables programming to interface, swapping implementations without touching call sites.
- Provides a place for index-specific operations (`get(int)`, `set(int,E)`, `subList`) that don't belong on `Set` or `Queue`.

## Interface Hierarchy

```text
                 Iterable<E>
                     │
               Collection<E>
                     │
                  List<E>
          ┌──────────┼───────────┬───────────────┐
          │          │           │               │
      ArrayList  LinkedList   Vector      CopyOnWriteArrayList
                                  │
                                Stack
```

- `List extends Collection` — inherits `add`, `remove`, `size`, `iterator`, `stream`, etc., and **adds** index-based contract.
- `Collection extends Iterable` — mandates `iterator()`, enabling for-each loops on every `List`.
- `RandomAccess` is a separate **marker interface** (no methods) implemented by `ArrayList`, `Vector`, `CopyOnWriteArrayList` — NOT by `LinkedList`.

## Relationship with Collection / Iterable

|Layer|Contributes|
|---|---|
|`Iterable<E>`|`iterator()` — enables for-each; base for all traversal|
|`Collection<E>`|bulk ops: `add`, `remove`, `contains`, `size`, `stream`, `toArray`|
|`List<E>`|index ops: `get`, `set`, `add(index,e)`, `remove(index)`, `indexOf`, `listIterator`, `subList`|

- `List` overrides `Collection`'s contract for `equals()`/`hashCode()`: two lists are equal iff same elements in same order — `Set`/`Collection` in general has no such order-sensitive equality guarantee.

## Why List Is an Interface, Not a Class

- Same sequence abstraction has fundamentally different optimal backing structures — contiguous array (`ArrayList`) vs doubly-linked nodes (`LinkedList`) vs thread-safe variants (`CopyOnWriteArrayList`).
- A single concrete class cannot be simultaneously O(1) random-access AND O(1) arbitrary-position insert — interface lets each implementation optimize for its use case while sharing one contract.
- Enables **Dependency Inversion**: service/business logic depends on `List<E>` abstraction, not `ArrayList` concretion (Effective Java Item 64: "refer to objects by their interfaces").

## Why RandomAccess Is a Marker Interface

- Purpose: signal to generic algorithms (`Collections.binarySearch`, `Collections.shuffle`, `Collections.sort` pre-Java-8) that indexed `get(i)` is O(1), so an index-based loop is preferred over an iterator-based loop.
- Without it, algorithms would use iterator traversal even on `ArrayList`, or worse, use index loops on `LinkedList` → accidental O(n²).
- Marker interface (no methods) chosen over a method like `isRandomAccessSupported()` — instanceof check has zero runtime dispatch cost, resolved at JIT/compile-time branch.

```java
if (list instanceof RandomAccess) {
    for (int i = 0; i < list.size(); i++) process(list.get(i));   // O(n) total
} else {
    for (E e : list) process(e);                                   // avoid O(n²) on LinkedList
}
```

## Why Set/Queue Don't Support Indexed Access

- `Set`: no defined element order guarantee (hash-based) — "index 3" is meaningless/unstable across mutations.
- `Queue`: models restricted access (FIFO/priority) — exposing arbitrary index access would violate the queue discipline the interface is designed to enforce.
- `List` is the **only** JCF interface that models a true mathematical sequence with a well-defined position for every element.

## Characteristics

- **Ordered**: iteration order == insertion/positional order (unlike `HashSet`/`HashMap`).
- **Duplicates allowed**: no uniqueness constraint — `list.add(e)` always succeeds (capacity permitting).
- **Duplicates allowed because**: `List` models a sequence, not a mathematical set; real-world use cases (transaction logs, ordered event streams) require repeated values.
- **Insertion order preserved because**: positional semantics require deterministic index-to-element mapping; if order weren't preserved, `get(i)` would be undefined behavior.
- **Null handling**: most implementations (`ArrayList`, `LinkedList`) permit multiple `null` elements; `CopyOnWriteArrayList` also permits null; contract itself doesn't forbid null (implementation-specific).
- **Mutability**: `List` interface doesn't mandate mutability — `List.of()` returns an immutable `List` that throws `UnsupportedOperationException` on any mutator call.
- **Generic**: `List<E>` — type erasure at runtime; compile-time type safety only.

## Why the Interface Can't Guarantee Complexity

- `List` defines **behavioral contract** (what operations exist and their semantics), not performance contract.
- Complexity is a property of the **backing data structure**, chosen per-implementation — array (`ArrayList`) vs nodes (`LinkedList`).
- This is intentional: coupling complexity to the interface would force one-size-fits-all backing structure, defeating the purpose of having multiple implementations.

## Complexity Table

|Operation|List Interface|ArrayList|LinkedList|
|---|---|---|---|
|`add(e)` (append)|unspecified|O(1) amortized|O(1)|
|`add(index, e)`|unspecified|O(n) (shift)|O(n) (traverse) + O(1) link|
|`get(index)`|unspecified|O(1)|O(n)|
|`set(index, e)`|unspecified|O(1)|O(n)|
|`remove(index)`|unspecified|O(n) (shift)|O(n) (traverse)|
|`remove(Object)`|unspecified|O(n)|O(n)|
|`contains(e)`|unspecified|O(n)|O(n)|
|`iterator()` creation|O(1)|O(1)|O(1)|
|`iterator()` full traversal|unspecified|O(n), cache-friendly|O(n), pointer-chasing|
|`listIterator()`|unspecified|O(1) creation|O(n) if starting mid-list|

## Iterator vs ListIterator

|Aspect|`Iterator<E>`|`ListIterator<E>`|
|---|---|---|
|Traversal direction|forward only|forward + backward|
|Available on|any `Collection`|only `List`|
|Methods|`hasNext()`, `next()`, `remove()`|+ `hasPrevious()`, `previous()`, `set(E)`, `add(E)`, `nextIndex()`, `previousIndex()`|
|Element modification|remove only|remove, replace (`set`), insert (`add`)|
|Cursor concept|implicit, one-directional|explicit position between elements, bidirectional|
|Starting position|beginning only|any index via `listIterator(int)`|
|Use case|generic traversal/removal|in-place edit while iterating, reverse traversal|
|Complexity per step|O(1)|O(1)|
|Fail-fast|yes|yes|

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
ListIterator<Integer> it = nums.listIterator();
while (it.hasNext()) {
    int val = it.next();
    if (val % 2 == 0) it.set(val * 10);   // safe in-place modification
}
// nums = [1, 20, 3, 40]
```

- Why `Iterator` can't move backward: minimal contract by design — `Collection`-wide traversal must work for structures with no meaningful "previous" (e.g., a conceptual `Set` bucket walk); adding `previous()` to `Iterator` would force implementations that can't support it.
- Why `ListIterator` exists only for `List`: bidirectional + index-aware traversal requires positional semantics that only sequences provide.
- Why Streams don't support backward traversal: `Stream` models a pipeline of transformations over a source, potentially lazy/parallel/infinite — a "position" concept is incompatible with parallel decomposition (`Spliterator.trySplit()`).

## Fail-Fast Behavior

- Every structural modification (`add`/`remove`, not `set`) increments `modCount`.
- `Iterator`/`ListIterator` capture `expectedModCount` at creation; each `next()` call checks `modCount == expectedModCount`, else throws `ConcurrentModificationException`.
- Best-effort only — not a synchronization guarantee, not always triggered (e.g., benign races may go undetected).
- `Iterator.remove()` / `ListIterator.set()`/`add()` update `expectedModCount` internally — safe way to mutate during iteration.

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
for (int x : list) {
    if (x == 2) list.remove(Integer.valueOf(x));   // throws ConcurrentModificationException
}
// Correct:
list.removeIf(x -> x == 2);   // or use Iterator.remove()
```

## Important Methods

|Method|Purpose|Complexity (ArrayList / LinkedList)|Hidden Cost|Interview Note|
|---|---|---|---|---|
|`add(E)`|append|O(1) amortized / O(1)|array resize (copy) on overflow|ask: what's the growth factor? (1.5x)|
|`add(int, E)`|insert at index|O(n) / O(n)|shift elements (array) or full traversal (linked)|LinkedList isn't automatically faster here|
|`get(int)`|positional read|O(1) / O(n)|LinkedList must walk from nearer end|classic trap: index loop on LinkedList = O(n²)|
|`set(int, E)`|positional overwrite|O(1) / O(n)|no resize; no structural mod (no `modCount` bump)|doesn't invalidate iterators|
|`remove(int)`|remove by index|O(n) / O(n)|array: shift left; linked: traverse + unlink|ambiguity: `remove(int)` vs `remove(Object)`|
|`remove(Object)`|remove by value|O(n) / O(n)|linear scan via `equals()`|autoboxing trap: `list.remove(2)` vs `list.remove(Integer.valueOf(2))`|
|`contains(Object)`|membership test|O(n) / O(n)|linear scan, no shortcut|O(n) — don't confuse with `HashSet`'s O(1)|
|`containsAll(Collection)`|subset check|O(n·m)|nested scans unless backed by hash structure|expensive for large inputs|
|`addAll(Collection)`|bulk append|O(m) amortized|may trigger single resize for whole batch|more efficient than looped `add`|
|`removeAll(Collection)`|bulk remove|O(n·m) or O(n) with hash-backed arg|pass a `HashSet` as arg to get O(n+m)|interviewer tests Big-O awareness|
|`retainAll(Collection)`|intersection in place|O(n·m) or O(n) with hash-backed arg|same optimization as `removeAll`||
|`clear()`|empty the list|O(n) (ArrayList nulls refs for GC) / O(1) conceptually|ArrayList nulls out entries to avoid memory leak (loitering objects)|why not just `size = 0`? — GC reachability|
|`indexOf(Object)`|first occurrence|O(n)|linear scan|returns -1 if absent|
|`lastIndexOf(Object)`|last occurrence|O(n)|scans from end for ArrayList; full scan needed for LinkedList||
|`isEmpty()`|size check|O(1)|—||
|`size()`|element count|O(1) (cached field, not computed)|—|LinkedList maintains `size` field too — no O(n) count|
|`iterator()`|forward-only cursor|O(1) creation|—||
|`listIterator()`|bidirectional cursor|O(1) (ArrayList) / O(n) if non-zero start index (LinkedList)|LinkedList must walk to starting position||
|`subList(from, to)`|view, not copy|O(1) creation|**structural modification of sublist reflects in parent and vice versa**; can throw CME|classic trap — mutating sublist after unrelated parent mutation|
|`replaceAll(UnaryOperator)`|in-place transform|O(n)|avoids creating new list|Java 8+|
|`sort(Comparator)`|in-place sort|O(n log n) (TimSort)|ArrayList sorts array directly; LinkedList: dumps to array, sorts, relinks|default interface method since Java 8|
|`toArray()`|snapshot copy|O(n)|defensive copy, safe to mutate||
|`stream()`|sequential stream|O(1) (lazy)|actual O(n) only on terminal op||
|`parallelStream()`|parallel stream|O(1) (lazy)|ArrayList splits well (RandomAccess); LinkedList splits poorly|interviewer probes: why is ArrayList better for parallelStream?|

## ASCII Diagram

**ArrayList memory layout:**

```
ArrayList<E> object
┌─────────────────────────┐
│ size: 3                 │
│ elementData: Object[] ──┼──► [e0][e1][e2][null][null]   (capacity=5)
└─────────────────────────┘
get(1) → elementData[1]   // O(1) direct offset calculation
```

**LinkedList memory layout:**

```
LinkedList<E> object
┌───────────────────────┐
│ first ──┐   last ──┐   │
│ size: 3 │           │  │
└─────────┼───────────┼──┘
          ▼           ▼
       [prev|E0|next]→[prev|E1|next]→[prev|E2|next]
          ▲___________________________________│
get(1) → walk from first OR last, whichever is closer → O(n) worst, O(n/2) avg
```

**subList view (shares backing array):**

```
parent:  [A][B][C][D][E]
                └──┬──┘
sub = parent.subList(1,4)  → view over [B][C][D], NOT a copy
sub.set(0, X) → parent becomes [A][X][C][D][E]
parent.add(F) directly → sub's iterator now throws ConcurrentModificationException
```

## Internal Design Decisions (OpenJDK Perspective)

- `List extends Collection`: enforces that every sequence is also a general-purpose collection — avoids duplicating bulk-op method signatures.
- `Collection extends Iterable`: guarantees for-each loop support framework-wide — a foundational JDK 5 addition alongside generics.
- `RandomAccess` deliberately empty (marker only): zero-cost `instanceof` dispatch vs. virtual method call overhead; matches JDK philosophy of cheap capability-tagging (cf. `Serializable`, `Cloneable`).
- `List` mandates `equals`/`hashCode` override to be **order-sensitive** — differs from `Set` (order-independent) and plain `Object` identity — architects wanted list equality to model sequence equality, not just "same bag of elements."
- Multiple implementations exist because no single structure Pareto-dominates across (a) random access, (b) insertion at arbitrary position, (c) memory density, (d) thread-safety needs — architects chose composability over a single "smart" hybrid structure.

## Advantages

- Program-to-interface flexibility — implementation swap is a one-line change.
- Rich default methods (`sort`, `replaceAll`, `removeIf` via `Collection`) available on every implementation for free.
- Uniform iteration contract (`Iterable`) across the entire framework.

## Disadvantages

- Interface can't express complexity guarantees — easy to accidentally use wrong implementation (index loop on `LinkedList`).
- `subList()` view semantics are a footgun — structural parent modification invalidates the view.
- Type erasure — no primitive `List<int>`; forced into `List<Integer>` with autoboxing overhead.

## Trade-offs

|Decision|Trade-off|
|---|---|
|Interface over concrete class|flexibility vs. no compile-time complexity guarantee|
|`subList` as view vs copy|memory efficiency vs. mutation-safety hazard|
|Order-sensitive `equals()`|intuitive sequence equality vs. can't freely mix with `Set` semantics in generic code|

## Edge Cases

- `Arrays.asList(arr)` returns a **fixed-size** list backed directly by `arr` — `add`/`remove` throw `UnsupportedOperationException`, but `set()` works and mutates the original array.
- `List.of(...)` throws on `null` elements (`NullPointerException`) — stricter than `ArrayList`.
- `subList` after structural parent modification → `ConcurrentModificationException` on next sublist access.
- Autoboxing ambiguity: `list.remove(1)` on `List<Integer>` removes by **index**, not value — must use `list.remove(Integer.valueOf(1))` to remove by value.
- Empty list edge cases: `list.get(0)` on empty list throws `IndexOutOfBoundsException`, not returns null.

## Common Pitfalls

- Index-based `for` loop on `LinkedList` → accidental O(n²).
- Treating `Arrays.asList()` result as fully mutable (fails on `add`/`remove`).
- Forgetting `subList()` is a live view, not a snapshot — mutate defensively with `new ArrayList<>(list.subList(...))` when a copy is needed.
- Structural modification during for-each without `Iterator.remove()`/`removeIf()`.
- Assuming `List.of()` result supports even `set()` — it doesn't; fully immutable.

## Thread Safety

|Implementation|Thread-safe?|Mechanism|
|---|---|---|
|`ArrayList`, `LinkedList`|No|—|
|`Vector`, `Stack`|Yes|`synchronized` on every method (coarse, legacy)|
|`Collections.synchronizedList(list)`|Yes|wrapper, single lock per call — manual `synchronized(list)` needed for compound/iteration ops|
|`CopyOnWriteArrayList`|Yes|copy-on-write: every mutation copies entire backing array; reads are lock-free and never throw CME|

```java
List<Integer> safe = Collections.synchronizedList(new ArrayList<>());
synchronized (safe) {                 // required for iteration — wrapper doesn't protect compound ops
    for (int x : safe) { /* ... */ }
}
```

## Spliterator / Stream Interaction

- `ArrayList.spliterator()`: `SIZED`, `SUBSIZED`, `ORDERED`, `RandomAccess`-backed → splits by index range, cheap and even → excellent parallel stream candidate.
- `LinkedList.spliterator()`: `ORDERED` but not efficiently splittable (no O(1) midpoint) → poor parallel stream performance, often worse than sequential.
- `stream()` is lazy — no traversal cost until a terminal operation (`collect`, `forEach`, `reduce`) executes.

## Comparison Tables

**List vs Collection**

||List|Collection|
|---|---|---|
|Order|guaranteed|not guaranteed (interface-level)|
|Indexed access|yes|no|
|Duplicates|allowed|allowed (impl-dependent for Set subtype)|
|`equals()` semantics|order-sensitive|impl-dependent|

**List vs Set**

||List|Set|
|---|---|---|
|Duplicates|allowed|disallowed|
|Indexed access|yes|no|
|Order|insertion/positional|none (Hash) / insertion (Linked) / sorted (Tree)|
|Backing (typical)|array or nodes|HashMap/TreeMap internally|

**List vs Queue**

||List|Queue|
|---|---|---|
|Access pattern|random (indexed)|restricted (FIFO/priority)|
|Purpose|general sequence|processing order discipline|

**List vs Array**

||List|Array|
|---|---|---|
|Size|dynamic|fixed|
|Primitives|boxed only|native primitive support|
|Type safety|generic, compile-time|reified, runtime-checked|
|API richness|full JCF|minimal|

**ArrayList vs LinkedList**

||ArrayList|LinkedList|
|---|---|---|
|Backing|dynamic array|doubly linked list|
|Random access|O(1)|O(n)|
|Insert/delete at ends|O(1) amortized|O(1)|
|Insert/delete middle|O(n) shift|O(n) traverse (search dominates)|
|Memory/element|low (contiguous)|high (node + 2 pointers)|
|Cache locality|excellent|poor|
|Implements `Deque`|no|yes|

**List vs Vector**

||List (via ArrayList)|Vector|
|---|---|---|
|Thread safety|no|yes (synchronized)|
|Growth factor|1.5x|2x|
|Performance|faster (no lock overhead)|slower|
|Status|modern standard|legacy|

**List vs CopyOnWriteArrayList**

||ArrayList|CopyOnWriteArrayList|
|---|---|---|
|Write cost|O(1) amortized|O(n) (full array copy)|
|Read cost|O(1)|O(1), lock-free|
|Best for|single-threaded/general|read-heavy concurrent|
|Iterator|fail-fast|fail-safe (snapshot, never throws CME)|

**Arrays.asList() vs List.of()**

||`Arrays.asList()`|`List.of()`|
|---|---|---|
|Mutability|fixed-size (set OK, add/remove throw)|fully immutable|
|Backed by|original array (mutations reflect)|internal compact array, independent|
|Null elements|allowed|throws NPE|
|Introduced|Java 1.2|Java 9|

**List.copyOf() vs Collections.unmodifiableList()**

||`List.copyOf()`|`Collections.unmodifiableList()`|
|---|---|---|
|Returns|new immutable copy|live view over original|
|Original mutation reflects?|no (decoupled)|yes (view — original changes propagate)|
|Null elements|throws NPE|allowed (delegates to original)|
|Use case|true immutable snapshot|read-only facade over mutable source|

## DSA Usage ⭐⭐⭐⭐⭐

## Pattern Recognition & Preferred Implementation

|Pattern|Why List fits|Preferred impl|
|---|---|---|
|**Sliding Window**|need indexed access to window bounds|`ArrayList` (or array)|
|**Two Pointers**|need O(1) random access at both ends|`ArrayList`|
|**Prefix Sum**|build cumulative array, index-based lookup|`ArrayList` / array|
|**DP (tabulation)**|1D/2D table with index access|`ArrayList<ArrayList<>>` or array|
|**BFS**|queue discipline via `Deque` (List's cousin)|`ArrayDeque` (not `LinkedList`)|
|**DFS / Backtracking**|maintain path, need append/removeLast|`ArrayList` (`add`/`remove(size-1)` = O(1))|
|**Graph Adjacency List**|list-of-lists, index = vertex id|`List<List<Integer>>` via `ArrayList`|
|**Merge Intervals**|sort + sequential scan|`ArrayList`, sorted via `sort(Comparator)`|
|**Binary Search**|requires O(1) random access|`ArrayList` only — never `LinkedList`|
|**Monotonic Stack**|push/pop at one end|`ArrayDeque`, not `List` directly|
|**Topological Sort**|adjacency list + result ordering|`ArrayList`|

## Common Interview Problems

1. **Merge Intervals** — sort `List<int[]>` by start (O(n log n)), single pass merge (O(n)).
2. **Rotate Array/List** — brute O(n·k) repeated shifts → optimal O(n) via reverse-three-times trick.
3. **Backtracking (Subsets/Permutations)** — `ArrayList` as mutable path, `add`/`remove(size-1)` for O(1) backtrack.
4. **Graph BFS/DFS** — `List<List<Integer>>` adjacency representation, O(V+E) traversal.
5. **Flatten Nested List** — recursive/iterative flattening using `ArrayList` accumulator.

## Optimization Strategies

- Prefer `ArrayList` unless frequent head insertions/deletions dominate (then consider `ArrayDeque`, rarely `LinkedList`).
- Pre-size `new ArrayList<>(n)` when final size is known/estimable — avoids repeated resize-copy.
- Use `removeIf()` (O(n), single pass) instead of loop + `remove()` (O(n²)) for conditional bulk removal.
- For backtracking, always remove the **last** element (`remove(size()-1)`, O(1)) — never remove from front/middle.

## Dry Run

**Rotate `[1,2,3,4,5]` right by 2 (reverse-three-times):**

```
reverse whole:        [5,4,3,2,1]
reverse first k=2:    [4,5,3,2,1]
reverse remaining n-k:[4,5,1,2,3]
result: [4,5,1,2,3]   // O(n) time, O(1) extra space
```

## Interview Questions ⭐⭐⭐⭐⭐

1. **Why is List an interface?** A: Multiple valid backing structures (array/linked) with different performance profiles; interface enables programming to abstraction. _Expects_: mention of Effective Java Item 64. _Mistake_: saying "for polymorphism" without explaining why polymorphism matters here.
    
2. **Why does List allow duplicates?** A: Models a sequence, not a set — real data (logs, ordered readings) legitimately repeats. _Mistake_: confusing with Set's uniqueness contract.
    
3. **Why is RandomAccess a marker interface?** A: Zero-overhead capability tag for algorithm selection (`instanceof` check) vs. method-call dispatch. _Follow-up_: name of the internal class using it (`Collections` static methods).
    
4. **Difference between List and Collection?** A: List adds order + indexed access + duplicate/positional semantics on top of Collection's bulk-operation contract.
    
5. **Difference between List and Iterable?** A: `Iterable` only guarantees `iterator()`; `List` is a full sequence contract several layers above it.
    
6. **Difference between List and ArrayList?** A: interface vs. concrete array-backed implementation. _Trap_: candidates conflating the two.
    
7. **Difference between List and LinkedList?** A: same relation — interface vs. node-based implementation; also LinkedList implements `Deque`, `List` doesn't require that.
    
8. **Why can't Set provide indexed access?** A: No stable positional meaning under hash-based storage; unordered by contract.
    
9. **Why is ListIterator only available for List?** A: Bidirectional/positional traversal requires index semantics unique to sequences.
    
10. **Why can't Iterator move backward?** A: Minimal contract needs to work for structures without a "previous" notion (e.g., some Set internals).
    
11. **Why don't Streams support backward traversal?** A: Pipeline/lazy/parallel model incompatible with stateful cursor position.
    
12. **Why does List preserve insertion order?** A: Required for `get(index)` to be well-defined and deterministic.
    
13. **Mutable vs immutable List?** A: Mutable (`ArrayList`) supports structural modification; immutable (`List.of()`) throws `UnsupportedOperationException` on any mutator.
    
14. **Arrays.asList() vs List.of()?** A: fixed-size + array-backed + nulls OK vs fully immutable + independent + no nulls. _Trap_: assuming `asList()` is fully mutable.
    
15. **List.copyOf() vs Collections.unmodifiableList()?** A: independent immutable snapshot vs. live read-only view over mutable source. _Trap_: assuming both are equivalent — original mutation propagates only through the latter.
    
16. **How does fail-fast work?** A: `modCount` tracked, compared against iterator's snapshot on each `next()`; best-effort, not a concurrency guarantee.
    
17. **Can List contain null?** A: Implementation-dependent — `ArrayList`/`LinkedList` yes, `List.of()` no (NPE).
    
18. **How does equals() work in List?** A: Order-sensitive element-wise comparison; two `List`s of different types (ArrayList vs LinkedList) can still be equal if same elements, same order.
    
19. **How does hashCode() work in List?** A: `Objects.hash`-style formula combining element hashes in order — defined explicitly in `List` javadoc: `31*hash + e.hashCode()` accumulated, sensitive to order.
    
20. **Why do List implementations have different complexities?** A: Interface defines behavior, not performance; backing structure dictates complexity.
    
21. **When should you program to the List interface?** A: Always in method signatures/fields unless a specific implementation's extra API (e.g., `Deque` methods on LinkedList) is genuinely required.
    
22. **Why is List preferred in dependency injection / Spring?** A: Enables swapping implementation (e.g., test double vs. production list) without touching consuming code; also Spring auto-wires all matching beans into a `List<T>`.
    
23. **What happens if you use `subList()` then modify the parent structurally?** A: Subsequent sublist operations throw `ConcurrentModificationException`.
    
24. **Why does `remove(int)` vs `remove(Object)` cause confusion with `List<Integer>`?** A: Autoboxing ambiguity — `remove(1)` resolves to index-based overload; must box explicitly for value-based removal.
    
25. **Why is ArrayList preferred over LinkedList in almost all modern use cases?** A: Better cache locality, lower memory overhead per element, and LinkedList's theoretical O(1) mid-insert is negated by O(n) search cost to reach that position.
    

## Real-World Backend Usage

- **Spring Boot**: `List<T>` used for DI collection injection (`@Autowired List<Strategy>` gathers all beans implementing an interface), REST response bodies as JSON arrays.
- **Hibernate/JPA**: `@OneToMany` associations commonly typed as `List<Entity>` with `@OrderColumn` for persistent ordering.
- **REST APIs / DTOs**: response payloads serialize `List<DTO>` to JSON arrays (Jackson: `List` ↔ `[]`).
- **Service/Repository layers**: `List` as the standard return type for query results (`findAll()` returns `List<Entity>`).
- **Batch processing**: chunked `List<T>` partitions for batch inserts/Spring Batch `ItemWriter`.
- **Streams/parallel processing**: `list.stream()`/`parallelStream()` for transformation pipelines; `RandomAccess` lists parallelize efficiently.

## Production Best Practices

- Program to `List<T>` interface in method signatures/return types, not `ArrayList<T>`.
- Return defensive copies or `Collections.unmodifiableList()`/`List.copyOf()` from getters to prevent external mutation of internal state.
- Pre-size `ArrayList` when count is known/estimable to reduce resize-copy overhead.
- Avoid `Vector`/`Stack` — legacy, unnecessary synchronization cost.
- Never use `LinkedList` for index-heavy access patterns — verify actual access pattern before choosing it (it rarely wins in practice; `ArrayDeque` beats it even for queue/stack use).
- Be explicit about `subList()` lifetime — copy into a new `ArrayList` if the sublist must outlive/be independent of parent mutations.
- Use `Collections.emptyList()` instead of `null` for "no results" — avoids NPE at call sites.

## Cheat Sheet

```
Need random access             → ArrayList (implements RandomAccess)
Need frequent head/tail ops    → ArrayDeque (not LinkedList)
Need thread-safety, read-heavy → CopyOnWriteArrayList
Need thread-safety, general    → Collections.synchronizedList()
Need immutability               → List.of() / List.copyOf()
Need positional edit while      → ListIterator
  iterating
Need index loop safety check    → instanceof RandomAccess
```

## Quick Revision

- `List extends Collection extends Iterable` — adds order + index-based access + duplicates.
- Interface ≠ complexity guarantee — complexity comes from the implementation, not the contract.
- `RandomAccess`: zero-cost marker interface to enable algorithm selection.
- `ListIterator` > `Iterator`: bidirectional, supports `set()`/`add()` during traversal.
- Fail-fast: best-effort `modCount` check, not a concurrency mechanism.
- `subList()` is a **live view**, not a copy — mutation hazard.
- `Arrays.asList()`: fixed-size, array-backed. `List.of()`: fully immutable, no nulls.
- Always prefer `ArrayList` unless proven otherwise by actual access-pattern profiling.
- `equals()`/`hashCode()` on `List` are order-sensitive — unlike `Set`.