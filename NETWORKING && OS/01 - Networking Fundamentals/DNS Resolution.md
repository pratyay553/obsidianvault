# DNS Resolution

## Definition

**DNS Resolution** is the process of converting a **domain name** (like `google.com`) into an **IP address** (like `142.250.183.14`) so a computer can communicate with the correct server.

> **Simple Definition**
>
> "DNS Resolution is the process of finding the IP address of a website from its domain name."

Example:

```
google.com

↓

142.250.xxx.xxx
```

Humans remember:

```
google.com
```

Computers use:

```
IP Address
```

---

# Why it Matters

Computers communicate using **IP addresses**, not domain names.

Imagine remembering:

```
172.217.167.78
```

instead of:

```
google.com
```

DNS makes the Internet easy to use.

Without DNS:

- You would need to remember IP addresses.
- Websites would be difficult to access.

---

# How it Works

When you type:

```
https://google.com
```

your computer performs DNS Resolution before creating a TCP connection.

Basic flow:

```
google.com

↓

DNS Resolution

↓

142.250.xxx.xxx

↓

TCP Handshake

↓

HTTPS Connection
```

---

# DNS Resolution Process

### Step 1: Browser Cache

The browser checks if it already knows the IP.

```
Browser Cache

↓

Found?

↓

Yes → Use IP
```

---

### Step 2: Operating System Cache

If not found:

```
Browser

↓

OS DNS Cache
```

---

### Step 3: Local DNS Resolver

Usually your:

- ISP
- Company DNS Server
- Public DNS

Examples:

- Google DNS (8.8.8.8)
- Cloudflare DNS (1.1.1.1)

---

### Step 4: Root DNS Server

If the resolver doesn't know the answer:

```
Resolver

↓

Root DNS Server
```

The root server tells where to find the **Top-Level Domain (TLD)** server.

---

### Step 5: TLD Server

For:

```
google.com
```

The TLD is:

```
.com
```

The TLD server tells where to find Google's authoritative DNS server.

---

### Step 6: Authoritative DNS Server

The authoritative server returns the actual IP address.

Example:

```
google.com

↓

142.250.xxx.xxx
```

---

### Step 7: Browser Connects

Now the browser connects to:

```
142.250.xxx.xxx

↓

TCP

↓

HTTPS
```

---

# Complete DNS Resolution Flow

```
User

↓

Browser Cache

↓

OS Cache

↓

DNS Resolver

↓

Root DNS

↓

TLD DNS

↓

Authoritative DNS

↓

IP Address

↓

TCP Handshake

↓

HTTPS Request

↓

Website
```

---

# DNS Records

DNS stores different types of records.

| Record | Purpose |
|---------|---------|
| A | Domain → IPv4 address |
| AAAA | Domain → IPv6 address |
| CNAME | Alias to another domain |
| MX | Mail server |
| NS | Name server |
| TXT | Verification and security information |

Example:

```
A Record

example.com

↓

192.168.1.10
```

---

# DNS Caching

To improve speed, DNS responses are cached.

```
First Request

↓

DNS Lookup

↓

Cache


Second Request

↓

Use Cache
```

Benefits:

- Faster browsing.
- Reduced DNS traffic.

---

# DNS Resolution in Enterprise

```
Employee

↓

Company DNS

↓

Internet

↓

Authoritative DNS

↓

Application Server
```

Companies often use internal DNS servers.

---

# DNS and HTTPS

DNS Resolution happens **before** HTTPS.

Flow:

```
Domain Name

↓

DNS Resolution

↓

IP Address

↓

TCP Handshake

↓

TLS Handshake

↓

HTTPS Request
```

---

# DNS vs DNS Resolution

| DNS | DNS Resolution |
|------|----------------|
| The naming system | The process of finding an IP address |
| Database of records | Lookup process |
| Stores domain information | Retrieves the correct IP |

---

# Common Pitfalls

## ❌ DNS stores websites.

Wrong.

DNS stores **domain records**, not website content.

---

## ❌ DNS Resolution happens after HTTPS.

Wrong.

DNS Resolution happens **before** the TCP and TLS handshakes.

---

## ❌ DNS always contacts the Root Server.

Wrong.

If the answer is cached, the resolver returns it immediately.

---

## ❌ DNS is only for websites.

Wrong.

DNS is also used for:

- Email (MX records)
- APIs
- Cloud services
- Internal company systems

---

## ❌ Domain name equals IP address.

Wrong.

A domain name maps to an IP address through DNS.

The IP can change without changing the domain name.

---

# Spring Boot / Java Notes

Spring Boot applications often connect to services using **hostnames**, not IP addresses.

Example:

```properties
spring.datasource.url=jdbc:mysql://db.company.com:3306/shop
```

Before connecting:

```
db.company.com

↓

DNS Resolution

↓

10.0.2.15

↓

Database Connection
```

---

# Microservices Example

```
Order Service

↓

payment-service.company.com

↓

DNS Resolution

↓

Payment Service
```

Applications don't need to know the actual IP address.

---

# Kubernetes Example

Kubernetes provides internal DNS.

Example:

```
order-service.default.svc.cluster.local

↓

DNS Resolution

↓

Pod IP
```

Services communicate using names instead of changing Pod IPs.

---

# Common Production Issues

## Unknown Host Exception

Example:

```
java.net.UnknownHostException
```

Possible causes:

- Wrong hostname.
- DNS server unavailable.
- Network issue.
- Missing DNS record.

---

## Slow Application Startup

Possible causes:

- Slow DNS server.
- DNS timeout.
- Network latency.

---

## Service Not Reachable

Check:

- DNS record exists.
- Correct hostname.
- Network connectivity.

---

# Quick Revision

- DNS Resolution converts a domain name into an IP address.
- Browsers and operating systems check caches first.
- If not cached, the resolver queries Root → TLD → Authoritative DNS servers.
- DNS Resolution happens **before** TCP and HTTPS.
- Common records:
  - A
  - AAAA
  - CNAME
  - MX
  - TXT
- Spring Boot applications usually connect using hostnames, not IP addresses.
- Kubernetes uses internal DNS for service discovery.

---

# Interview Questions

### 1. What is DNS Resolution?

DNS Resolution is the process of converting a domain name into an IP address.

---

### 2. Why is DNS Resolution needed?

Because computers communicate using IP addresses, while humans use domain names.

---

### 3. What is the order of DNS Resolution?

Browser Cache → OS Cache → DNS Resolver → Root DNS → TLD DNS → Authoritative DNS.

---

### 4. What is an Authoritative DNS Server?

It stores the official DNS records for a domain and returns the final IP address.

---

### 5. What is DNS caching?

Storing DNS responses temporarily to reduce lookup time and improve performance.

---

### 6. Does DNS Resolution happen before or after TCP Handshake?

Before the TCP Handshake.

---

### 7. What is an A Record?

It maps a domain name to an IPv4 address.

---

### 8. Why do Spring Boot applications use hostnames instead of IP addresses?

Hostnames remain stable even if the underlying server IP changes.

---

### 9. What causes `UnknownHostException` in Java?

Usually:

- Invalid hostname.
- DNS failure.
- Network problem.
- Missing DNS record.

---

### 10. Why should a Spring Boot developer understand DNS Resolution?

Because databases, APIs, microservices, cloud services, and Kubernetes all rely on DNS for service discovery and communication.

---

# References

- [[DNS]]
- [[IP]]
- [[TCP Handshake]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[TCP-IP]]
- [[Ports]]
- [[Routing]]
- [[Spring Boot Networking]]