# IP (Internet Protocol)

## Definition

**IP (Internet Protocol)** is a **Network Layer protocol** responsible for **addressing and routing data packets** from one device to another across a network.

IP gives every device a unique address so that data knows:

- Where it is coming from.
- Where it needs to go.

> **Simple Definition**
>
> "IP is the addressing system of the Internet that helps data find the correct destination."

---

## Why it Matters

Every device connected to a network needs an address.

Examples:

- Your laptop
- Mobile phone
- Server
- Database server
- Cloud instance

When you open a website:

```
Your Computer

IP Address

        ↓

Internet

        ↓

Server IP Address
```

The network uses IP addresses to deliver your request to the correct server.

Without IP:

- Devices cannot find each other.
- Routers cannot forward packets.
- Internet communication is impossible.

---

## How it Works

IP works mainly at the **Network Layer (Layer 3)** of the OSI model.

The main responsibility of IP is:

1. Assign addresses.
2. Create packets.
3. Route packets between networks.

---

## Basic IP Communication

Example:

Your laptop:

```
192.168.1.10
```

Server:

```
142.250.182.14
```

When you send a request:

```
Source IP

192.168.1.10

        ↓

Destination IP

142.250.182.14
```

Routers read the destination IP and decide where to send the packet next.

---

## IP Packet Flow

```
Application Data

↓

TCP/UDP adds Port Number

↓

IP adds Source + Destination IP

↓

Router forwards packet

↓

Destination Server
```

---

## Types of IP Addresses

## 1. IPv4

Most commonly used format.

Example:

```
192.168.1.100
```

IPv4 uses:

```
32 bits
```

Total addresses:

```
~4.3 billion
```

---

## 2. IPv6

Created because IPv4 addresses are limited.

Example:

```
2001:0db8:85a3::8a2e:0370:7334
```

IPv6 uses:

```
128 bits
```

Advantages:

- Huge address space.
- Better support for modern networks.
- Improved routing efficiency.

---

## Public IP vs Private IP

## Public IP

Used on the Internet.

Example:

```
142.250.182.14
```

Assigned by:

- ISP
- Cloud provider

Anyone on the Internet can reach it.

---

## Private IP

Used inside local networks.

Examples:

```
192.168.x.x

10.x.x.x

172.16.x.x - 172.31.x.x
```

Used for:

- Home networks
- Office networks
- Internal servers

---

## Static IP vs Dynamic IP

## Static IP

IP address remains fixed.

Used for:

- Servers
- Websites
- Databases

Example:

```
Production Server

10.20.1.50
```

---

## Dynamic IP

IP address changes automatically.

Assigned using:

```
DHCP
```

Common for:

- Home users
- Mobile devices

---

## IP Routing

Routers make decisions based on IP addresses.

Example:

```
Computer

192.168.1.10

        ↓

Router

        ↓

Internet

        ↓

Server

142.250.182.14
```

Router checks:

"Which path should this packet take?"

Then forwards it.

---

## IP Does Not Guarantee Delivery

IP is a **best-effort protocol**.

It does not guarantee:

- Delivery
- Order
- Error recovery

Example:

```
Packet 1 ✅

Packet 2 ❌ Lost

Packet 3 ✅
```

IP does not resend Packet 2.

TCP handles reliability.

---

## IP Address vs MAC Address

| IP Address | MAC Address |
|-|-|
| Layer 3 | Layer 2 |
| Logical address | Physical address |
| Changes possible | Usually fixed |
| Used by routers | Used by switches |
| Example: 192.168.1.10 | Example: A4:B1:C2:D3:E4:F5 |

Easy understanding:

```
IP = House Address

MAC = Person inside the house
```

---

## Common Protocols Related to IP

| Protocol | Purpose |
|----------|---------|
| IPv4 | Addressing |
| IPv6 | Next generation addressing |
| ICMP | Error reporting and diagnostics |
| ARP | Find MAC address from IP |
| DHCP | Assign IP addresses |

---

## Common Pitfalls

### ❌ IP guarantees delivery.

Wrong.

IP only provides addressing and routing.

TCP provides reliable delivery.

---

### ❌ Public IP and Private IP are the same.

Wrong.

Public IP is Internet-facing.

Private IP is internal network addressing.

---

### ❌ Every device has only one IP.

Wrong.

A server can have:

- Multiple IP addresses.
- Multiple network interfaces.

---

### ❌ IP address identifies a person.

Wrong.

An IP identifies a network interface, not a specific person.

---

### ❌ Changing IP means changing the device.

Wrong.

A device can receive different IP addresses over time.

---

## Spring Boot / Java Notes

A Spring Boot application runs on a server with an IP address.

Example:

```
Client

↓

Public IP

↓

Load Balancer

↓

Spring Boot Server

Private IP

↓

Database
```

---

When calling an API:

Example:

```
GET https://api.example.com/users
```

The flow:

```
DNS

↓

Domain converted to IP

↓

TCP Connection

↓

HTTP Request

↓

Spring Boot Application
```

---

Common networking issues in Spring Boot:

### Connection Refused

Possible reasons:

- Server not running.
- Wrong IP.
- Wrong port.
- Firewall blocking.

---

### Connection Timeout

Possible reasons:

- Network problem.
- Security group issue.
- Server unreachable.

---

### Cannot Connect to Database

Check:

- Database IP.
- Port.
- Firewall rules.
- Network configuration.

---

Useful Java methods:

```java
InetAddress.getLocalHost();
```

Gets local machine IP information.

Example:

```java
InetAddress address =
InetAddress.getByName("google.com");
```

Resolves hostname to IP.

---

## Quick Revision

- IP works at Layer 3 (Network Layer).
- IP provides addressing and routing.
- IPv4 uses 32 bits.
- IPv6 uses 128 bits.
- Public IP is Internet-facing.
- Private IP is internal.
- IP does not guarantee delivery.
- TCP provides reliability.
- Routers use IP addresses.
- Switches use MAC addresses.

---

## Interview Questions

### 1. What is IP?

IP is a Network Layer protocol responsible for addressing and routing packets between devices.

---

### 2. Which OSI layer does IP belong to?

Layer 3 (Network Layer).

---

### 3. What is the difference between IPv4 and IPv6?

IPv4 uses 32-bit addresses, while IPv6 uses 128-bit addresses and provides a much larger address space.

---

### 4. Does IP guarantee delivery?

No. IP provides best-effort delivery. TCP provides reliability.

---

### 5. What is a private IP?

An IP address used inside a local network that is not directly reachable from the Internet.

---

### 6. What is a public IP?

An Internet-facing IP address assigned by an ISP or cloud provider.

---

### 7. What device uses IP addresses?

Routers use IP addresses to forward packets between networks.

---

### 8. Difference between IP and MAC address?

IP is a logical network address used for routing. MAC is a physical address used for local network communication.

---

### 9. How does a browser find the server IP?

Through DNS resolution.

---

### 10. Why should a Java developer understand IP?

Because every API call, database connection, cloud deployment, and server communication depends on IP networking.

---

## References

- [[OSI Model]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[UDP]]
- [[DNS]]
- [[Ports]]
- [[Routing]]
- [[CIDR]]
- [[NAT]]
- [[Firewall]]