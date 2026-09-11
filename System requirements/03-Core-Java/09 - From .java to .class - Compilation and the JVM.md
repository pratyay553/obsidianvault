---
title: "From .java to .class - Compilation and the JVM"
aliases: [JVM, javac, bytecode]
tags: [java, jvm, fundamentals]
difficulty: beginner
status: learning
---

# From `.java` to `.class` — Compilation and the JVM

> [!abstract] One-Line Definition
> The `javac` compiler turns human-written `.java` source files into platform-independent `.class` bytecode, which the **JVM** then runs.

---

## 1. Big Picture / Mental Model

```mermaid
flowchart LR
    A[Main.java] -->|javac| B[Main.class]
    B --> C[JVM]
    C --> D[RUN]
```

> [!quote] From the notes
> `Main.java` → `javac` → `Main.class` → **JVM** → `RUN`

## 2. Why It Matters

Java's famous promise is "write once, run anywhere." That's only possible because your source code isn't compiled straight to machine code for one specific CPU/OS — it's compiled to an intermediate format (**bytecode**) that any machine with a JVM can execute.

## 3. How It Works

1. You write `Main.java` — human-readable source code.
2. `javac Main.java` (the Java compiler) checks syntax and translates it into `Main.class` — **bytecode**, not machine code.
3. The **JVM** (Java Virtual Machine) loads `Main.class` and executes it, translating bytecode into actual machine instructions for whatever OS/CPU it's running on.

```text
Main.java  --[javac]-->  Main.class  --[loaded by]-->  JVM  --[executes]-->  RUN
```

## 4. Spring Boot / Java Notes

This exact pipeline is what happens (many times over) when you build and run a Spring Boot app:
- IntelliJ (or Maven) compiles all your `.java` files into `.class` files.
- Spring Boot packages them into a single runnable **JAR** (Java Archive) — see [[19 - Spring Initializr and Dependencies]] for JAR packaging.
- Running the JAR starts the JVM, which runs your `main()` method, which calls `SpringApplication.run(...)`.

## 5. Common Pitfalls & Misconceptions

- ❌ Thinking `.class` files are "the program" in the same sense as `.exe` on Windows — they're bytecode, not native machine code; the JVM is still required to run them.
- ❌ Confusing `java -version` (checks the *runtime*) with the compiler `javac` (does the *compiling*) — you need both installed to build and run Java projects.

## 6. Interview Questions

**Beginner**
1. What's the difference between `.java` and `.class` files?
2. Why does Java need a JVM instead of compiling directly to machine code?

## 7. Quick Revision
- `.java` → (compiled by `javac`) → `.class` (bytecode) → (run by) → JVM.
- This is the basis of Java's "write once, run anywhere."

## Related
- [[08 - Core Java Structure - Main Method]]
- [[19 - Spring Initializr and Dependencies]]
