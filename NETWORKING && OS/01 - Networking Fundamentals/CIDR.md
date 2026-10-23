# CIDR (Classless Inter-Domain Routing)

## Definition

**CIDR (Classless Inter-Domain Routing)** is a method of allocating IP addresses and defining network ranges using **prefix notation** (e.g., `/24`, `/16`, `/32`).

Instead of using old fixed classes (A, B, C), CIDR allows flexible network sizes.

> **Simple Definition**
>
> "CIDR tells us how many bits belong to the network and how many belong to the hosts."

Example:

```
192.168.1.0/24
```

Here:

- `192.168.1` → Network
- `.0 - .255` → Address range

---

# Why it Matters

CIDR is used everywhere in modern networking.

Examples:

- Cloud (AWS, Azure, GCP)
- Kubernetes
- Docker
- Enterprise networks
- Firewalls
- VPCs
- Routing

Without CIDR:

- IP addresses would be wasted.
- Network management would be difficult.

---

# How it Works

CIDR uses the format:

```
IP Address / Prefix Length
```

Example:

```
192.168.1.0/24
```

Meaning:

- First **24 bits** → Network
- Remaining **8 bits** → Host

---

# Understanding Prefix Length

Example:

```
192.168.1.0/24
```

```
Network Bits      Host Bits

24                8
```

Diagram:

```
11111111.11111111.11111111.00000000
```

The `1`s represent the **network part**.

The `0`s represent the **host part**.

---

# Common CIDR Blocks

| CIDR | Subnet Mask     |  Total IPs | Usable Hosts |
| ---- | --------------- | ---------: | -----------: |
| /32  | 255.255.255.255 |          1 |            1 |
| /30  | 255.255.255.252 |          4 |            2 |
| /29  | 255.255.255.248 |          8 |            6 |
| /28  | 255.255.255.240 |         16 |           14 |
| /27  | 255.255.255.224 |         32 |           30 |
| /26  | 255.255.255.192 |         64 |           62 |
| /25  | 255.255.255.128 |        128 |          126 |
| /24  | 255.255.255.0   |        256 |          254 |
| /16  | 255.255.0.0     |     65,536 |       65,534 |
| /8   | 255.0.0.0       | 16,777,216 |   16,777,214 |
|      |                 |            |              |

> **Interview Tip:**  
> A `/24` network has **256 total IP addresses**, but only **254 are usable** because one is the **network address** and one is the **broadcast address**.

---

# CIDR Example

Example:

```
192.168.10.0/24
```

Network Address:

```
192.168.10.0
```

Usable Hosts:

```
192.168.10.1

↓

192.168.10.254
```

Broadcast Address:

```
192.168.10.255
```

---

# Another Example

```
10.0.0.0/16
```

Network:

```
10.0
```

Host range:

```
10.0.0.1

↓

10.0.255.254
```

---

# CIDR in Cloud

AWS VPC example:

```
VPC

10.0.0.0/16
```

Subnets:

```
Public

10.0.1.0/24


Private

10.0.2.0/24


Database

10.0.3.0/24
```

Each subnet belongs to the same VPC.

---

# CIDR and Routing

Routers use CIDR to decide where packets should go.

Example:

```
Destination IP

↓

Routing Table

↓

Matching CIDR

↓

Forward Packet
```

---

# CIDR and Firewalls

Firewall rules often use CIDR.

Example:

Allow:

```
10.0.0.0/16
```

Meaning:

Allow every IP inside that network.

Block:

```
0.0.0.0/0
```

Meaning:

Matches every IPv4 address (the entire Internet).

---

# CIDR vs Subnet Mask

These two represent the same thing.

| CIDR | Subnet Mask |
|------|-------------|
| /24 | 255.255.255.0 |
| /16 | 255.255.0.0 |
| /8 | 255.0.0.0 |

Example:

```
192.168.1.0/24

=

192.168.1.0
255.255.255.0
```

---

# Common Private Networks

| Network | CIDR |
|---------|------|
| 10.0.0.0 | /8 |
| 172.16.0.0 | /12 |
| 192.168.0.0 | /16 |

These are **private IP ranges**.

---

# Common Pitfalls

## ❌ CIDR and subnet mask are different.

Wrong.

They represent the same network information in different formats.

---

## ❌ `/24` means 24 IP addresses.

Wrong.

`/24` means **24 network bits**, not 24 IPs.

A `/24` network has **256 total IP addresses**.

---

## ❌ Every IP in a subnet can be assigned.

Wrong.

Normally:

- First IP → Network Address
- Last IP → Broadcast Address

They are not assigned to hosts (in traditional IPv4 networks).

---

## ❌ Bigger CIDR means bigger network.

Wrong.

A larger prefix means a **smaller network**.

Example:

```
/16

↓

Large network


/24

↓

Smaller network


/32

↓

Single IP
```

---

## ❌ CIDR is only used for routing.

Wrong.

CIDR is also used in:

- Firewalls
- Cloud networking
- Kubernetes
- Docker
- VPNs
- Access control

---

# Spring Boot / Java Notes

Spring Boot applications usually don't configure CIDR directly, but deployment infrastructure does.

Example:

```
Internet

↓

Load Balancer

↓

Spring Boot Server

10.0.2.10
```

Firewall rule:

```
Allow

10.0.0.0/16
```

Only servers inside the VPC can connect.

---

# Cloud Deployment Example

AWS:

```
VPC

10.0.0.0/16


Public Subnet

10.0.1.0/24


Private Subnet

10.0.2.0/24


Spring Boot App

10.0.2.15
```

Database:

```
10.0.3.20
```

Everything communicates using private IPs.

---

# Kubernetes Example

Pod Network:

```
10.244.0.0/16
```

Service Network:

```
10.96.0.0/12
```

CIDR defines the available IP ranges.

---

# Quick Revision

- CIDR stands for **Classless Inter-Domain Routing**.
- CIDR format: `IP/Prefix`.
- `/24` = 256 total IPs (254 usable).
- `/16` is larger than `/24`.
- CIDR replaces old Class A, B, and C addressing.
- Used in routing, firewalls, cloud networking, and Kubernetes.
- `0.0.0.0/0` represents all IPv4 addresses.
- CIDR and subnet masks represent the same concept.

---

# Interview Questions

### 1. What is CIDR?

CIDR is a method of representing IP networks using prefix notation like `/24`.

---

### 2. Why was CIDR introduced?

To efficiently allocate IP addresses and replace the old fixed Class A, B, and C system.

---

### 3. What does `/24` mean?

The first 24 bits represent the network, leaving 8 bits for hosts.

---

### 4. How many usable hosts are available in a `/24` network?

254 usable hosts.

---

### 5. What is the subnet mask for `/24`?

```
255.255.255.0
```

---

### 6. Which is larger: `/16` or `/24`?

`/16` is a larger network because it has more host addresses.

---

### 7. What does `0.0.0.0/0` mean?

It matches every IPv4 address and is commonly used as the default route or to allow traffic from anywhere.

---

### 8. Where is CIDR commonly used?

- Cloud networking
- Firewalls
- Routing
- Kubernetes
- Docker
- VPNs

---

### 9. Why is CIDR important for cloud platforms?

It defines VPCs, subnets, routing, and security rules.

---

### 10. Why should a Spring Boot developer understand CIDR?

Because deploying applications in AWS, Azure, Kubernetes, or Docker requires understanding network ranges, subnets, and firewall rules.

---

# References

- [[IP]]
- [[NAT]]
- [[Routing]]
- [[Firewall]]
- [[TCP-IP]]
- [[Ports]]
- [[VPN]]
- [[Kubernetes Networking]]
- [[AWS VPC]]