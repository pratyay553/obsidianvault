---
title: "DNS and Default Ports"
aliases: [DNS, Default Ports]
tags: [networking, dns, http]
difficulty: beginner
status: learning
---

# DNS and Default Ports

> [!abstract] One-Line Definition
> DNS translates a human-friendly domain name into an IP address; the URL's protocol (`http`/`https`) tells the browser which **default port** to use if none is specified.

---

## 1. Big Picture / Mental Model

When you type a normal website URL like `www.coderarmy.in`, you never mention a port number. That's because:

1. **DNS** resolves the domain name to an IP address.
2. The **protocol** (`http://` or `https://`) decides the default port.

```text
https://www.coderarmy.in
   |             |
 port 443    DNS resolves this to an IP
```

## 2. Why It Matters

Without DNS, you'd have to memorize IP addresses instead of names. Without default ports, every URL on the internet would need to end in `:80` or `:443`.

## 3. How It Works

| Protocol | Default Port |
|---|---|
| `http` | 80 |
| `https` | 443 |

- `https://www.coderarmy.in` → browser automatically assumes port **443**.
- `http://www.coderarmy.in` → browser automatically assumes port **80**.
- A local Spring Boot app usually runs on port **8080**, which is *not* a browser default, so you must mention it manually: `http://localhost:8080`.

> [!quote] Summary from the notes
> DNS resolves `www.coderarmy.in` to an IP address.
> HTTPS tells the browser to use port 443 by default.

## 4. Spring Boot / Java Notes

This is exactly why every example in this vault types `http://localhost:8080/hello` in full — 8080 is Spring Boot's default embedded-Tomcat port, and browsers have no built-in default for it. Compare with [[25 - Changing the Server Port]].

## 5. Common Pitfalls & Misconceptions

- ❌ Assuming DNS also determines the port — it doesn't; DNS only resolves *names to IPs*. The port comes from the protocol default or an explicit `:port` in the URL.
- ❌ Forgetting `:8080` when testing a local Spring Boot app and wondering why the browser can't connect (it silently tries port 80 instead).

## 6. Interview Questions

**Beginner**
1. What does DNS actually resolve — a name to what?
2. What are the default ports for HTTP and HTTPS?

## 7. Quick Revision
- DNS: domain name → IP address.
- Port: decided by URL protocol default, or explicitly stated (`:8080`).
- `https` → 443, `http` → 80, Spring Boot dev server → 8080 (must be explicit).

## Related
- [[02 - IP Address vs Port Number]]
- [[03 - Localhost and 127.0.0.1]]
- [[05 - What is HTTP and HTTPS]]
