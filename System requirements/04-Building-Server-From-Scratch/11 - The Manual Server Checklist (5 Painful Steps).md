---
title: "The Manual Server Checklist (5 Painful Steps)"
aliases: [manual HTTP server, BR checklist]
tags: [java, networking, servlets]
difficulty: intermediate
status: learning
---

# The Manual Server Checklist — 5 Painful Steps

> [!abstract] One-Line Definition
> If you build your own server with raw `java.net`, here are the five things *you* would have to implement by hand — everything Tomcat and Spring Boot do automatically.

---

## 1. Big Picture / Mental Model

```text
GET /courses
HOST: localhost:8080 (127.0.0.1)
      |
      v
   java.net
      |
      v
   [ 5 manual steps ]
```

## 2. Why It Matters

This is the single best argument for *why Spring Boot exists*. Once you see everything you'd have to build by hand, "Spring Boot hides a lot of complexity" (from Lecture 2) stops being an abstract sentence and becomes very concrete.

## 3. How It Works — the 5 steps, from the notes

1. **Read I/P streams** — read the raw incoming bytes from the socket's `InputStream`.
2. **Parse the request manually** — split those bytes into method, path, headers, body yourself.
3. **Map a method to an endpoint** — essentially hand-rolling routing:
   ```java
   if (endpoint.equals("courses")) {
       // handle /courses
   } else {
       // handle everything else / 404
   }
   ```
4. **Manually build the HTTP response** — construct the status line, headers, and body as raw text/bytes and write them to the `OutputStream`.
5. **Multi-threading** — handle multiple clients *at the same time* (otherwise your `while(true)` loop can only serve one request at a time before it can accept the next).

## 4. Spring Boot / Java Notes

Compare each manual step to what Spring Boot gives you automatically:

| Manual step | Spring Boot equivalent |
|---|---|
| Read raw bytes | Handled inside embedded Tomcat |
| Parse request manually | `HttpServletRequest` object, fully parsed |
| Map endpoint via `if/else` | `@GetMapping("/courses")` |
| Manually build response | `return "some string";` → auto-wrapped into an HTTP response |
| Multi-threading | Tomcat's built-in thread pool manages this |

## 5. Common Pitfalls & Misconceptions

- ❌ Assuming step 3 ("map a method to an endpoint") is trivial at scale — with dozens of endpoints, hand-written `if/else` chains become unmanageable. This is exactly the problem [[13 - What is a Servlet|Servlets]] and later Spring MVC's routing solve.
- ❌ Skipping multi-threading in a toy server and being surprised when concurrent users block each other.

## 6. Interview Questions

**Intermediate**
1. List the manual steps required to build an HTTP server from raw sockets.
2. Why is multi-threading necessary even for a simple server?

## 7. Quick Revision
- 5 manual steps: read bytes → parse request → map endpoint → build response → multi-thread.
- Every one of these is exactly what a Servlet Container automates.

## Related
- [[10 - Building a Raw Server with java.net]]
- [[13 - What is a Servlet]]
- [[26 - Why Spring Boot Feels Like Magic]]
