---
title: "Core Java Structure - Main Method"
aliases: [Core Java, main method]
tags: [java, fundamentals]
difficulty: beginner
status: learning
---

# Core Java Structure — The Main Method

> [!abstract] One-Line Definition
> Every standalone Java program starts execution from one fixed entry point: `public static void main(String[] args)`.

---

## 1. Big Picture / Mental Model

```java
public class Main {
    public static void main(String[] args) {
        // execution starts here
    }
}
```

> [!quote] From the notes
> `Main.java` contains `public class Main { public static void main(String[] args) { } }`

## 2. Why It Matters

The JVM needs one predictable, unambiguous place to start running your code. That's what `main` is — a contract every Java program follows so the JVM knows exactly where to begin.

## 3. How It Works — dissecting the signature

| Keyword | Meaning |
|---|---|
| `public` | Accessible from outside the class — the JVM (external caller) needs to reach it |
| `static` | Belongs to the class itself, not an instance — the JVM can call it without creating an object first |
| `void` | Returns nothing |
| `main` | The fixed, required method name the JVM looks for |
| `String[] args` | Command-line arguments passed in when the program is run |

## 4. Spring Boot / Java Notes

This exact pattern reappears, barely changed, as the entry point of every Spring Boot application:

```java
@SpringBootApplication
public class FirstSpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(FirstSpringBootApplication.class, args);
    }
}
```

The only new thing is the single line inside `main` — `SpringApplication.run(...)`, which hands control over to Spring Boot's own startup process (starts the embedded Tomcat, scans for components, etc.). See [[21 - The Main Application Class]].

## 5. Common Pitfalls & Misconceptions

- ❌ Thinking `main` can have any name — the JVM specifically looks for a method named exactly `main` with that exact signature.
- ❌ Forgetting `static` — without it, the JVM would need an *instance* of your class to call the method, but no instance exists yet at startup.

## 6. Interview Questions

**Beginner**
1. Why must `main` be `static`?
2. What does the JVM do if it can't find a `main` method?

## 7. Quick Revision
- Fixed signature: `public static void main(String[] args)`.
- `static` → callable without an object instance.
- Spring Boot's `main` just delegates to `SpringApplication.run(...)`.

## Related
- [[09 - From .java to .class - Compilation and the JVM]]
- [[21 - The Main Application Class]]
