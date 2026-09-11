---
title: "Load Balancer"
aliases:
  - Load Balancing
  - Load Balancer
tags:
  - system-design
  - computer-networks
  - backend
  - distributed-systems
  - spring-boot
  - devops
difficulty: beginner
status: learning
---

# Load Balancer

> [!abstract] One-Line Definition
> A **Load Balancer** is a component that receives incoming client requests and distributes them across multiple servers so that no single server becomes overloaded.

---

## 1. Big Picture

### Simple Explanation

Explain Load Balancer as if teaching a complete beginner.

Use a real-life analogy such as:

> A restaurant has many counters. Instead of every customer going to the same counter, a person directs customers to different counters.

Then map it to software:

```text
Users
  |
  v
Load Balancer
  |
  +--------> Server 1
  |
  +--------> Server 2
  |
  +--------> Server 3**

