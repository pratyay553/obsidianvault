---
tags: [hld]
---

# Load Balancing

## 🧠 Problem
SAloon now runs on five app servers. Clients need one URL to hit — they shouldn't need to know there are five servers, and if one server dies, traffic needs to stop going to it automatically.

## 🤔 Why does the problem exist
Horizontal scaling ([[01 - Scalability]]) only helps if traffic is actually *spread* across the new machines. Without something in front deciding which server handles which request, either every client hardcodes one server's address (defeating the point of having more than one) or traffic piles unevenly onto whichever server happens to be easiest to reach.

## 💡 Idea
A **load balancer** sits between clients and app servers, distributing incoming requests across the healthy servers behind it, and removing servers from rotation when they stop responding to health checks.

## 🏗️ Structure
```
Client ──▶ Load Balancer ──▶ Server 1 (healthy)
                        ├──▶ Server 2 (healthy)
                        └──▶ Server 3 (✗ failed health check — skipped)
```

## ⚙️ Common algorithms
- **Round robin** — requests go to servers in rotation. Simple, assumes servers are roughly equal.
- **Least connections** — send to whichever server currently has the fewest active requests. Better when requests vary in cost.
- **IP hash / consistent hashing** — same client IP always lands on the same server, useful when some state is still tied to a server (sticky sessions). See [[07 - Consistent Hashing]].
- **Weighted** — bigger servers get proportionally more traffic.

Load balancers also operate at different layers: **L4** (routes on IP/TCP, fast, doesn't see HTTP content) vs **L7** (routes on HTTP headers/path — e.g. `/api/payments` to one service, `/api/bookings` to another).

## 🏢 SAloon Example
SAloon puts an L7 load balancer in front of its app servers, routing `/bookings/*` and `/payments/*` to different backend clusters, and running a `/health` check against every server every few seconds. When `Server 3` fails two checks in a row, the load balancer stops sending it traffic — the client never sees a failure, they just experience slightly higher load on the remaining servers.

## 🎯 When should I use it?
As soon as there's more than one instance of a service — which, per [[01 - Scalability]], is as soon as you scale horizontally at all.

## ⚠️ When NOT to use it?
A single-server deployment doesn't need one — it's pure overhead. Also watch out: the load balancer itself can become a single point of failure, so production setups usually run at least two load balancers behind DNS or a floating IP.

## 🔥 Interview Question
*"How does a load balancer know a server is down?"*
→ Active health checks: it periodically pings a lightweight endpoint (e.g. `GET /health`) on each server and stops routing to any server that fails a threshold number of consecutive checks, re-adding it once it starts passing again.

## 🔗 Related
[[01 - Scalability]]
[[07 - Consistent Hashing]]
[[04 - CDN]]
