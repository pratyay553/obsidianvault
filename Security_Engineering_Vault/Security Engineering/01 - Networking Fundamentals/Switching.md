# Switching

## Definition

**Switching** is the process of forwarding data **between devices within the same Local Area Network (LAN)** using **MAC addresses**.

A **network switch** learns the MAC address of each connected device and sends data only to the correct destination.

> **Simple Definition**
>
> "Switching connects devices inside the same network and delivers data to the correct device."

Example:

```
          Switch

      /     |     \

 Laptop   Server   Printer
```

All devices communicate through the switch.

---

# Why it Matters

Imagine an office with:

- 100 computers
- 20 printers
- 10 servers

Without a switch:

- Every device would receive every packet.
- Network performance would be poor.

A switch sends data **only to the intended device**, making communication:

- Faster
- More efficient
- More secure

---

# How it Works

Every network device has a unique **MAC Address**.

Example:

```
Laptop

MAC:
AA:BB:CC:11:22:33
```

The switch learns which MAC address is connected to which port.

When a packet arrives:

1. Read destination MAC address.
2. Check the MAC Address Table.
3. Forward the packet to the correct port.

Example:

```
Laptop

↓

Switch

↓

Server
```

---

# MAC Address Table

The switch maintains a table.

Example:

| MAC Address | Switch Port |
|-------------|-------------|
| AA:BB:CC:11:22:33 | Port 1 |
| DD:EE:FF:44:55:66 | Port 2 |
| 11:22:33:44:55:66 | Port 3 |

When a packet is received:

```
Destination MAC

↓

MAC Table

↓

Correct Port
```

---

# Learning Process

Initially, the switch knows nothing.

### Step 1

Laptop sends data.

```
Laptop

↓

Switch
```

The switch learns:

```
Laptop MAC

↓

Port 1
```

---

### Step 2

Server replies.

```
Server

↓

Switch
```

Now the switch learns:

```
Server MAC

↓

Port 2
```

The MAC table grows automatically.

---

# Unknown Destination

If the destination MAC is unknown:

The switch performs **Flooding**.

```
Incoming Packet

↓

Send to all ports

(except incoming port)
```

Once the destination replies, the switch learns its MAC address.

---

# Broadcast Traffic

Some packets are meant for **all devices**.

Example:

```
ARP Request
```

These packets are broadcast.

The switch forwards them to every device in the LAN.

---

# Switching vs Routing

| Switching | Routing |
|-----------|---------|
| Connects devices in the same network | Connects different networks |
| Uses MAC addresses | Uses IP addresses |
| Layer 2 (Data Link Layer) | Layer 3 (Network Layer) |
| Performed by a switch | Performed by a router |

Easy memory:

```
Switch

↓

Same Network


Router

↓

Different Networks
```

---

# Switch vs Hub

| Switch | Hub |
|---------|-----|
| Intelligent | Dumb |
| Uses MAC table | No MAC table |
| Sends data to one device | Sends data to all devices |
| Better performance | More collisions |

Today, switches are used instead of hubs.

---

# Enterprise Network Example

```
              Router

                 |

              Network Switch

      /        |         \

 Laptop     Server     Printer
```

The switch handles communication within the office network.

The router connects the office to other networks or the Internet.

---

# Switching in Data Centers

Modern data centers use high-speed switches to connect:

- Application servers
- Database servers
- Storage systems
- Load balancers

Example:

```
Application Server

        |

Top-of-Rack Switch

        |

Database Server
```

---

# Common Pitfalls

## ❌ Switch and Router are the same.

Wrong.

A switch connects devices in the same network.

A router connects different networks.

---

## ❌ Switch uses IP addresses.

Wrong.

A Layer 2 switch uses **MAC addresses**.

---

## ❌ Switch broadcasts every packet.

Wrong.

Only unknown and broadcast traffic is flooded.

Known destinations are sent only to the correct port.

---

## ❌ Every packet reaches every device.

Wrong.

A switch forwards packets only to the destination port.

---

## ❌ Switching provides Internet access.

Wrong.

Internet access requires a **router**.

---

# Spring Boot / Java Notes

Spring Boot applications usually communicate through switches inside a data center.

Example:

```
Spring Boot App

↓

Network Switch

↓

Database Server
```

The application is not aware of the switching process.

---

# Production Architecture

```
Internet

↓

Router

↓

Firewall

↓

Switch

↓

Load Balancer

↓

Spring Boot Servers
```

The switch efficiently connects internal servers.

---

# Docker/Kubernetes

Even in container environments, the underlying infrastructure uses switching to connect:

- Worker nodes
- Storage
- Network devices

---

# Common Production Issues

## Server Cannot Reach Database

Possible causes:

- Switch port down.
- Wrong VLAN.
- Cable issue.
- Network configuration error.

---

## Slow Network

Possible causes:

- Network congestion.
- Faulty switch.
- Duplex mismatch.
- Hardware failure.

---

## Intermittent Connectivity

Check:

- Switch health.
- Port status.
- MAC address table.
- Physical connections.

---

# Quick Revision

- Switching connects devices within the same network.
- Switches use **MAC addresses**.
- Switching works at **OSI Layer 2**.
- Switches maintain a **MAC Address Table**.
- Unknown destinations are flooded.
- Broadcast packets reach all devices.
- Switches are faster and smarter than hubs.
- Routers connect different networks.

---

# Interview Questions

### 1. What is switching?

Switching is the process of forwarding data between devices within the same LAN using MAC addresses.

---

### 2. Which device performs switching?

A network switch.

---

### 3. Which OSI layer does switching work on?

Layer 2 (Data Link Layer).

---

### 4. What address does a switch use?

MAC address.

---

### 5. What is a MAC Address Table?

A table that maps MAC addresses to switch ports.

---

### 6. What happens if the destination MAC address is unknown?

The switch floods the packet to all ports except the incoming port.

---

### 7. Difference between a switch and a router?

A switch connects devices in the same network using MAC addresses.

A router connects different networks using IP addresses.

---

### 8. Why is a switch better than a hub?

Because it sends data only to the intended device instead of broadcasting every packet.

---

### 9. Does Spring Boot perform switching?

No. Switching is handled by the network infrastructure.

---

### 10. Why should a Spring Boot developer understand switching?

Because enterprise applications communicate over switched networks, and understanding switching helps troubleshoot connectivity and deployment issues.

---

# References

- [[OSI Model]]
- [[TCP-IP]]
- [[IP]]
- [[MAC Address]]
- [[Routing]]
- [[Ports]]
- [[Firewall]]
- [[VLAN]]
- [[NAT]]
- [[DNS]]