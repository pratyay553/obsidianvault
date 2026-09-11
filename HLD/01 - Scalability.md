---
tags: [hld]
---

# Scalability

## 🧠 Problem
SAloon's single app server handled 50 requests/sec fine at launch. After a marketing push it needs to handle 5,000 requests/sec. The server's CPU is pegged at 100% and requests start timing out.

## 🤔 Why does the problem exist
A single machine has a hard ceiling on CPU, RAM, disk I/O, and network throughput. Once traffic exceeds what one machine can do, no amount of code optimization saves you — you must add capacity, either by making that one machine bigger or by adding more machines.

## 💡 Idea
**Scalability** is a system's ability to handle growing load by adding resources. There are two directions to add resources:
- **Vertical scaling (scale up):** bigger machine — more CPU/RAM on the same box.
- **Horizontal scaling (scale out):** more machines — spread load across many boxes.

## 🏗️ Structure
```
Vertical Scaling                Horizontal Scaling
┌─────────┐                     ┌───────┐┌───────┐┌───────┐
│ 4 vCPU  │  ──▶  ┌─────────┐   │Server1││Server2││Server3│
│ 8GB RAM │       │ 32 vCPU │   └───────┘└───────┘└───────┘
└─────────┘       │ 128GB   │        ▲ needs a Load Balancer
                  └─────────┘        │ in front — see [[02 - Load Balancing]]
```

## ⚙️ Trade-offs
| | Vertical | Horizontal |
|---|---|---|
| Ceiling | Hits a hardware limit eventually | Effectively unlimited |
| Complexity | None — same single-machine code | Needs load balancing, session handling, distributed state |
| Failure | Single point of failure | One server dying doesn't take the system down |
| Cost curve | Gets expensive fast at the high end | Cheaper commodity machines, pay-as-you-grow |

The real-world path is usually: scale vertically first (it's free — no code changes), and switch to horizontal once you've maxed out what a single machine can reasonably give you, or once you need the fault-tolerance that comes with multiple machines regardless of load.

## 🏢 SAloon Example
SAloon's app servers are stateless — any server can handle any request because session data and slot-availability state live in the shared DB/cache, not in server memory. That statelessness is *what makes horizontal scaling possible at all*: if `BookingService` cached "which stylist is free" in a local in-memory `HashMap`, adding a second server would mean two servers with two different answers.

## 🎯 When should I use it?
Vertical scaling: quick wins, early-stage products, or truly single-threaded bottlenecks. Horizontal scaling: once you need fault tolerance, or once load genuinely exceeds one machine's ceiling.

## ⚠️ When NOT to use it?
Don't horizontally scale a system that isn't stateless yet — you'll just get inconsistent behavior across servers. Fix statelessness first (externalize session/state to a shared store), then scale out.

## 🔥 Interview Question
*"Why is statelessness a prerequisite for horizontal scaling?"*
→ If a server holds request-specific state in local memory, a later request routed to a *different* server won't see that state. Horizontal scaling requires any server to be able to serve any request identically, which means state must live outside the app server — in a DB, cache, or distributed session store.

## 🔗 Related
[[00 - Introduction to HLD]]
[[02 - Load Balancing]]
[[05 - Database Replication and Sharding]]
