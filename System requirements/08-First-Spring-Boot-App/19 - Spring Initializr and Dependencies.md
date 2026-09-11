---
title: "Spring Initializr and Dependencies"
aliases: [Spring Initializr, dependency, pom.xml, JAR]
tags: [spring-boot, maven, setup]
difficulty: beginner
status: learning
---

# Spring Initializr and Dependencies

> [!abstract] One-Line Definition
> **Spring Initializr** is a project generator that produces a ready-to-use Spring Boot project skeleton, so you don't have to hand-build folder structure, Maven config, and dependencies from scratch.

---

## 1. Big Picture / Mental Model

Imagine starting a Java web project completely from scratch — before writing a single line of business logic you'd need to:
```text
Create folder structure
Create Maven configuration
Select Spring Boot version
Select Java version
Add required dependencies
Configure package structure
Prepare project metadata
```
This setup is repetitive and nearly identical across every project. Spring Initializr exists purely to eliminate it.

## 2. Why It Matters

> In simple words: **Spring Initializr helps us create a ready-to-use Spring Boot project.**

It lets you choose: project type, programming language, Spring Boot version, project metadata, packaging type, Java version, and dependencies — then generates a downloadable, ready-to-open project.

## 3. How It Works

### What is a Dependency?

> A **dependency** is an external library required by our project.

Examples: Spring Web, MySQL Connector, Spring Data JPA, Spring Security, Lombok, Hibernate.

In Maven projects, dependencies are declared inside **`pom.xml`**.

### Choosing a Spring Boot version

| Version type | Meaning |
|---|---|
| **SNAPSHOT** | Work-in-progress, may contain bugs — not final |
| **RC** (Release Candidate) | Almost ready, still not the final stable release |
| **Stable release** | Tested, suitable for real development — **prefer this** |

### JAR Packaging

> **JAR** = Java Archive.

A Spring Boot JAR bundles your compiled application code *plus* everything needed to run it (including the embedded Tomcat server). This is why you can run a Spring Boot app with a single command instead of manually deploying to an external server.

## 4. Spring Boot / Java Notes

Once generated and opened in IntelliJ, a standard Spring Boot project looks like:

```text
src/main/java          → your Java source code
src/main/resources     → config files and static resources
pom.xml                → Maven configuration file (dependencies, project info)
application.properties → app configuration (e.g. server.port)
Main application class → contains main(), starts the app
```

See the full breakdown: [[20 - Anatomy of a Spring Boot Project]].

## 5. Common Pitfalls & Misconceptions

- ❌ Picking a SNAPSHOT version for a "real" project — reserve those for testing bleeding-edge features.
- ❌ Thinking you need to hand-edit `pom.xml` from an empty file — Spring Initializr generates a correct starting point; you only add/remove dependencies as needed.

## 6. Interview Questions

**Beginner**
1. What problem does Spring Initializr solve?
2. What is a "dependency" in the context of a Java project?
3. Where are dependencies declared in a Maven project?

## 7. Quick Revision
- Spring Initializr = project generator, removes repetitive setup.
- Dependency = external library your project needs, declared in `pom.xml`.
- Prefer **stable releases** over SNAPSHOT/RC for real projects.
- JAR = Java Archive; Spring Boot JARs are self-contained and runnable.

## Related
- [[09 - From .java to .class - Compilation and the JVM]]
- [[20 - Anatomy of a Spring Boot Project]]
- [[17 - Spring Boot vs Spring Framework]]
