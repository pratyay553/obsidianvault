---
title: "The Main Application Class"
aliases: ["@SpringBootApplication", "SpringApplication.run"]
tags: [spring-boot, java]
difficulty: beginner
status: learning
---

# The Main Application Class

> [!abstract] One-Line Definition
> The class annotated `@SpringBootApplication` and containing `main()` is where every Spring Boot application starts.

---

## 1. Big Picture / Mental Model

```java
@SpringBootApplication
public class FirstSpringBootApplication {

    public static void main(String[] args) {
        SpringApplication.run(FirstSpringBootApplication.class, args);
    }
}
```

## 2. Why It Matters

This class is the bridge between plain Java (see [[08 - Core Java Structure - Main Method]]) and the entire Spring Boot machinery. For now, per the original notes: *"we only need to understand that this class starts our Spring Boot application. We will understand `@SpringBootApplication` deeply later."*

## 3. How It Works

- `public static void main(String[] args)` — the standard Java entry point the JVM looks for.
- `SpringApplication.run(FirstSpringBootApplication.class, args)` — this single line hands control to Spring Boot, which then:
  1. Creates the application context.
  2. Starts the embedded Tomcat server.
  3. Configures Spring MVC.
  4. Scans for and registers controller classes.
  5. Starts listening for HTTP requests.

`@SpringBootApplication` is itself a convenience annotation that bundles together several other annotations (component scanning, auto-configuration, and marking this as a configuration class) — the notes intentionally defer this detail for a later lecture.

## 4. Spring Boot / Java Notes

You typically create **one** of these classes per project, and it lives at the root of your source package so Spring's component scanning can find everything beneath it (like your `HelloController`).

## 5. Common Pitfalls & Misconceptions

- ❌ Placing controllers/components in a package *outside* the main application class's package tree — Spring's default component scan only looks at the current package and its sub-packages.
- ❌ Creating multiple `@SpringBootApplication` classes in one project without understanding the implications — usually you want exactly one.

## 6. Interview Questions

**Beginner**
1. What does `SpringApplication.run(...)` do?
2. Why does the main class need `@SpringBootApplication`?

## 7. Quick Revision
- One class, annotated `@SpringBootApplication`, with a standard Java `main()`.
- `SpringApplication.run(...)` kicks off the entire Spring Boot startup sequence.

## Related
- [[08 - Core Java Structure - Main Method]]
- [[22 - Controllers and @RestController]]
- [[26 - Why Spring Boot Feels Like Magic]]
