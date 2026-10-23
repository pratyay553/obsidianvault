---
tags: [networking, ip, port, fundamentals]
---

# IP Address vs Port Number

## The problem this solves

Your laptop is one machine on a network, but it's running **many applications simultaneously**:

```
Chrome
WhatsApp
Zoom
Spotify
Spring Boot application3
```

If a packet of data arrives at your machine's IP address, **how does the operating system know which of these five applications it's meant for?**

This is exactly the problem **ports** solve.

## Definitions

> [!abstract] IP Address
> Identifies **the machine** on a network.
> Example: `192.168.1.10`

> [!abstract] Port Number
> Identifies **the application running inside that machine**.
> Example: `192.168.1.10:8080`

So the combination `IP:PORT` uniquely identifies *one running program on one machine*.

## Visual model

```
Internet
   |
   v
192.168.1.10
--------------------------------
| Chrome           Port 5000  |
| WhatsApp         Port 5222  |
| Zoom             Port 8801  |
| Spring Boot App  Port 8080  |
--------------------------------
```

- The **IP address** gets the data to the *correct machine*.
- The **port number** gets the data to the *correct application* on that machine.

> [!important]
> This is also why, when you later run a Spring Boot app, you'll see log output like `Tomcat started on port 8080` — see [[14 - Servlet Containers and Tomcat]]. Tomcat is *literally the application listening on that port*, exactly like Chrome listens on port 5000 in the diagram above.

## Why 8080 specifically?

`8080` isn't magic — it's just Spring Boot's **default** configured port for its embedded server. You can change it any time. See [[25 - Changing the Server Port]].

## Related
- [[01 - Client, Server & The Request-Response Cycle]]
- [[03 - Localhost and 127.0.0.1]]
- [[04 - DNS and Default Ports]]
- [[25 - Changing the Server Port]]
