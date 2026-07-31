# NAT (Network Address Translation)

## Definition

**NAT (Network Address Translation)** is a networking technique that converts **private IP addresses into public IP addresses** and vice versa while communicating over the Internet.

It allows multiple devices inside a private network to share a single public IP address.

> **Simple Definition**
>
> "NAT is a translator between private network IP addresses and public Internet IP addresses."

Example:

Private Network:

```
Laptop

192.168.1.10


Mobile

192.168.1.11


Server

192.168.1.12
```

All access Internet through:

```
Public IP

45.20.10.5
```

---

# Why it Matters

Without NAT:

Every device would need a unique public IP address.

Problem:

```
Millions of devices

+

Limited IPv4 addresses
```

NAT helps by:

- Saving IPv4 addresses.
- Hiding internal network structure.
- Providing basic network security.
- Allowing private networks.

---

# Real World Example

Home network:

```
                 Internet

                    |

              Public IP

             103.50.x.x

                    |

                 Router

        ---------------------

        |         |         |

     Laptop    Mobile    TV

192.168.1.10  .11       .12
```

The router performs NAT.

---

# How it Works

NAT changes IP information in network packets.

Example:

## Before NAT

Private device sends request:

```
Source IP:

192.168.1.10


Destination:

google.com
```

---

## NAT Router Changes It

Router replaces:

```
192.168.1.10
```

with:

```
45.20.10.5
```

---

## After NAT

Internet sees:

```
Source IP:

45.20.10.5


Destination:

google.com
```

---

# NAT Table

The router maintains a mapping table.

Example:

| Private IP | Private Port | Public IP | Public Port |
|-|-|-|-|
|192.168.1.10|5000|45.20.10.5|30001|
|192.168.1.11|5001|45.20.10.5|30002|

When response comes back:

```
Public Port

↓

NAT Table

↓

Correct Private Device
```

---

# Types of NAT

## 1. Static NAT

One private IP maps to one public IP.

Example:

```
Private Server

192.168.1.10

        ↓

Public IP

45.20.10.10
```

Used for:

- Public servers.
- Dedicated systems.

---

## 2. Dynamic NAT

Private IP gets a public IP from a pool.

Example:

```
Private IP

↓

Available Public IP
```

---

## 3. PAT (Port Address Translation)

Also called:

```
NAT Overload
```

Most common type.

Many private devices share one public IP using different ports.

Example:

```
Device 1:

192.168.1.10:5000


Device 2:

192.168.1.11:5001


↓

Same Public IP:

45.20.10.5
```

Different ports identify devices.

---

# NAT Flow

Example:

User opens website:

```
https://example.com
```

Flow:

```
Laptop

Private IP

192.168.1.10

        |

        |

Router NAT

        |

        |

Public IP

45.20.10.5

        |

        |

Internet

        |

        |

Website Server
```

---

# NAT and Private IP Addresses

Common private IP ranges:

## Class A

```
10.0.0.0/8
```

---

## Class B

```
172.16.0.0/12
```

---

## Class C

```
192.168.0.0/16
```

These addresses are not directly reachable from the Internet.

---

# NAT vs Firewall

| NAT | Firewall |
|-|-|
| Translates IP addresses | Allows or blocks traffic |
| Network address management | Security control |
| Hides internal IPs | Enforces rules |
| Router functionality | Security system |

They are often used together.

---

# NAT vs Proxy

| NAT | Proxy |
|-|-|
| Works at network level | Works at application level |
| Changes IP addresses | Handles requests |
| Transparent to applications | Application aware |
| Router feature | Server feature |

---

# NAT and Port Forwarding

Normally:

```
Internet

↓

NAT Router

↓

Private Network
```

External users cannot directly access private servers.

Port forwarding creates a rule:

Example:

```
Public IP:443

↓

Internal Server:8080
```

Used for:

- Hosting applications.
- Remote access.
- Game servers.

---

# Common Pitfalls

## ❌ NAT is a security solution.

Wrong.

NAT hides internal IPs but does not replace:

- Firewall.
- Authentication.
- Encryption.

---

## ❌ NAT encrypts data.

Wrong.

NAT only changes addresses.

Encryption is handled by:

- TLS.
- VPN.
- IPsec.

---

## ❌ Private IP can be accessed directly from Internet.

Wrong.

Private IP addresses require:

- NAT.
- Port forwarding.
- VPN.

---

## ❌ NAT is only used at home.

Wrong.

NAT is heavily used in:

- Cloud environments.
- Enterprise networks.
- Data centers.

---

## ❌ NAT improves application performance.

Wrong.

NAT mainly provides address translation.

It may sometimes add slight latency.

---

# Spring Boot / Java Notes

Spring Boot applications often run behind NAT in production environments.

Example:

```
Internet

        |

    Public IP

        |

    NAT Gateway

        |

 Private Network

        |

 Spring Boot Server
```

---

# Cloud Architecture Example

Typical AWS architecture:

```
Internet

   |

Internet Gateway

   |

Public Subnet

   |

NAT Gateway

   |

Private Subnet

   |

Spring Boot Application
```

---

# Why Keep Spring Boot Servers Private?

Bad design:

```
Internet

↓

Spring Boot Server
```

Better:

```
Internet

↓

Load Balancer

↓

Private Spring Boot Server

↓

Database
```

Benefits:

- Reduced attack surface.
- Better security.
- Controlled access.

---

# Database Access Example

Spring Boot:

```
10.0.2.10
```

Database:

```
10.0.3.20
```

Both communicate using private IPs.

NAT is not needed for internal communication.

---

# Java Networking Notes

Java applications see:

```
Private IP

or

Proxy IP
```

depending on architecture.

Example:

User IP may be hidden because of:

```
Client

↓

NAT

↓

Load Balancer

↓

Spring Boot
```

Applications often use headers:

```
X-Forwarded-For
```

to identify original clients.

---

# Common Production Issues

## Application Cannot Access Internet

Check:

```
Private Server

↓

NAT Gateway

↓

Internet Gateway
```

---

## External User Cannot Access Application

Check:

```
Public IP

↓

Port Forwarding

↓

Firewall Rule

↓

Application Port
```

---

## Wrong Client IP Logging

Cause:

NAT hides original IP.

Solution:

Use:

- Proxy headers.
- Load balancer configuration.

---

# Quick Revision

- NAT converts private IPs to public IPs.
- NAT helps save IPv4 addresses.
- PAT allows many devices to share one public IP.
- NAT works mainly at Network Layer.
- NAT does not encrypt traffic.
- NAT is not a firewall.
- Private IPs are not directly reachable from Internet.
- Cloud systems use NAT Gateway for private servers.

---

# Interview Questions

### 1. What is NAT?

NAT converts private IP addresses into public IP addresses for Internet communication.

---

### 2. Why do we use NAT?

To save IPv4 addresses and allow private networks to access the Internet.

---

### 3. How does NAT work?

NAT modifies source/destination IP information and maintains a translation table.

---

### 4. What is PAT?

Port Address Translation allows multiple devices to share one public IP using different ports.

---

### 5. Difference between NAT and Firewall?

NAT translates addresses.

Firewall controls allowed and blocked traffic.

---

### 6. Can NAT provide security?

NAT hides internal IP addresses but is not a complete security mechanism.

---

### 7. What are private IP ranges?

```
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

---

### 8. Why are private IPs not used directly on the Internet?

Because they are not globally routable.

---

### 9. What is NAT Gateway?

A cloud service that allows private resources to access the Internet securely.

---

### 10. Why should Spring Boot developers understand NAT?

Because production applications often run inside private networks using NAT, load balancers, and cloud networking.

---

# References

- [[IP]]
- [[CIDR]]
- [[Ports]]
- [[Routing]]
- [[Firewall]]
- [[VPN]]
- [[Proxy]]
- [[Reverse Proxy]]
- [[Load Balancer]]
- [[TCP-IP]]