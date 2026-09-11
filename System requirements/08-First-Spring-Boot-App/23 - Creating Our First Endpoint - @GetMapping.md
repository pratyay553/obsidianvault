---
title: "Creating Our First Endpoint - @GetMapping"
aliases: ["@GetMapping", First Endpoint]
tags: [spring-boot, spring-mvc, java]
difficulty: beginner
status: learning
---

# Creating Our First Endpoint — `@GetMapping`

> [!abstract] One-Line Definition
> `@GetMapping("/hello")` tells Spring that a specific Java method should run whenever an HTTP **GET** request arrives at `/hello`.

---

## 1. Big Picture / Mental Model

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello World";
    }
}
```

Opening `http://localhost:8080/hello` in a browser displays:
```text
Hello World
```

## 2. Why It Matters

This tiny piece of code is the payoff of everything else in this vault — [[01 - Client, Server & The Request-Response Cycle|client/server]], [[02 - IP Address vs Port Number|ports]], [[05 - What is HTTP and HTTPS|HTTP methods]], [[14 - Servlet Containers and Tomcat|Tomcat]], and [[22 - Controllers and @RestController|controllers]] all converge here into eight lines of working code.

## 3. How It Works — line by line

| Line | Meaning |
|---|---|
| `@RestController` | Tells Spring this class can handle HTTP requests. See [[22 - Controllers and @RestController]]. |
| `@GetMapping("/hello")` | Tells Spring this method should run when a **GET** request comes to `/hello`. |
| `sayHello()` | The Java method that actually gets executed. |
| `return "Hello World";` | The response sent back to the browser. |

## 4. Spring Boot / Java Notes

`@GetMapping` is one of several HTTP-method-specific mapping annotations that mirror the [[06 - Anatomy of an HTTP Request|HTTP methods]] you already know:

| Annotation | HTTP method |
|---|---|
| `@GetMapping` | `GET` |
| `@PostMapping` | `POST` |
| `@PutMapping` | `PUT` |
| `@PatchMapping` | `PATCH` |
| `@DeleteMapping` | `DELETE` |

## 5. Common Pitfalls & Misconceptions

- ❌ Typing the wrong path (e.g. `/Hello` vs `/hello`) — URL paths are case-sensitive by default.
- ❌ Testing a `@PostMapping` endpoint directly in a browser address bar — browsers issue `GET` requests by default; you need a tool like Postman (or a form/JS) to send `POST`.
- ❌ Seeing `404 Not Found` and not knowing where to start debugging — see the full checklist in [[27 - Why You Still Need to Learn the Internals]].

## 6. Interview Questions

**Beginner**
1. What does `@GetMapping("/hello")` do?
2. What will the browser display at `http://localhost:8080/hello` given the code above?

## 7. Quick Revision
- `@GetMapping("/path")` maps a GET request at that path to the annotated method.
- The method's return value becomes the HTTP response body.

## Related
- [[22 - Controllers and @RestController]]
- [[24 - Complete Request Flow Walkthrough]]
- [[06 - Anatomy of an HTTP Request]]
