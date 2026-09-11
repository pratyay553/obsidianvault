---
title: "Anatomy of a Spring Boot Project"
aliases: [Spring Boot Project Structure]
tags: [spring-boot, maven, setup]
difficulty: beginner
status: learning
---

# Anatomy of a Spring Boot Project

> [!abstract] One-Line Definition
> A generated Spring Boot project follows a fixed, predictable folder layout — knowing what lives where means you can navigate *any* Spring Boot codebase, not just your own.

---

## 1. Big Picture / Mental Model

```text
my-project/
├── src/main/java/          ← your Java source code
├── src/main/resources/     ← configuration files & static resources
│   └── application.properties
└── pom.xml                 ← Maven configuration file
```

## 2. Why It Matters

Development environment setup (IntelliJ IDEA is recommended in the notes for its Java/Maven/Spring Boot support) plus a consistent project layout means every Spring Boot developer can jump into any project and immediately know where the entry point, config, and dependencies live.

## 3. How It Works — each piece explained

| Path | Purpose |
|---|---|
| `src/main/java` | Contains your Java source code — controllers, services, entities, and the main application class. |
| `src/main/resources` | Contains configuration files and static resources (like `application.properties`, templates, static assets). |
| `pom.xml` | The Maven configuration file — contains project info and [[19 - Spring Initializr and Dependencies\|dependencies]]. |
| `application.properties` | Configures your Spring Boot app — e.g., changing the server port (see [[25 - Changing the Server Port]]). |
| Main Application Class | The class containing `main()`, from where the app starts — see [[21 - The Main Application Class]]. |

## 4. Spring Boot / Java Notes

Before creating a project, verify Java is installed:
```bash
java -version
```
If it's missing, install it first — nothing downstream (compilation, running, IntelliJ tooling) works without it.

## 5. Common Pitfalls & Misconceptions

- ❌ Putting config values directly in Java code instead of `application.properties` — this makes environment-specific changes (like ports, DB credentials) require a recompile instead of a config edit.
- ❌ Placing source files outside `src/main/java` — Maven's build process expects this exact convention-based layout.

## 6. Interview Questions

**Beginner**
1. What is the purpose of `src/main/resources`?
2. What does `pom.xml` contain?

## 7. Quick Revision
- `src/main/java` = code, `src/main/resources` = config/static files, `pom.xml` = Maven/dependencies.
- `application.properties` configures runtime behavior (e.g., port).

## Related
- [[19 - Spring Initializr and Dependencies]]
- [[21 - The Main Application Class]]
- [[25 - Changing the Server Port]]
