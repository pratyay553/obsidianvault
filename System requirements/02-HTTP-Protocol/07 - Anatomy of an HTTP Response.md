---
title: "Anatomy of an HTTP Response"
aliases: [HTTP Response]
tags: [http, networking]
difficulty: beginner
status: learning
---

# Anatomy of an HTTP Response

> [!abstract] One-Line Definition
> An HTTP response is a plain-text message with three parts: **status code, headers, and a body**.

---

## 1. Big Picture / Mental Model

> [!quote] From the notes
> HTTP Response:
> - Status code (200 OK)
> - Headers
> - Body

## 2. Why It Matters

The client needs a quick, standardized way to know: *did it work?* (status code), *what kind of data is coming back?* (headers), and *what's the actual data?* (body).

## 3. How It Works

### Example — response to the login request

```http
200 OK
Content-Type: application/json

{
  "message": "login successful"
}
```

### Status codes at a glance

| Range | Meaning | Example |
|---|---|---|
| 2xx | Success | `200 OK` |
| 3xx | Redirection | `301 Moved Permanently` |
| 4xx | Client error | `404 Not Found` |
| 5xx | Server error | `500 Internal Server Error` |

`404 Not Found` shows up constantly while debugging Spring Boot apps — see [[27 - Why You Still Need to Learn the Internals]] for the full checklist of what to investigate when you see one.

## 4. Spring Boot / Java Notes

When a `@GetMapping` method returns a `String`, Spring Boot automatically wraps it into a full HTTP response — status `200`, a `Content-Type` header, and your returned value as the body:

```java
@GetMapping("/hello")
public String sayHello() {
    return "Hello World"; // becomes the response BODY
}
```

See [[23 - Creating Our First Endpoint - @GetMapping]] for the full example.

## 5. Common Pitfalls & Misconceptions

- ❌ Assuming "no error shown" means "200 OK" — always check the actual status code, especially in tools like Postman or browser dev tools.
- ❌ Ignoring `Content-Type` on the response — a client (e.g. a frontend app) needs it to know whether to parse the body as JSON, HTML, plain text, etc.

## 6. Interview Questions

**Beginner**
1. What three parts make up an HTTP response?
2. What does a `404` status code mean vs a `200`?

## 7. Quick Revision
- Response = Status Code + Headers + Body.
- `200 OK` = success; `404` = not found; `500` = server error.

## Related
- [[06 - Anatomy of an HTTP Request]]
- [[24 - Complete Request Flow Walkthrough]]
