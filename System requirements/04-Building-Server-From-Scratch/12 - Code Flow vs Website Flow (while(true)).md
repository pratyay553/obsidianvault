---
title: "Code Flow vs Website Flow (while(true))"
aliases: [Code Flow, Website Flow]
tags: [java, servlets, fundamentals]
difficulty: beginner
status: learning
---

# Code Flow vs Website Flow

> [!abstract] One-Line Definition
> A normal Java program runs **once and exits**; a server must run **forever**, continuously waiting for the next request.

---

## 1. Big Picture / Mental Model

> [!quote] From the notes — two side-by-side flows

**Code Flow** (a normal program):
```text
START
RUN
STOP
EXIT
```

**Website Flow** (a server):
```text
START
RUN
Wait for Request.
Give Response.
Keep Running.
```

## 2. Why It Matters

This distinction explains *why* servers are structured so differently from the small console programs you first learn Java with. A server that stopped after handling one request would be useless — the next visitor would find nothing listening.

## 3. How It Works

The "Keep Running" behavior is implemented, at the code level, with an infinite loop:

```java
while (true) {
    // wait for a request
    // handle it
    // send a response
    // loop back and wait again
}
```

This is a direct continuation of [[10 - Building a Raw Server with java.net]] — the `server.accept()` call blocks (waits) inside this loop until the next client connects.

## 4. Spring Boot / Java Notes

When you run a Spring Boot app and see `Tomcat started on port 8080` in the console, and the process just... sits there — that's this exact loop, running inside Tomcat, keeping the JVM alive and listening. See [[14 - Servlet Containers and Tomcat]].

## 5. Common Pitfalls & Misconceptions

- ❌ Thinking a "hanging" Spring Boot process in your terminal (no more log output after startup) means it's broken — it's actually idle and *waiting* for requests, which is correct behavior.

## 6. Interview Questions

**Beginner**
1. Why can't a server use a normal "start → run → stop → exit" flow?

## 7. Quick Revision
- Normal programs: run once, exit.
- Servers: run forever, waiting for the next request via `while(true)`.

## Related
- [[10 - Building a Raw Server with java.net]]
- [[14 - Servlet Containers and Tomcat]]
