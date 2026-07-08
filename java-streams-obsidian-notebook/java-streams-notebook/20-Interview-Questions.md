---
tags: [java, streams, interview-questions]
---

# 20. 50 Interview Questions with Answers

⬅ Back to [[00-Home]] | Prev: [[19-Best-Practices]] | Next: [[21-Coding-Problems]]

> Organized by theme. Each answer is interview-length — expand using the linked notes for depth.

## Fundamentals (Q1–Q10)

**Q1. What is a Stream in Java?**
A pipeline over a data source enabling declarative, functional-style operations (filter, map, reduce) without storing data itself. See [[01-Introduction]].

**Q2. Is a Stream a data structure?**
No. It holds no elements; it computes them on demand from a source (Collection, array, generator, I/O). See [[01-Introduction]].

**Q3. Can a Stream be reused after a terminal operation?**
No — it throws `IllegalStateException`. Rebuild from the source instead. See [[04-Stream-Pipeline]].

**Q4. What are the three parts of a Stream pipeline?**
Source, zero-or-more intermediate operations, exactly one terminal operation. See [[04-Stream-Pipeline]].

**Q5. Why were Streams introduced in Java 8?**
To reduce boilerplate from external iteration, exploit multi-core hardware easily via built-in parallelism, and complement the newly introduced lambda expressions. See [[02-Why-Streams]].

**Q6. What is internal vs external iteration?**
External: the caller manages the loop/iterator (`for`, `Iterator.hasNext/next`). Internal: the library (Stream) drives iteration; the caller only supplies behavior (lambdas). See [[02-Why-Streams]].

**Q7. What does "lazy evaluation" mean for Streams?**
Intermediate operations don't execute until a terminal operation is invoked; then, each element flows fully through the chain before the next starts. See [[05-Lazy-Evaluation]].

**Q8. Can Streams be infinite?**
Yes — `Stream.iterate(seed, fn)` and `Stream.generate(supplier)` create unbounded streams, made usable via laziness + short-circuiting ops like `limit`/`findFirst`. See [[05-Lazy-Evaluation]].

**Q9. What interface underlies every Stream implementation?**
`Spliterator<T>`, wrapped internally by `AbstractPipeline`/`ReferencePipeline`. See [[03-Internal-Architecture]] and [[12-Spliterator]].

**Q10. What is a `Sink` in Stream internals?**
The internal chain-of-responsibility abstraction (`Sink<T> extends Consumer<T>`) that connects pipeline stages; each intermediate op wraps the downstream Sink. See [[03-Internal-Architecture]].

## Intermediate & Terminal Operations (Q11–Q20)

**Q11. Difference between intermediate and terminal operations?**
Intermediate: lazy, return a new Stream (`filter`, `map`, `sorted`). Terminal: eager, produce a result/side-effect and close the stream (`collect`, `forEach`, `reduce`). See [[06-Intermediate-Operations]], [[07-Terminal-Operations]].

**Q12. Which intermediate operations are stateful, and why does it matter?**
`sorted`, `distinct`, `limit`, `skip`, `dropWhile` — they need to see some/all elements before emitting, which can require buffering and hurts parallel performance vs stateless ops like `filter`/`map`. See [[06-Intermediate-Operations]].

**Q13. Difference between `map` and `flatMap`?**
`map` is 1-to-1; `flatMap` is 1-to-many with automatic flattening of nested streams. See [[10-FlatMap]].

**Q14. Difference between `findFirst()` and `findAny()`?**
`findFirst` always returns the first element in encounter order (even parallel, at extra coordination cost); `findAny` may return any matching element, faster in parallel, non-deterministic. See [[07-Terminal-Operations]].

**Q15. What do `anyMatch`, `allMatch`, `noneMatch` return on an empty stream?**
`anyMatch` → false; `allMatch` → true (vacuous truth); `noneMatch` → true. See [[07-Terminal-Operations]].

**Q16. What's the difference between `takeWhile` and `filter`?**
`filter` checks every element; `takeWhile` stops at the first element failing the predicate (short-circuits), suited to sorted/prefix scenarios. See [[06-Intermediate-Operations]].

**Q17. Why is `peek()` risky to rely on?**
The JVM may skip traversal (and thus `peek` calls) if a terminal op like `count()` can compute its result without visiting elements. Use it for debugging only. See [[05-Lazy-Evaluation]], [[18-Common-Bugs]].

**Q18. Can `count()` avoid traversing the stream?**
Yes, if the stream's size is exactly known (`SIZED` flag preserved, no filtering) — the JDK can return size without visiting elements. See [[07-Terminal-Operations]].

**Q19. What happens if you call a terminal operation twice on the same stream?**
`IllegalStateException: stream has already been operated upon or closed`. See [[04-Stream-Pipeline]].

**Q20. How do you convert a `Stream<Integer>` to an `IntStream` and back?**
`stream.mapToInt(Integer::intValue)` and `intStream.boxed()`. See [[13-Primitive-Streams]].

## Collectors & Reduce (Q21–Q30)

**Q21. What are the four components of the `Collector` interface?**
`supplier` (new container), `accumulator` (fold element in), `combiner` (merge containers, for parallel), `finisher` (container → result). See [[08-Collectors]].

**Q22. Difference between `groupingBy` and `partitioningBy`?**
`groupingBy` groups by an arbitrary key function into `Map<K, List<T>>`; `partitioningBy` always splits into exactly two boolean buckets `Map<Boolean, List<T>>`. See [[08-Collectors]].

**Q23. How do you handle duplicate keys in `Collectors.toMap`?**
Supply a 3rd merge-function argument: `toMap(keyFn, valFn, (a,b) -> a)`, otherwise it throws `IllegalStateException`. See [[08-Collectors]].

**Q24. What does `Collectors.mapping` do?**
Adapts a downstream collector to first transform each element, useful nested inside `groupingBy`, e.g., collecting just names per department. See [[08-Collectors]].

**Q25. What's `Collectors.teeing()` used for?**
Runs two collectors in a single pass over the stream and merges their results (Java 12+) — e.g., computing min and max simultaneously. See [[08-Collectors]].

**Q26. Difference between the three `reduce()` overloads?**
No-identity → `Optional<T>`; with identity → `T`; with identity+accumulator+combiner → `U` (type-changing, parallel-safe). See [[09-Reduce]].

**Q27. Why does 3-arg `reduce` need a combiner even for sequential streams?**
For API consistency and correctness if the same code later runs in parallel — switching `.stream()` to `.parallelStream()` shouldn't change semantics. See [[09-Reduce]].

**Q28. Why shouldn't you build a mutable collection with `reduce`?**
It risks unsafe mutation of shared state, especially in parallel; `collect()` exists specifically for safe mutable reduction. See [[09-Reduce]], [[18-Common-Bugs]].

**Q29. How would you compute min, max, sum, count, average in a single pass?**
`IntStream.summaryStatistics()` (or Long/DoubleSummaryStatistics) — computes all five in one traversal. See [[13-Primitive-Streams]].

**Q30. What's `Collectors.collectingAndThen` for?**
Wraps a collector and post-processes its result, e.g., making a mutable list immutable: `collectingAndThen(toList(), Collections::unmodifiableList)`. See [[08-Collectors]].

## Parallel Streams & Spliterator (Q31–Q38)

**Q31. What thread pool do parallel streams use by default?**
`ForkJoinPool.commonPool()`, shared JVM-wide. See [[11-Parallel-Streams]].

**Q32. When should you avoid parallel streams?**
Small datasets, I/O-bound work, poorly-splitting sources (`LinkedList`, `Stream.iterate`), or when order must be strictly preserved. See [[11-Parallel-Streams]].

**Q33. What is a `Spliterator` and what does `trySplit()` do?**
The traversal+splitting abstraction backing every Stream; `trySplit()` divides the remaining elements into two parts so each can be processed independently (often in parallel). See [[12-Spliterator]].

**Q34. Why can `LinkedList` parallel streams perform poorly?**
Its Spliterator can't binary-split efficiently (no random access; must walk node links to find a midpoint), leading to unbalanced or ineffective splitting. See [[11-Parallel-Streams]], [[12-Spliterator]].

**Q35. What Spliterator characteristic flags exist, and why do they matter?**
`ORDERED`, `SIZED`, `SUBSIZED`, `SORTED`, `DISTINCT`, `NONNULL`, `IMMUTABLE`, `CONCURRENT` — they let the pipeline skip redundant work (e.g., skip re-sorting an already-`SORTED` source). See [[12-Spliterator]].

**Q36. How can a long-running parallel stream harm the rest of an application?**
It can starve other tasks sharing the same common `ForkJoinPool` (e.g., `CompletableFuture.supplyAsync`); mitigate by submitting to a custom `ForkJoinPool`. See [[11-Parallel-Streams]].

**Q37. `findFirst()` vs `findAny()` in parallel — which is faster and why?**
`findAny()` — it doesn't need to coordinate to guarantee encounter order, so it can return as soon as any thread finds a match. See [[07-Terminal-Operations]], [[11-Parallel-Streams]].

**Q38. How do you write a custom Spliterator?**
Implement `tryAdvance`, `trySplit`, `estimateSize`, `characteristics`, then feed it to `StreamSupport.stream(spliterator, parallel)`. See [[12-Spliterator]].

## Design, Comparisons & Edge Cases (Q39–Q50)

**Q39. Difference between a Stream and a Collection?**
Collection stores data, supports repeated external iteration and mutation; Stream is a one-shot lazy computation pipeline with internal iteration, can be infinite. See [[16-Stream-vs-Collection]].

**Q40. When would you prefer a loop over a Stream?**
When you need checked-exception handling, complex early-exit/branching (labeled `break`), or mutable local accumulator logic. See [[17-Stream-vs-Loop]].

**Q41. Why does `orElse()` sometimes cause performance bugs?**
Its argument is eagerly evaluated even when the `Optional` has a value; use `orElseGet(Supplier)` for expensive defaults. See [[14-Optional-with-Streams]], [[18-Common-Bugs]].

**Q42. Why shouldn't `Optional` be used as a method parameter or field type?**
It's designed as a return type signaling "may be absent" — using it elsewhere adds serialization/boilerplate overhead without idiomatic benefit. See [[14-Optional-with-Streams]].

**Q43. What's the difference between `Stream.of(null)` behavior and `Optional.ofNullable`?**
`Stream.of(null)` throws NPE for certain operations expecting `NONNULL`, while `Stream.ofNullable(x)` creates an empty stream if `x` is null (Java 9+), and `Optional.ofNullable` wraps a possibly-null value safely.

**Q44. Why can't you mutate a local variable inside a lambda passed to a Stream?**
Java requires captured local variables to be effectively final, ensuring safe, unambiguous semantics, particularly under parallel execution. See [[18-Common-Bugs]].

**Q45. How would you process a huge file without loading it all into memory?**
`Files.lines(path)` returns a lazily-read `Stream<String>` backed by a `BufferedReader`; wrap in try-with-resources to release the file handle. See [[19-Best-Practices]].

**Q46. What does `Collectors.groupingBy(fn, TreeMap::new, downstream)` give you that the default doesn't?**
Deterministic, sorted-by-key iteration order — the default `groupingBy` uses `HashMap`, with no order guarantee. See [[08-Collectors]].

**Q47. Explain a case where `.distinct()` doesn't behave as expected.**
If the element type doesn't override `equals`/`hashCode` (default identity semantics), logically "equal" objects won't be deduplicated. See [[18-Common-Bugs]].

**Q48. Are Streams thread-safe?**
The Stream pipeline machinery is designed for safe (even parallel) execution *if* your lambdas are pure and side-effect-free; if you introduce shared mutable state in a lambda, you can break thread-safety yourself. See [[18-Common-Bugs]].

**Q49. What's the benefit of `mapMulti` (Java 16+) over `flatMap`?**
It avoids constructing an intermediate `Stream` object per element when the "many" side is small — can be faster in hot paths. See [[10-FlatMap]].

**Q50. How would you explain Streams to a junior developer in one sentence?**
"A Stream lets you describe a data-processing pipeline (filter, transform, aggregate) declaratively, and Java handles the iteration — sequentially or in parallel — for you." See [[01-Introduction]].

---
Next: [[21-Coding-Problems]]
