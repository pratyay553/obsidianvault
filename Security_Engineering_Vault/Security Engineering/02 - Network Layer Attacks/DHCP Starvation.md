# DHCP Starvation

## Definition

**DHCP Starvation** is a network attack where an attacker sends a large number of **fake DHCP requests** using different spoofed MAC addresses to consume all available IP addresses in a DHCP server's address pool.

Once the IP pool is exhausted, legitimate users cannot obtain an IP address and cannot join the network.

> **Simple Definition**
>
> "DHCP Starvation exhausts all available IP addresses so real users cannot connect to the network."

---

# How it Works

### Normal DHCP Process

A new device joins the network.

```
Client

↓

DHCP Discover

↓

DHCP Server

↓

DHCP Offer

↓

DHCP Request

↓

DHCP ACK

↓

IP Assigned
```

---

### During DHCP Starvation

The attacker continuously sends DHCP requests.

```
Fake MAC 1

↓

DHCP Request

↓

IP Assigned


Fake MAC 2

↓

DHCP Request

↓

IP Assigned


Fake MAC 3

↓

DHCP Request

↓

IP Assigned

...
```

Eventually:

```
DHCP IP Pool

↓

EMPTY
```

Now legitimate users cannot receive an IP address.

---

# Target Layer

**OSI Layer:** **Layer 2 (Data Link Layer)**

Reason:

- The attack relies on spoofed **MAC addresses** to obtain multiple IP leases.
- Although DHCP uses UDP at the Transport Layer, the attack primarily abuses Layer 2 device identity.

---

# Prerequisites

For DHCP Starvation:

- Attacker must be connected to the **same LAN**.
- DHCP must be enabled.
- DHCP server has a limited IP pool.
- Switch lacks DHCP security features.

Common targets:

- Office networks
- Schools
- Public Wi-Fi
- Enterprise LANs

---

# Real-world Example

Suppose the DHCP pool is:

```
192.168.1.100

↓

192.168.1.200
```

The attacker sends hundreds of fake requests.

```
MAC 1 → 192.168.1.100

MAC 2 → 192.168.1.101

MAC 3 → 192.168.1.102

...

MAC 101 → 192.168.1.200
```

All IP addresses are leased.

A real user connects.

```
Laptop

↓

DHCP Discover

↓

No Available IP

↓

Connection Failed
```

---

# Impact

Possible consequences:

- Legitimate users cannot join the network.
- Network outage (Denial of Service).
- Business disruption.
- Loss of productivity.
- Can be combined with a **Rogue DHCP Server** attack to redirect users to a malicious gateway or DNS server.

---

# Detection

Signs of DHCP Starvation:

- Users fail to obtain IP addresses.
- DHCP server reports no free leases.
- Large number of DHCP requests from one switch port.
- Rapid increase in DHCP lease allocations.
- Network monitoring alerts.

Useful tools:

- DHCP server logs.
- Switch logs.
- Wireshark.
- IDS/IPS.
- SNMP monitoring.

---

# Prevention

### 1. DHCP Snooping ✅ (Best Protection)

Switches inspect DHCP messages and allow DHCP responses only from trusted ports.

This also limits fake DHCP requests.

---

### 2. Port Security

Limit the number of MAC addresses allowed on a switch port.

Example:

```
Port 5

↓

Maximum 2 MAC Addresses
```

---

### 3. Rate Limiting

Limit the number of DHCP requests allowed per second from a port.

---

### 4. 802.1X Network Authentication

Require devices to authenticate before receiving network access.

---

### 5. Monitor DHCP Leases

Alert administrators when lease usage suddenly spikes.

---

### 6. Disable Unused Ports

Unused switch ports should remain disabled.

---

# Spring Boot Protection

Spring Boot cannot prevent DHCP Starvation because it occurs before an application even receives network connectivity.

However, production applications should:

- Use static IPs for servers when appropriate.
- Monitor infrastructure health.
- Implement health checks.
- Deploy in redundant environments.

Example:

```
Internet

↓

Switch

↓

DHCP

↓

Spring Boot Server
```

If the server uses a static IP, DHCP Starvation does not affect it directly.

---

# Infrastructure Protection

Enterprise networks should implement:

- DHCP Snooping.
- Port Security.
- 802.1X authentication.
- VLAN segmentation.
- IDS/IPS.
- Network monitoring.
- Redundant DHCP servers.

Example:

```
Internet

↓

Firewall

↓

Managed Switch

↓

DHCP Snooping

↓

Application Servers
```

---

# Best Practices

- Enable DHCP Snooping on managed switches.
- Limit MAC addresses per access port.
- Enable Port Security.
- Use 802.1X authentication.
- Monitor DHCP lease utilization.
- Disable unused switch ports.
- Use static IP addresses for critical servers.
- Regularly review DHCP logs.

---

# Interview Questions

### 1. What is DHCP Starvation?

A Denial-of-Service attack that exhausts a DHCP server's available IP address pool using fake DHCP requests.

---

### 2. Which OSI layer does DHCP Starvation target?

Primarily Layer 2 because it relies on spoofed MAC addresses to obtain multiple DHCP leases.

---

### 3. What is the goal of DHCP Starvation?

To prevent legitimate users from obtaining IP addresses.

---

### 4. What happens when the DHCP pool is exhausted?

New devices cannot receive an IP address and cannot communicate on the network.

---

### 5. What is DHCP Snooping?

A switch security feature that validates DHCP traffic and allows DHCP responses only from trusted ports.

---

### 6. Can DHCP Starvation lead to other attacks?

Yes.

It is often followed by a **Rogue DHCP Server** attack to provide victims with malicious network settings.

---

### 7. Can Spring Boot prevent DHCP Starvation?

No.

The attack occurs before the application layer and affects network infrastructure.

---

### 8. How can enterprises prevent DHCP Starvation?

By using:

- DHCP Snooping
- Port Security
- Rate Limiting
- 802.1X Authentication
- VLAN Segmentation
- Network Monitoring

---

### 9. Why are static IP addresses used for servers?

Critical servers should remain reachable even if the DHCP service has issues.

---

### 10. How is DHCP Starvation different from MAC Flooding?

| DHCP Starvation | MAC Flooding |
|-----------------|--------------|
| Exhausts DHCP IP leases | Fills the switch's MAC table |
| Targets the DHCP service | Targets the switch |
| Prevents new devices from joining | Causes the switch to flood traffic |
| Denial-of-Service attack | Packet sniffing–enabling attack |

---

# References

- [[DHCP]]
- [[MAC Address]]
- [[Switching]]
- [[VLAN]]
- [[ARP Spoofing]]
- [[MAC Flooding]]
- [[OSI Model]]
- [[TCP-IP]]
- [[Firewall]]
- [[Ports]]
- [[UDP]]