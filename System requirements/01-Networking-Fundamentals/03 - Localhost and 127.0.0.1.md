---
title: "Localhost and 127.0.0.1"
aliases: [localhost, "127.0.0.1"]
tags: [networking, fundamentals]
difficulty: beginner
status: learning
---

# Localhost and 127.0.0.1

> [!abstract] One-Line Definition
> `localhost` is a special hostname that always points back to **your own machine** — it never leaves your computer.

---

## 1. Big Picture / Mental Model

Students often confuse `localhost`, IP address, and port number. Think of it this way:

```text
localhost   = "myself" (your own machine)
127.0.0.1   = the numeric address for "myself"
:8080       = which application on "myself" to talk to
```

> [!quote] From the notes
> `localhost = your own machine`
> `localhost` is equivalent to `127.0.0.1`

## 2. Why It Matters

When developing, you don't have a public domain yet — you just want your browser to talk to the server running *on the very same laptop*. `localhost` is the universally-agreed-upon shortcut for "this machine, whatever it is."

## 3. How It Works

```text
localhost:8080
```
means:
```text
Send the request to my own machine on port 8080.
```

If your Spring Boot app is running on port `8080`, the browser reaches it with:

```text
http://localhost:8080
```

This combines three ideas you now know:
- `http://` → protocol → see [[05 - What is HTTP and HTTPS]]
- `localhost` → this machine → `127.0.0.1`
- `:8080` → the specific application (port) → see [[02 - IP Address vs Port Number]]

## 4. Spring Boot / Java Notes

The very first endpoint built in these notes is reached at:

```text
http://localhost:8080/hello
```

See the full walkthrough: [[23 - Creating Our First Endpoint - @GetMapping]].

## 5. Common Pitfalls & Misconceptions

- ❌ Thinking `localhost` is "the internet" — it never leaves your machine; another computer on your network **cannot** reach your app via `localhost`, it needs your machine's real IP.
- ❌ Forgetting the port — `localhost` alone (no port) implicitly uses the browser's default port assumption (80 for HTTP), not 8080.

## 6. Interview Questions

**Beginner**
1. What is `localhost` and what IP address does it map to?
2. Why do local development servers usually need a port number in the URL but production websites don't?

## 7. Quick Revision
- `localhost` = your own machine = `127.0.0.1`
- `localhost:8080` = "my own machine, on the app listening at port 8080"

## Related
- [[02 - IP Address vs Port Number]]
- [[04 - DNS and Default Ports]]
- [[23 - Creating Our First Endpoint - @GetMapping]]
