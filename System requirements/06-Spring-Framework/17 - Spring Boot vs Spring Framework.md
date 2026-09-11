---
title: "Spring Boot vs Spring Framework"
aliases: [Spring Boot]
tags: [spring, spring-boot]
difficulty: beginner
status: learning
---

# Spring Boot vs Spring Framework

> [!abstract] One-Line Definition
> **Spring Framework** is the underlying set of modules (Core, MVC, Data, Security...); **Spring Boot** is an opinionated layer on top that auto-configures those modules so you can start building immediately.

---

## 1. Big Picture / Mental Model

> [!quote] From Lecture 2
> "The goal is not to understand every internal concept immediately. That is intentional. Spring Boot hides a lot of complexity in the beginning so that developers can quickly start building applications."

## 2. Why It Matters

Before Spring Boot, using the Spring Framework meant a lot of manual XML/Java configuration: registering a `DispatcherServlet`, configuring a server, wiring beans by hand. Spring Boot's entire purpose is to eliminate that setup tax.

## 3. How It Works

Spring Boot gives you, out of the box:
- An **embedded Tomcat server** (no separate install — see [[14 - Servlet Containers and Tomcat]])
- **Auto-configuration** — sensible defaults inferred from your dependencies
- A **starter dependency system** (`spring-boot-starter-web`, etc. — see [[19 - Spring Initializr and Dependencies]])
- **One command to run** — no manual deployment to an external server

```text
Browser
  |
  | Request
  v
Spring Boot Application
  |
  | Response
  v
Browser
```

## 4. Spring Boot / Java Notes

The entire first working app in these notes is just this:

```java
@SpringBootApplication
public class FirstSpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(FirstSpringBootApplication.class, args);
    }
}
```

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String sayHello() {
        return "Hello World";
    }
}
```

Two small classes — yet behind them, Spring Boot: created an application context, started embedded Tomcat, configured Spring MVC, detected the controller, and registered the `/hello` endpoint. See [[26 - Why Spring Boot Feels Like Magic]] for the full list.

## 5. Common Pitfalls & Misconceptions

- ❌ Thinking Spring Boot is "a different framework" from Spring — it's a *distribution/configuration layer* over the same Spring Framework modules.
- ❌ Believing "fast to start" means "nothing to learn" — see [[27 - Why You Still Need to Learn the Internals]].

## 6. Interview Questions

**Beginner**
1. What problem does Spring Boot solve that plain Spring Framework doesn't?
2. What comes bundled with Spring Boot that you'd otherwise have to configure by hand?

## 7. Quick Revision
- Spring Framework = the modules.
- Spring Boot = auto-configuration + embedded server + starters, on top of those modules.

## Related
- [[15 - Spring Framework Overview]]
- [[19 - Spring Initializr and Dependencies]]
- [[26 - Why Spring Boot Feels Like Magic]]
