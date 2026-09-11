# Routing

## Definition

**Routing** is the process of finding the best path for data packets to travel from a **source device** to a **destination device** across one or more networks.

This decision is made by a **router** using a **routing table**.

> **Simple Definition**
>
> "Routing is the process of choosing the best path for data to reach its destination."

Example:

```
Laptop

↓

Router

↓

Internet

↓

Server
```

---

# Why it Matters

The Internet is made up of millions of interconnected networks.

When you open:

```
https://google.com
```

Your data travels through multiple routers before reaching Google's server.

Without routing:

- Devices on different networks could not communicate.
- The Internet would not work.

---

# How it Works

Every router maintains a **routing table**.

When a packet arrives:

1. Read the destination IP.
2. Check the routing table.
3. Find the best matching route.
4. Forward the packet to the next router.

Example:

```
Source

↓

Router A

↓

Router B

↓

Router C

↓

Destination
```

Each router only decides the **next hop**, not the complete journey.

---

# Routing Table

Example:

| Destination Network | Next Hop |
|---------------------|----------|
| 10.0.0.0/16 | Router A |
| 172.16.0.0/16 | Router B |
| 0.0.0.0/0 | Internet Gateway |

When a packet's destination matches one of these networks, the router forwards it accordingly.

---

# Default Route

Sometimes the router doesn't know the exact destination.

It uses the **default route**.

```
0.0.0.0/0
```

Meaning:

```
Unknown Network

↓

Send to Internet Gateway
```

This is why `0.0.0.0/0` is called the **default route**.

---

# Packet Routing Example

Suppose your laptop wants to access:

```
https://example.com
```

Flow:

```
Laptop

↓

Home Router

↓

ISP Router

↓

Internet Routers

↓

Example Server
```

Each router checks the destination IP and forwards the packet.

---

# Longest Prefix Match

If multiple routes match, the router chooses the **most specific** one.

Example:

Routing Table:

```
10.0.0.0/8

10.1.0.0/16

10.1.2.0/24
```

Packet:

```
10.1.2.25
```

The router selects:

```
10.1.2.0/24
```

because it is the **longest (most specific) match**.

---

# Types of Routing

## 1. Static Routing

Routes are manually configured.

Example:

```
Destination

10.0.0.0/16

↓

Router A
```

Advantages:

- Simple
- Predictable

Disadvantages:

- Doesn't adjust automatically.

---

## 2. Dynamic Routing

Routers automatically exchange routing information.

Common protocols:

- RIP
- OSPF
- EIGRP
- BGP

Advantages:

- Automatically adapts to network changes.

Disadvantages:

- More complex.

---

# Routing vs Switching

| Routing | Switching |
|----------|-----------|
| Connects different networks | Connects devices in the same network |
| Uses IP addresses | Uses MAC addresses |
| Layer 3 (Network Layer) | Layer 2 (Data Link Layer) |
| Performed by routers | Performed by switches |

Easy memory:

```
Switch

↓

Inside one network


Router

↓

Between different networks
```

---

# Routing in Cloud

Example AWS VPC:

```
Internet

↓

Internet Gateway

↓

Public Route Table

↓

Public Subnet

↓

Private Route Table

↓

Private Subnet
```

Route tables decide where packets go.

---

# Routing in Kubernetes

Example:

```
Client

↓

Ingress

↓

Service

↓

Pod
```

Kubernetes uses routing to send traffic to the correct Pod.

---

# Routing in Enterprise Architecture

```
Client

↓

Internet

↓

Firewall

↓

Load Balancer

↓

Reverse Proxy

↓

Spring Boot Application

↓

Database
```

Every network device routes traffic to the next destination.

---

# Common Pitfalls

## ❌ Router knows the complete path.

Wrong.

A router only decides the **next hop**.

---

## ❌ Routing and switching are the same.

Wrong.

Switching works inside a network.

Routing connects different networks.

---

## ❌ Every route is manually configured.

Wrong.

Large networks use **dynamic routing protocols**.

---

## ❌ `0.0.0.0/0` blocks all traffic.

Wrong.

It represents the **default route**, not a block rule.

---

## ❌ Routing provides security.

Wrong.

Routing forwards packets.

Security is provided by:

- Firewalls
- VPNs
- Access control
- Authentication

---

# Spring Boot / Java Notes

Spring Boot applications don't perform IP routing directly.

Instead, they rely on the operating system and network infrastructure.

Example:

```
Browser

↓

Router

↓

Load Balancer

↓

Reverse Proxy

↓

Spring Boot API
```

---

# Cloud Deployment Example

```
Internet

↓

Application Load Balancer

↓

Private Subnet

↓

Spring Boot

↓

Database
```

Route tables determine how packets reach each component.

---

# Docker Example

```
Container A

↓

Docker Network

↓

Container B
```

Docker automatically creates routing rules between containers.

---

# Common Production Issues

## Cannot Reach Server

Possible causes:

- Missing route.
- Wrong gateway.
- Firewall.
- Network outage.

---

## Timeout

Possible causes:

- Incorrect route.
- Server down.
- Network congestion.

---

## No Internet Access

Check:

- Default gateway.
- Default route (`0.0.0.0/0`).
- Internet Gateway (cloud).
- NAT Gateway (private subnet).

---

# Quick Revision

- Routing sends packets between different networks.
- Routers use routing tables.
- Routers choose the **next hop**.
- `0.0.0.0/0` is the default route.
- Longest Prefix Match selects the most specific route.
- Static routing is manual.
- Dynamic routing updates automatically.
- Routing works at **OSI Layer 3**.

---

# Interview Questions

### 1. What is routing?

Routing is the process of selecting the best path for network packets to reach their destination.

---

### 2. What device performs routing?

A router.

---

### 3. What is a routing table?

A table containing destination networks and the next hop used to forward packets.

---

### 4. What is the default route?

```
0.0.0.0/0
```

It is used when no more specific route matches.

---

### 5. What is Longest Prefix Match?

The router selects the most specific matching route (largest prefix length).

---

### 6. Difference between routing and switching?

Routing connects different networks using IP addresses.

Switching connects devices within the same network using MAC addresses.

---

### 7. What is static routing?

Routes are manually configured by an administrator.

---

### 8. What is dynamic routing?

Routers automatically exchange and update routing information using routing protocols.

---

### 9. Does Spring Boot perform routing?

No. Routing is handled by the operating system and network infrastructure.

---

### 10. Why should a Spring Boot developer understand routing?

Because cloud deployments, Kubernetes, Docker, load balancers, and production troubleshooting all depend on correct network routing.

---

# References

- [[IP]]
- [[CIDR]]
- [[NAT]]
- [[Ports]]
- [[TCP-IP]]
- [[Firewall]]
- [[Switching]]
- [[Reverse Proxy]]
- [[Load Balancer]]
- [[DNS]]