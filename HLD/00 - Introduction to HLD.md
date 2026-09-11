---
tags: [hld]
---

# Introduction to High-Level Design (HLD)

## 🧠 Problem
Given "design WhatsApp" or "design an e-commerce checkout," I don't know where to start. Jumping straight into classes and methods (LLD) produces a system that looks clean on paper but falls over the moment real traffic, real failures, or real data volume hit it.

## 🤔 Why does the problem exist
LLD answers "how do I structure this code so it's maintainable?" It assumes a single machine, a happy-path request, and a fixed load. Real systems run on many machines, fail constantly (disks die, networks partition, servers restart), and have load that grows 100x between launch and success. Those are HLD's concerns, and they change the *shape* of the system, not just its code — no amount of good class design fixes a single database that can't take the write volume.

## 💡 Idea
HLD is the discipline of deciding a system's **major components and how they talk to each other**, before worrying about the code inside any one component. It answers: how many servers, how do they find each other, where does data live, what happens when a piece dies, how does the system grow. LLD then fills in each box with clean, SOLID-compliant code.

## 🏗️ Structure
```
HLD asks:                         LLD asks:
┌─────────────┐                   ┌───────────────────┐
│  Client      │                   │ class BookingSvc {  │
└──────┬───────┘                   │   PricingStrategy  │
       │                           │   BookingRepo      │
┌──────▼───────┐  ┌────────────┐   │   NotificationSvc  │
│ Load Balancer│─▶│ App Servers │   │ }                  │
└──────────────┘  └──────┬─────┘   └────────────────────┘
                          │
                 ┌────────▼────────┐
                 │ DB / Cache / MQ │
                 └─────────────────┘
```
Everything above the dotted line is HLD. Everything inside one box is LLD — this is why the two vaults connect: [[01 - OCP]] and friends live *inside* the "App Servers" box.

## ⚙️ How to approach an HLD problem
1. **Clarify requirements** — functional (what must it do) and non-functional (how many users, how much data, read-heavy or write-heavy, latency budget, consistency needs).
2. **Estimate scale** — back-of-envelope: requests/sec, storage/year, bandwidth. This decides whether you need one DB or ten.
3. **Define the API** at a high level — what does the client call.
4. **Draw the high-level components** — clients, load balancers, services, databases, caches, queues.
5. **Deep-dive the 1–2 hardest parts** — usually data storage/partitioning and one tricky flow (e.g. "how do we guarantee only one booking per slot").
6. **Identify bottlenecks and single points of failure**, then fix with the tools in this vault: [[01 - Scalability]], [[02 - Load Balancing]], [[03 - Caching]], etc.

## 🏢 SAloon Example
SAloon (the salon booking app) starts as one server + one Postgres DB — that's a valid HLD for 100 users. The HLD work only kicks in when the requirements change: "SAloon now needs to support 500 salons nationwide with real-time slot availability." That single sentence is what forces load balancers, read replicas, and caching into the picture — the LLD (`BookingService`, `PricingStrategy`) barely has to change at all.

## 🎯 When should I use it?
Any time the question is about the system's *shape* rather than a class's *behavior*: how it scales, survives failure, stores data, or how components communicate. Always start here before LLD in an interview or a real design doc.

## ⚠️ When NOT to use it?
Don't over-engineer HLD for a system that will only ever run on one machine for a handful of users — a single server with a single DB is a perfectly valid HLD. Distributed-systems machinery has a real operational cost; only reach for it when the estimated scale actually demands it.

## 🔥 Interview Question
*"What's the difference between HLD and LLD?"*
→ HLD decides the components and their interactions (services, databases, queues, how they scale and survive failure). LLD decides the classes and methods inside one of those components (SOLID, design patterns). HLD is the floor plan of a building; LLD is the wiring inside one room.

## 🔗 Related
[[01 - Scalability]]
[[00 - SRP]]
