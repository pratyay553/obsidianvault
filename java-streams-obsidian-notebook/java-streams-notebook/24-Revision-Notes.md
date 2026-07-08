---
tags: [java, streams, revision]
---

# 24. Revision Notes — 15-Minute Final Skim

⬅ Back to [[00-Home]] | Prev: [[23-Cheat-Sheet]]

> Read this the morning of the interview. Every line links to the full note if you need to jog your memory further.

## The One-Paragraph Summary

A **Stream** ([[01-Introduction]]) is a lazy, one-shot, functional-style pipeline over a data source — not a data structure. It was introduced ([[02-Why-Streams]]) to reduce iteration boilerplate and make multi-core parallelism accessible. Internally ([[03-Internal-Architecture]]), every stream is a chain of pipeline stages wrapping `Sink`s over a `Spliterator` ([[12-Spliterator]]); nothing runs until a **terminal operation** ([[07-Terminal-Operations]]) pulls elements through the chain one at a time, enabling short-circuiting and infinite streams ([[05-Lazy-Evaluation]]). Intermediate operations ([[06-Intermediate-Operations]]) are lazy and either stateless (`filter`, `map`) or stateful (`sorted`, `distinct`, `limit`). `collect()` with `Collectors` ([[08-Collectors]]) is the standard way to build mutable results (lists, maps, grouped/partitioned data); `reduce()` ([[09-Reduce]]) is for immutable, associative combination. `flatMap()` ([[10-FlatMap]]) flattens one-to-many relationships. `IntStream`/`LongStream`/`DoubleStream` ([[13-Primitive-Streams]]) avoid boxing overhead. `Optional` ([[14-Optional-with-Streams]]) replaces `null` for "maybe absent" results. Parallel streams ([[11-Parallel-Streams]]) use the common `ForkJoinPool` and help only for large, CPU-bound, well-splitting workloads — always measure ([[15-Performance]]).

## Top 15 Facts to Have Instantly Ready

1. A Stream is not a data structure — it stores nothing, computes on demand.
2. A Stream can be traversed exactly once; reuse throws `IllegalStateException`.
3. Intermediate ops are lazy; terminal ops are eager and trigger execution.
4. Elements flow one-at-a-time through the *entire* stateless chain (not stage-by-stage batch processing).
5. `sorted`/`distinct`/`limit`/`skip` are **stateful** — may require buffering.
6. `findFirst` respects order; `findAny` doesn't (faster in parallel).
7. `anyMatch`/`allMatch`/`noneMatch` short-circuit.
8. `map` = 1-to-1; `flatMap` = 1-to-many, flattened.
9. `reduce` = immutable combination; `collect` = mutable accumulation (thread-safe in parallel).
10. `Collectors.toMap` throws on duplicate keys unless you supply a merge function.
11. `groupingBy` → arbitrary-key map; `partitioningBy` → always exactly `{true, false}` keys.
12. `orElse` is eager; `orElseGet` is lazy — use the latter for expensive defaults.
13. `peek()` is debug-only; may be skipped by the JVM if traversal isn't otherwise required.
14. Parallel streams use `ForkJoinPool.commonPool()`; poor-splitting sources (`LinkedList`, `Stream.iterate`) gain little or nothing.
15. Primitive streams (`IntStream` etc.) avoid autoboxing — prefer them for numeric aggregation.

## Formulas / Patterns to Memorize Cold

```java
// Group + count
Map<K, Long> counts = list.stream().collect(Collectors.groupingBy(keyFn, Collectors.counting()));

// Group + downstream transform
Map<K, D> grouped = list.stream().collect(Collectors.groupingBy(keyFn, downstreamCollector));

// Partition
Map<Boolean, List<T>> parts = list.stream().collect(Collectors.partitioningBy(predicate));

// Safe toMap with duplicates
Map<K, V> map = list.stream().collect(Collectors.toMap(keyFn, valFn, (a, b) -> a));

// Single-pass stats
IntSummaryStatistics stats = list.stream().mapToInt(fn).summaryStatistics();

// Flatten nested
List<T> flat = nested.stream().flatMap(Collection::stream).toList();

// Immutable result
List<T> immutable = list.stream().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
```

## The Five Bugs Interviewers Love to Ask About

See full list in [[18-Common-Bugs]]:
1. Reusing a consumed stream.
2. Unsafe shared mutable state in `parallelStream().forEach()`.
3. `orElse()` eager evaluation trap.
4. `peek()` skipped by optimization.
5. `Collectors.toMap` duplicate-key crash.

## Final Self-Test (Answer Out Loud Before the Interview)

- [ ] Explain the difference between `map` and `flatMap` with an example.
- [ ] Explain why `reduce()` needs a combiner even in sequential mode.
- [ ] Explain what happens internally when you call `.filter().map().collect()`.
- [ ] Explain when NOT to use a parallel stream.
- [ ] Explain the difference between a Stream and a Collection in one sentence.
- [ ] Write `groupingBy` + `partitioningBy` from memory.
- [ ] Explain the `Optional.orElse` vs `orElseGet` trap.
- [ ] Explain what a `Spliterator` is and its role in parallelism.

If you can answer all eight without opening this vault — you're ready. Good luck! 🚀

⬅ Back to [[00-Home]]
