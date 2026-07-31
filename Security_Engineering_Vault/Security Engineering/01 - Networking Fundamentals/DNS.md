# DNS (Domain Name System)

## Definition

**DNS (Domain Name System)** is a distributed naming system that converts **human-readable domain names into IP addresses** so that computers can communicate with each other.

> **Simple Definition**
>
> "DNS is the phonebook of the Internet. It converts website names into IP addresses."

Example:

Humans remember:

```
google.com
```

Computers communicate using:

```
142.250.182.14
```

DNS performs this conversion.

---

## Why it Matters

Without DNS, we would need to remember IP addresses for every website and application.

Instead of:

```
https://142.250.182.14
```

we use:

```
https://google.com
```

DNS provides:

- Easy access to websites.
- Service discovery.
- Load balancing.
- High availability.
- Domain management.

---

## How it Works

When you type:

```
www.example.com
```

into your browser, DNS resolution starts.

The process:

```
User Browser

↓

DNS Resolver

↓

Root DNS Server

↓

TLD DNS Server

↓

Authoritative DNS Server

↓

IP Address Returned

↓

Browser Connects to Server
```

---

## DNS Resolution Process

### Step 1: Browser Cache Check

The browser checks:

"Do I already know this domain's IP?"

Example:

```
google.com = 142.250.x.x
```

If found, it uses the cached value.

---

### Step 2: Operating System Cache

If browser cache fails:

The OS checks its DNS cache.

Example:

Windows:

```
ipconfig /displaydns
```

---

### Step 3: DNS Resolver

If still not found, the request goes to a DNS resolver.

Usually provided by:

- ISP
- Google DNS
- Cloudflare DNS

Examples:

```
Google DNS

8.8.8.8


Cloudflare DNS

1.1.1.1
```

---

### Step 4: Root DNS Server

Resolver asks:

"Who manages .com domains?"

Root server replies:

"Ask the .com TLD server."

---

### Step 5: TLD Server

TLD means:

Top Level Domain.

Examples:

```
.com

.org

.net

.in
```

The TLD server says:

"Ask the authoritative DNS server."

---

### Step 6: Authoritative DNS Server

This server contains the actual DNS records.

Example:

```
example.com

↓

93.184.216.34
```

It returns the IP address.

---

### Step 7: Browser Connects

Now the browser knows:

```
Domain

↓

IP Address
```

Then:

```
TCP Handshake

↓

TLS Handshake

↓

HTTP Request

↓

Server Response
```

---

## DNS Record Types

| Record | Purpose | Example |
|-|-|-|
| A | Domain → IPv4 | example.com → 192.168.1.10 |
| AAAA | Domain → IPv6 | example.com → IPv6 address |
| CNAME | Alias to another domain | www → example.com |
| MX | Mail server | Gmail mail server |
| TXT | Text information | SPF, verification |
| NS | Name server | DNS authority |
| PTR | IP → Domain lookup | Reverse DNS |

---

## Common DNS Terms

### Domain Name

Human-readable name.

Example:

```
amazon.com
```

---

### IP Address

Computer-readable address.

Example:

```
54.239.x.x
```

---

### DNS Resolver

Server that finds IP addresses.

Example:

```
8.8.8.8
```

---

### Authoritative DNS Server

The final source of truth for a domain.

---

## DNS Caching

DNS responses are stored temporarily.

This reduces:

- Network traffic.
- DNS lookup time.
- Server load.

Caching duration is controlled by:

```
TTL (Time To Live)
```

Example:

```
TTL = 3600 seconds

(1 hour)
```

---

## DNS and Load Balancing

DNS can return multiple IP addresses.

Example:

```
api.example.com

↓

10.1.1.10

10.1.1.20

10.1.1.30
```

Traffic can be distributed across servers.

---

## DNS Security

Common security features:

### DNSSEC

Provides DNS response authentication.

Protects against:

- DNS spoofing
- DNS cache poisoning

---

### Private DNS

Used inside companies.

Example:

Internal services:

```
payment.company.local

inventory.company.local
```

---

## Common Pitfalls

### ❌ DNS stores website content.

Wrong.

DNS only stores:

```
Domain → IP mapping
```

It does not store:

- HTML
- Images
- Application data

---

### ❌ DNS works only with websites.

Wrong.

DNS is used for:

- APIs
- Email
- Databases
- Cloud services

---

### ❌ DNS changes are immediate.

Wrong.

DNS changes depend on TTL.

Old records may remain cached.

---

### ❌ DNS and HTTP are the same.

Wrong.

DNS finds the server.

HTTP communicates with the server.

Flow:

```
DNS

↓

Find IP

↓

HTTP

↓

Exchange Data
```

---

### ❌ DNS always uses TCP.

Normally:

```
DNS Query → UDP Port 53
```

TCP is used for:

- Large responses.
- Zone transfers.
- DNSSEC cases.

---

## Spring Boot / Java Notes

Spring Boot applications are usually accessed through DNS names.

Example:

Instead of:

```
http://10.20.30.40:8080
```

we use:

```
https://api.company.com
```

DNS maps:

```
api.company.com

↓

Load Balancer IP
```

---

## Real Production Flow

Example:

User calls:

```
https://api.shop.com/orders
```

Flow:

```
User

↓

DNS Lookup

↓

Load Balancer IP

↓

Spring Boot Application

↓

Database
```

---

## Spring Boot Service Discovery

In microservices:

```
Order Service

↓

payment-service
```

DNS/service discovery helps locate services.

Examples:

- Kubernetes DNS
- Eureka
- Consul

---

## Common Java Networking APIs

Resolve hostname:

```java
InetAddress address =
InetAddress.getByName("google.com");

System.out.println(address);
```

Output:

```
google.com/142.250.x.x
```

---

## DNS Issues in Production

### DNS Resolution Failed

Possible causes:

- Wrong DNS configuration.
- DNS server unavailable.
- Domain expired.

---

### Application Cannot Reach Service

Check:

```
Domain

↓

DNS

↓

IP

↓

Port

↓

Firewall
```

---

### Slow API First Request

Possible reason:

DNS lookup delay.

---

## Quick Revision

- DNS converts domain names into IP addresses.
- DNS is like the Internet phonebook.
- DNS works at the Application Layer.
- DNS commonly uses UDP port 53.
- TCP is used for large DNS communication.
- DNS records store domain information.
- TTL controls DNS cache duration.
- DNS happens before HTTP communication.

---

## Interview Questions

### 1. What is DNS?

DNS is a system that translates domain names into IP addresses.

---

### 2. Why do we need DNS?

Because humans remember names better than IP addresses.

---

### 3. Explain DNS resolution.

Browser checks cache → DNS resolver → Root server → TLD server → Authoritative server → IP returned.

---

### 4. What is an A record?

A DNS record that maps a domain name to an IPv4 address.

---

### 5. Difference between A and CNAME record?

A record points directly to an IP address.

CNAME points one domain name to another domain name.

---

### 6. What is TTL in DNS?

TTL defines how long DNS information can be cached.

---

### 7. Which port does DNS use?

UDP port 53 normally.

TCP port 53 for specific cases.

---

### 8. What happens when DNS fails?

The browser cannot find the server IP, so communication cannot start.

---

### 9. How does DNS help cloud applications?

DNS maps user-friendly domains to load balancers, servers, and cloud services.

---

### 10. Why should a Spring Boot developer understand DNS?

Because APIs, microservices, cloud deployments, and production debugging depend heavily on DNS.

---

## References

- [[OSI Model]]
- [[TCP-IP]]
- [[TCP Handshake]]
- [[UDP]]
- [[IP]]
- [[HTTP]]
- [[HTTPS]]
- [[TLS]]
- [[Load Balancer]]
- [[Reverse Proxy]]
- [[Firewall]]