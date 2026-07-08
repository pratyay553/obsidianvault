---
tags: [java, streams, primitives]
---

# 13. Primitive Streams

⬅ Back to [[00-Home]] | Prev: [[12-Spliterator]] | Next: [[14-Optional-with-Streams]]

## Why They Exist: Avoiding Boxing Overhead

`Stream<Integer>` boxes every `int` into an `Integer` object — extra memory, extra GC pressure, cache-unfriendly. `IntStream`, `LongStream`, `DoubleStream` store **primitives directly**, avoiding this entirely.

```java
// Boxed — slower, more memory
Stream<Integer> boxed = list.stream().map(...);

// Primitive — faster, no boxing
IntStream primitive = list.stream().mapToInt(...);
```

## The Three Primitive Stream Types

| Type | Wraps | Notable extra methods |
|---|---|---|
| `IntStream` | `int` | `range`, `rangeClosed`, `asLongStream`, `asDoubleStream` |
| `LongStream` | `long` | same pattern |
| `DoubleStream` | `double` | same pattern (no `range`, since doubles aren't naturally "counted") |

## Creating Primitive Streams

```java
IntStream.range(0, 5);          // 0,1,2,3,4  (end exclusive)
IntStream.rangeClosed(0, 5);    // 0,1,2,3,4,5 (end inclusive)
IntStream.of(1, 2, 3);
Arrays.stream(intArray);
"hello".chars();                 // IntStream of char codes
random.ints(5);                  // IntStream from java.util.Random
```

## Converting Between Object and Primitive Streams

```java
Stream<Integer> boxedStream = intStream.boxed();          // IntStream -> Stream<Integer>
IntStream intStream = stream.mapToInt(Integer::intValue); // Stream<Integer> -> IntStream

// map/mapToObj for type-changing:
IntStream lengths = words.stream().mapToInt(String::length);      // Stream<String> -> IntStream
Stream<String> backToStrings = lengths.mapToObj(String::valueOf); // IntStream -> Stream<String>
```

## Extra Terminal Operations Only on Primitive Streams

```java
IntStream nums = IntStream.rangeClosed(1, 10);

int sum = nums.sum();
OptionalDouble avg = IntStream.rangeClosed(1, 10).average();
OptionalInt max = IntStream.rangeClosed(1, 10).max();
OptionalInt min = IntStream.rangeClosed(1, 10).min();
IntSummaryStatistics stats = IntStream.rangeClosed(1, 10).summaryStatistics();

System.out.println(stats.getMin() + " " + stats.getMax() + " " + stats.getAverage() + " " + stats.getSum() + " " + stats.getCount());
```
`summaryStatistics()` computes **min, max, sum, average, count in a single pass** — far more efficient than 5 separate terminal operations.

## `mapToInt` vs `map` on Object Streams

```java
// map() would give Stream<Integer> (boxed)
Stream<Integer> boxedLengths = words.stream().map(String::length);

// mapToInt() gives IntStream (unboxed) — preferred for numeric aggregation
int totalChars = words.stream().mapToInt(String::length).sum();
```

## `Optional` Counterparts for Primitives

Primitive streams use specialized `Optional` types to avoid boxing:

| Stream type | Optional-returning method result |
|---|---|
| `IntStream` | `OptionalInt` |
| `LongStream` | `OptionalLong` |
| `DoubleStream` | `OptionalDouble` |

See [[14-Optional-with-Streams]] for details on why `getAsInt()`, not `get()`.

## When NOT to Use Primitive Streams

- When you need object-oriented features (custom `Comparator`s referencing full objects, `null` handling) — primitives can't be `null`.
- When conversions (`boxed()`/`mapToInt()`) happen so often that they add more overhead than they save — profile if unsure.

## Interview Soundbite

> "`IntStream`/`LongStream`/`DoubleStream` avoid the memory and performance overhead of autoboxing every primitive into a wrapper object. They provide numeric-specific terminal ops like `sum()`, `average()`, and `summaryStatistics()`, and their optional-returning ops use primitive-specialized `OptionalInt/Long/Double` rather than a boxed `Optional<Integer>`."

Next: [[14-Optional-with-Streams]]
