# VLAN (Virtual Local Area Network)

## Definition

A **VLAN (Virtual Local Area Network)** is a technology that divides one physical network into multiple **logical (virtual) networks**.

Devices in different VLANs behave as if they are on separate physical networks, even when connected to the same switch.

> **Simple Definition**
>
> "A VLAN creates separate virtual networks inside the same physical switch."

Example:

```
           Switch

      /      |      \

 PC1       PC2      PC3

VLAN 10   VLAN 20  VLAN 10
```

PC1 and PC3 can communicate directly.

PC2 is isolated because it belongs to a different VLAN.

---

# Why it Matters

Without VLAN:

```
All devices

↓

Same Network
```

Problems:

- More broadcast traffic.
- Lower security.
- Difficult network management.

With VLAN:

```
HR Department

↓

VLAN 10


Finance

↓

VLAN 20


IT

↓

VLAN 30
```

Each department has its own isolated network.

---

# Real World Example

A company has three departments:

- HR
- Finance
- Development

Instead of installing three separate switches, one switch can create three VLANs.

```
           One Switch

        -----------------

       VLAN 10  → HR

       VLAN 20  → Finance

       VLAN 30  → Development
```

---

# How it Works

Every switch port is assigned to a VLAN.

Example:

| Port | VLAN |
|------|------|
| Port 1 | VLAN 10 |
| Port 2 | VLAN 10 |
| Port 3 | VLAN 20 |
| Port 4 | VLAN 30 |

Devices connected to the same VLAN can communicate directly.

Devices in different VLANs cannot communicate unless a **router** or **Layer 3 switch** is used.

---

# VLAN Communication

Example:

```
PC1

VLAN 10

↓

Switch

↓

PC2

VLAN 10
```

Communication succeeds.

---

Different VLAN:

```
PC1

VLAN 10

↓

Switch

↓

PC3

VLAN 20
```

Communication fails unless inter-VLAN routing is configured.

---

# Broadcast Domains

Each VLAN creates its own **broadcast domain**.

Without VLAN:

```
Broadcast

↓

Every Device
```

With VLAN:

```
Broadcast

↓

Only Devices

Inside VLAN 10
```

Benefits:

- Less unnecessary traffic.
- Better performance.

---

# VLAN IDs

A VLAN is identified by a **VLAN ID**.

Common range:

```
1 - 4094
```

Examples:

```
VLAN 10

HR


VLAN 20

Finance


VLAN 30

Development
```

---

# Access Port

An **Access Port** belongs to only one VLAN.

Example:

```
Laptop

↓

Switch Port

↓

VLAN 10
```

Used for:

- PCs
- Printers
- Servers

---

# Trunk Port

A **Trunk Port** carries traffic for multiple VLANs between network devices.

Example:

```
Switch A

↓

Trunk Port

↓

Switch B
```

Traffic for:

- VLAN 10
- VLAN 20
- VLAN 30

can all travel through the same cable.

---

# VLAN vs Subnet

| VLAN | Subnet |
|------|--------|
| Layer 2 | Layer 3 |
| Creates logical LAN | Defines IP network |
| Uses VLAN IDs | Uses IP addresses |
| Switch feature | Router feature |

Often:

```
One VLAN

↓

One Subnet
```

Example:

```
VLAN 10

↓

192.168.10.0/24


VLAN 20

↓

192.168.20.0/24
```

---

# Inter-VLAN Routing

Devices in different VLANs need a router or Layer 3 switch.

Example:

```
VLAN 10

↓

Router

↓

VLAN 20
```

Without routing:

```
No Communication
```

---

# Enterprise Example

```
              Router

                 |

            Core Switch

        /      |      \

   VLAN10  VLAN20  VLAN30

      |        |        |

     HR     Finance   Dev
```

Each department is isolated but can communicate through the router if allowed.

---

# VLAN in Data Centers

Servers are often separated using VLANs.

Example:

```
VLAN 100

Application Servers


VLAN 200

Database Servers


VLAN 300

Management Network
```

Benefits:

- Better security.
- Easier management.
- Reduced broadcast traffic.

---

# Common Pitfalls

## ❌ VLAN is a physical network.

Wrong.

A VLAN is a **logical** network.

---

## ❌ Devices in different VLANs communicate automatically.

Wrong.

They need:

- Router
- Layer 3 Switch

---

## ❌ VLAN replaces a firewall.

Wrong.

VLAN provides network segmentation.

Firewall provides security policies.

---

## ❌ VLAN improves Internet speed.

Wrong.

VLAN reduces unnecessary local traffic but does not increase Internet bandwidth.

---

## ❌ VLAN and Subnet are the same.

Wrong.

VLAN is a Layer 2 concept.

Subnet is a Layer 3 concept.

---

# Spring Boot / Java Notes

Enterprise applications are often deployed in different VLANs.

Example:

```
VLAN 100

Spring Boot Servers


VLAN 200

Database Servers


VLAN 300

Monitoring Servers
```

Communication between VLANs is controlled using routers and firewalls.

---

# Cloud Equivalent

Cloud providers don't usually use traditional VLANs.

Instead, they provide similar isolation using:

- VPCs
- Subnets
- Security Groups
- Network ACLs

The concept is similar: isolate network traffic.

---

# Common Production Issues

## Application Cannot Reach Database

Possible causes:

- Different VLANs.
- Missing inter-VLAN routing.
- Firewall rules.
- Incorrect VLAN configuration.

---

## Wrong VLAN Assignment

A server connected to the wrong VLAN cannot communicate with expected services.

---

## Broadcast Problems

If many devices are placed in one VLAN:

- Broadcast traffic increases.
- Network performance decreases.

---

# Quick Revision

- VLAN stands for **Virtual Local Area Network**.
- VLAN creates logical networks inside one physical switch.
- VLAN works at **OSI Layer 2**.
- Devices in different VLANs cannot communicate directly.
- Inter-VLAN communication requires a router or Layer 3 switch.
- Access ports belong to one VLAN.
- Trunk ports carry multiple VLANs.
- VLANs improve security and reduce broadcast traffic.

---

# Interview Questions

### 1. What is a VLAN?

A VLAN is a logical network that separates devices into different broadcast domains on the same physical switch.

---

### 2. Why do we use VLANs?

To improve security, reduce broadcast traffic, and logically separate departments or applications.

---

### 3. Which OSI layer does VLAN operate on?

Layer 2 (Data Link Layer).

---

### 4. Can devices in different VLANs communicate?

Not directly.

They require inter-VLAN routing using a router or Layer 3 switch.

---

### 5. What is an Access Port?

A switch port assigned to a single VLAN.

---

### 6. What is a Trunk Port?

A switch port that carries traffic for multiple VLANs.

---

### 7. Difference between VLAN and Subnet?

A VLAN is a Layer 2 logical network.

A subnet is a Layer 3 IP network.

---

### 8. How does VLAN improve security?

It isolates devices into separate networks, reducing unnecessary communication and limiting access.

---

### 9. Are VLANs used in cloud environments?

Traditional VLANs are uncommon, but similar isolation is achieved using VPCs, subnets, security groups, and network ACLs.

---

### 10. Why should a Spring Boot developer understand VLANs?

Because enterprise applications, databases, and infrastructure are often deployed on separate network segments, and VLAN knowledge helps troubleshoot connectivity and security issues.

---

# References

- [[Switching]]
- [[Routing]]
- [[IP]]
- [[CIDR]]
- [[Firewall]]
- [[NAT]]
- [[Ports]]
- [[TCP-IP]]
- [[OSI Model]]
- [[VPN]]