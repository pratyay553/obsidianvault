---
tags: [java, streams, introduction]
---

# 1. Introduction to Java Stream API

⬅ Back to [[00-Home]] | Next: [[02-Why-Streams]]

## What is a Stream?

A **Stream** (`java.util.stream`) is an abstraction introduced in **Java 8** that represents a **sequence of elements supporting functional-style, declarative operations** on data — filtering, mapping, sorting, reducing — without exposing or mutating the underlying data structure.

> **A Stream is NOT a data structure.** It doesn't store elements. It's a *pipeline* that computes elements on demand from a source (Collection, array, I/O channel, generator function, etc.).

## Key Characteristics

| Property                 | Meaning                                                                                       |
| ------------------------ | --------------------------------------------------------------------------------------------- |
| **Not a storage device** | Streams carry values *through* a pipeline; they don't hold them.                              |
| **Functional in nature** | Operations produce a result without mutating the source.                                      |
| **Lazily evaluated**     | Nothing runs until a terminal operation is invoked. See [[05-Lazy-Evaluation]].               |
| **Possibly unbounded**   | Streams can be infinite (`Stream.iterate`, `Stream.generate`).                                |
| **Consumable only once** | Once a terminal operation runs, the stream is closed. Reusing throws `IllegalStateException`. |
| **Can be parallelized**  | `.parallelStream()` or `.parallel()` — see [[11-Parallel-Streams]].                           |

## Minimal Example

```java
List<String> names = List.of("Ravi", "Anita", "Suresh", "Meena");

List<String> result = names.stream()          // source
        .filter(n -> n.length() > 4)          // intermediate
        .map(String::toUpperCase)             // intermediate
        .sorted()                             // intermediate
        .collect(Collectors.toList());        // terminal

System.out.println(result); // [ANITA, SURESH]
```

## Package Overview

```
java.util.stream
 ├── Stream<T>            — object streams
 ├── IntStream            — primitive int stream
 ├── LongStream           — primitive long stream
 ├── DoubleStream         — primitive double stream
 ├── Collector<T,A,R>     — reduction recipe (used with collect())
 ├── Collectors           — factory of common Collector implementations
 └── StreamSupport        — bridge for custom Spliterators
```

## Stream Creation Methods (Quick Reference)

```java
Stream.of(1, 2, 3);
Stream.empty();
Stream.ofNullable(maybeNull);
Arrays.stream(arr);
list.stream();
list.parallelStream();
Stream.iterate(0, n -> n + 2);                 // infinite
Stream.iterate(0, n -> n < 20, n -> n + 2);     // bounded (Java 9+)
Stream.generate(Math::random);                  // infinite
Files.lines(path);                              // I/O backed
IntStream.range(0, 10);
Pattern.compile(",").splitAsStream(csvLine);
```

## Why This Matters for Interviews

Interviewers almost always start with *"What is a Stream? Is it a data structure?"* — the correct, crisp answer is:

> "A Stream is a pipeline over a data source that lets you express *what* to compute declaratively; it does not store data, is lazily evaluated, and can be traversed only once."

Continue to [[02-Why-Streams]] to understand the motivation behind this design.
