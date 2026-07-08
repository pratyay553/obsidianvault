---
tags: [java, streams, optional]
---

# 14. Optional with Streams

⬅ Back to [[00-Home]] | Prev: [[13-Primitive-Streams]] | Next: [[15-Performance]]

## Why Streams Return `Optional`

Many terminal operations may legitimately have **no answer** (empty stream, no match found): `findFirst`, `findAny`, `min`, `max`, `reduce()` (1-arg). Rather than returning `null` (inviting `NullPointerException`), Streams return `Optional<T>` to force the caller to **explicitly handle absence**.

```java
Optional<String> first = names.stream().filter(n -> n.startsWith("Z")).findFirst();
```

## Core `Optional` API

| Method | Behavior |
|---|---|
| `isPresent()` | `true` if value exists |
| `isEmpty()` (Java 11+) | `true` if no value |
| `get()` | Returns value or throws `NoSuchElementException` — **avoid using directly** |
| `orElse(default)` | Returns value or a default (default is **always evaluated eagerly**, even if not used!) |
| `orElseGet(Supplier)` | Returns value or lazily computed default — **prefer this over `orElse` for expensive defaults** |
| `orElseThrow()` | Throws `NoSuchElementException` if empty |
| `orElseThrow(Supplier<X>)` | Throws custom exception if empty |
| `ifPresent(Consumer)` | Runs consumer only if present |
| `ifPresentOrElse(Consumer, Runnable)` (Java 9+) | Present/absent branches, both explicit |
| `map(Function)` | Transform value if present, else stays empty |
| `flatMap(Function<T,Optional<U>>)` | Chain optional-returning functions without nested `Optional<Optional<U>>` |
| `filter(Predicate)` | Keep value only if predicate true, else becomes empty |
| `stream()` (Java 9+) | `Optional<T>` → `Stream<T>` (0 or 1 element) — great with `flatMap` |
| `or(Supplier<Optional<T>>)` (Java 9+) | Fallback to another `Optional` if empty |

## `orElse` vs `orElseGet` — Classic Gotcha

```java
// BUG: computeExpensiveDefault() runs EVERY time, even if value is present!
String result = optional.orElse(computeExpensiveDefault());

// FIX: lazy, only runs if empty
String result = optional.orElseGet(() -> computeExpensiveDefault());
```
This is one of the most commonly asked "spot the bug" interview questions — see Q entries in [[20-Interview-Questions]].

## Chaining Cleanly Without `if`/`isPresent()`

```java
// Imperative (avoid):
Optional<Employee> emp = findById(id);
if (emp.isPresent()) {
    Employee e = emp.get();
    if (e.getManager() != null) {
        System.out.println(e.getManager().getName());
    }
}

// Functional (preferred):
findById(id)
    .map(Employee::getManager)
    .map(Manager::getName)
    .ifPresentOrElse(System.out::println, () -> System.out.println("No manager"));
```

## `Optional.stream()` + `flatMap` — Removing Empties Elegantly

```java
List<Optional<String>> maybeNames = List.of(Optional.of("A"), Optional.empty(), Optional.of("B"));

List<String> present = maybeNames.stream()
        .flatMap(Optional::stream)   // drops empties automatically
        .toList();
```

## Primitive Optionals: `OptionalInt`, `OptionalLong`, `OptionalDouble`

```java
OptionalInt max = IntStream.of(3, 1, 4, 1, 5).max();
int value = max.getAsInt();   // NOT get()! primitive optionals use getAsX()
```
⚠️ These do **not** support `map`/`flatMap` returning object types as flexibly as `Optional<T>` — API is intentionally leaner. See [[13-Primitive-Streams]].

## Anti-Patterns to Avoid

```java
// ❌ Defeats the purpose of Optional entirely
if (optional.get() != null) { ... }

// ❌ Optional as a field or method parameter (Optional is meant as a RETURN type only)
class Employee {
    private Optional<String> middleName;   // avoid — use nullable field or empty string instead
}

// ❌ Calling get() without checking isPresent() first
String s = optional.get();  // NoSuchElementException risk
```

## Interview Soundbite

> "Stream terminal ops that might not produce a value return `Optional` instead of `null`, forcing explicit handling. Prefer `orElseGet` over `orElse` when the default is expensive (laziness), use `map`/`flatMap`/`filter` to chain without manual null checks, and never use `Optional` as a field type or method parameter — it's designed as a return type only."

Next: [[15-Performance]]
