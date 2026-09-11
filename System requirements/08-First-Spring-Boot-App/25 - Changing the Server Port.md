---
title: "Changing the Server Port"
aliases: [server.port, application.properties]
tags: [spring-boot, tomcat, configuration]
difficulty: beginner
status: learning
---

# Changing the Server Port

> [!abstract] One-Line Definition
> Spring Boot's embedded Tomcat listens on port `8080` by default — you can change this in one line via `application.properties`.

---

## 1. Big Picture / Mental Model

```properties
server.port=9090
```

After changing this and restarting the application, the app runs on `http://localhost:9090`, and the endpoint becomes `http://localhost:9090/hello`.

## 2. Why It Matters

Multiple apps can't share the same port (see [[02 - IP Address vs Port Number]]) — being able to reconfigure the port is essential when running multiple services locally, or when a hosting environment assigns a specific port.

## 3. How It Works

> [!important] Important point from the notes
> Spring Boot **reads** the configuration, but the **embedded Tomcat server** is what actually listens on the configured port.

So the flow is:
```text
application.properties (server.port=9090)
        ↓
Spring Boot reads this config at startup
        ↓
Embedded Tomcat binds to port 9090 instead of 8080
```

## 4. Spring Boot / Java Notes

This single config line is a perfect small example of the layering discussed in [[24 - Complete Request Flow Walkthrough]]: Spring Boot itself doesn't "listen" on anything — it's Tomcat (embedded inside it) that does, and Spring Boot's job is just to configure Tomcat correctly before starting it.

## 5. Common Pitfalls & Misconceptions

- ❌ Forgetting to restart the app after editing `application.properties` — config changes aren't hot-reloaded by default.
- ❌ Picking a port already in use by another process — this causes a startup failure (`Port already in use`), not a silent fallback to a different port.

## 6. Interview Questions

**Beginner**
1. Where do you configure the port a Spring Boot app runs on?
2. What actually "listens" on that port — Spring Boot itself, or something else?

## 7. Quick Revision
- Default port: `8080`.
- Change via `server.port=<port>` in `application.properties`.
- Tomcat (not Spring Boot itself) is what binds to the configured port.

## Related
- [[02 - IP Address vs Port Number]]
- [[14 - Servlet Containers and Tomcat]]
- [[20 - Anatomy of a Spring Boot Project]]
