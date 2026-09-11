---
tags: [hld]
---

# CAP Theorem & Consistency Patterns

## 🧠 Problem
SAloon's booking data is now replicated across three regions ([[05 - Database Replication and Sharding]]). A network glitch briefly cuts the Mumbai replica off from the others. Should Mumbai keep accepting bookings (and risk showing stale/conflicting data), or refuse writes until it can confirm with the others (and risk turning away real customers)?

## 🤔 Why does the problem exist
In any distributed system, network partitions *will* happen eventually — cables get cut, data centers lose connectivity, packets get dropped. When that happens, a node has to choose: answer requests using only the data it can see (possibly wrong/stale), or refuse to answer until it can confirm correctness with the rest of the system (possibly unavailable). Both copies of the data can't simultaneously stay perfectly in sync *and* keep serving every request while they can't talk to each other.

## 💡 Idea
The **CAP theorem** states a distributed system can only guarantee two of these three at once:
- **Consistency (C):** every read gets the most recent write, everywhere.
- **Availability (A):** every request gets a (non-error) response, even if it's not the latest data.
- **Partition tolerance (P):** the system keeps working despite network partitions between nodes.

Since real networks *do* partition, **P is not optional** — the real-world choice is between **CP** (consistent but may refuse requests during a partition) and **AP** (available but may serve stale data during a partition).

## 🏗️ Structure
```
                CAP triangle
                    C
                   / \
                  /   \
                 /     \
                CP------AP
                    P (mandatory in practice)

CP system: partition happens → node refuses writes it can't confirm → correctness over uptime
AP system: partition happens → node keeps serving → uptime over correctness (fixed later)
```

## ⚙️ Consistency patterns in practice
- **Strong consistency:** every read reflects the latest write immediately (needs coordination — e.g. quorum reads/writes, or routing all reads to the primary). Costs latency.
- **Eventual consistency:** replicas converge to the same value *eventually*, but a read right after a write might see stale data briefly. Common in AP systems (DNS, most NoSQL stores by default).
- **Quorum (N/W/R):** with N replicas, require W replicas to ack a write and R replicas to agree on a read, such that W + R > N — tunable middle ground between full consistency and full availability.

## 🏢 SAloon Example
SAloon treats **booking a slot** as CP: during a network partition, the booking service in the affected region refuses new bookings for shared slots rather than risk two people double-booking the same chair — losing a few bookings temporarily beats an unfixable double-booking. But **browsing salon listings** is AP: if a region can't confirm it has the latest listing data, it serves what it has rather than showing an error page — a slightly stale listing is a fine trade-off for staying up.

## 🎯 When should I use it?
Use CAP as a framing question whenever designing any feature that touches replicated/distributed data: "if the network splits right now, does this feature need to stay correct, or does it need to stay up?" The answer differs per feature, not per whole system — as the SAloon example shows.

## ⚠️ When NOT to use it?
Don't apply CAP theorem reasoning to a single-node, non-distributed system — there's no partition to tolerate, so the trade-off doesn't exist. It's specifically a distributed-systems concept.

## 🔥 Interview Question
*"Why is 'CA' (consistent and available, no partition tolerance) not a real option?"*
→ It would only work on a network that never partitions, which doesn't exist for any system with more than one node communicating over a real network. In practice you always have to plan for P, which forces the actual choice to be between CP and AP.

## 🔗 Related
[[05 - Database Replication and Sharding]]
[[03 - Caching]]
