---
title: "Controllers and @RestController"
aliases: ["@RestController", Controller]
tags: [spring-boot, spring-mvc, java]
difficulty: beginner
status: learning
---

# Controllers and `@RestController`

> [!abstract] One-Line Definition
> A **Controller** is the entry point for incoming web requests in a Spring app; `@RestController` is the annotation that tells Spring a class can handle HTTP requests and return data directly as the response.

---

## 1. Big Picture / Mental Model

> [!quote] From the notes
> "A controller is like the **receptionist** of our application."

```text
Browser sends request to /hello
        ↓
Controller receives the request
        ↓
Controller returns "Hello World"
```

## 2. Why It Matters

Somewhere, some piece of code needs to be the *first thing* that runs when a request for a specific URL arrives. In Spring MVC, that's the Controller's job — it decides what response should be sent back.

## 3. How It Works

```java
@RestController
public class HelloController {

}
```

`@RestController` tells Spring:
> "This class can handle HTTP requests and return data directly as a response."

It's an annotation — metadata that makes the class *eligible* to receive web requests when Spring scans for components at startup.

## 4. Spring Boot / Java Notes

`@RestController` is actually a combination of two ideas under one annotation:
- `@Controller` — marks the class as a Spring MVC controller (eligible to handle requests).
- `@ResponseBody` — tells Spring to serialize whatever the method returns *directly* into the HTTP response body, rather than treating it as a view name (like a JSP page) to render.

This is why `return "Hello World";` in the next note becomes literal response text, not a lookup for an HTML template.

## 5. Common Pitfalls & Misconceptions

- ❌ Forgetting the annotation entirely — an un-annotated class, no matter how correctly its methods are written, is invisible to Spring's request routing.
- ❌ Confusing `@RestController` with plain `@Controller` — `@Controller` alone expects you to return *view names* to render templates, not raw data.

## 6. Interview Questions

**Beginner**
1. What does `@RestController` tell Spring?
2. Use an analogy to explain what a Controller does.

**Intermediate**
1. What two responsibilities does `@RestController` combine?

## 7. Quick Revision
- Controller = entry point for web requests ("receptionist").
- `@RestController` = handles HTTP requests + returns data directly as the response.

## Related
- [[21 - The Main Application Class]]
- [[23 - Creating Our First Endpoint - @GetMapping]]
- [[16 - Spring Core - DI, IoC, Beans]]
