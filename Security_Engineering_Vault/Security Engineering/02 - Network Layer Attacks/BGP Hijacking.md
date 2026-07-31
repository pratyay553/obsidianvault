```markdown
# BGP Hijacking

## Definition

**BGP Hijacking** is a routing attack where an attacker or misconfigured network **announces false BGP (Border Gateway Protocol) routes** to redirect Internet traffic through an unauthorized network.

BGP is the protocol that connects different networks (Autonomous Systems) across the Internet.

> **Simple Definition**
>
> "BGP Hijacking happens when someone announces fake Internet routes and steals or redirects traffic."

Example:

Normal:

```

User

↓

ISP

↓

Bank Network

```

After BGP Hijacking:

```

User

↓

ISP

↓

Attacker Network

↓

Bank Network

```

---

# How it Works

## Normal BGP Operation

BGP allows networks to exchange routing information.

Example:

```

Company Network

10.10.0.0/16

↓

Internet Service Provider

↓

Global Internet

```

Routers learn:

```

10.10.0.0/16 belongs to Company

```

Traffic is forwarded correctly.

---

## During BGP Hijacking

The attacker advertises ownership of someone else's IP prefix.

Example:

Real owner:

```

Company

owns

203.0.113.0/24

```

Attacker announces:

```

Attacker Network

owns

203.0.113.0/24

```

Other routers may believe the false announcement.

Traffic moves:

```

Users

↓

Attacker Network

↓

Original Network

```

or:

```

Users

↓

Attacker Network

↓

Traffic Dropped

```

---

# Attack Flow

```

Attacker

↓

Fake BGP Announcement

↓

Internet Routers Update Routes

↓

Traffic Redirected

↓

Attacker Network

```

---

# Types of BGP Hijacking

## 1. Origin Hijacking

Attacker claims ownership of an IP prefix.

Example:

```

Real:

AS100 owns 10.0.0.0/24

Fake:

AS200 owns 10.0.0.0/24

```

---

## 2. Route Leak

A legitimate network accidentally advertises routes it should not announce.

---

## 3. Man-in-the-Middle BGP Hijacking

Traffic is redirected through the attacker, who forwards it after inspection.

---

# Target Layer

**OSI Layer:** Layer 3 (Network Layer)

Reason:

- BGP controls IP routing.
- The attack manipulates Internet routing tables.

---

# Prerequisites

For BGP Hijacking:

- Access to a BGP-speaking router.
- Ability to advertise routes.
- Weak route validation.
- Lack of proper filtering.

Common attackers:

- Malicious networks.
- Compromised routers.
- Misconfigured ISPs.

---

# Real-world Example

A company hosts a website:

```

example.com

IP:

203.0.113.10

```

The attacker announces:

```

203.0.113.0/24

belongs to attacker

```

Internet routers update their routes.

Users now connect:

```

User

↓

Attacker Network

↓

Fake Website

````

Possible outcomes:

- Traffic interception.
- Data theft.
- Website outage.

---

# Impact

Possible consequences:

- Website downtime.
- Traffic interception.
- Credential theft.
- Data modification.
- Malware delivery.
- Loss of customer trust.
- Internet service disruption.

BGP Hijacking can affect:

- Companies.
- Cloud providers.
- ISPs.
- Government networks.

---

# Detection

Signs of BGP Hijacking:

- Unexpected traffic paths.
- Sudden route changes.
- Website unreachable from some regions.
- Increased latency.
- Certificate warnings.

Useful tools:

- BGP monitoring systems.
- Route collectors.
- Internet routing databases.
- Network monitoring platforms.

Examples:

- RouteViews
- RIPE RIS
- BGPStream

Network checks:

```bash
traceroute example.com
````

Look for unexpected autonomous systems (AS).

---

# Prevention

## 1. Route Filtering ✅

ISPs should allow only authorized prefixes.

Example:

```
Customer A

Allowed:

203.0.113.0/24
```

Anything else is rejected.

---

## 2. RPKI (Resource Public Key Infrastructure)

RPKI verifies:

```
IP Prefix

+

Authorized Owner

+

Valid Route Origin
```

---

## 3. BGP Authentication

Use:

- TCP MD5 authentication.
    
- TCP-AO where supported.
    

---

## 4. Prefix Monitoring

Monitor your IP ranges for unauthorized announcements.

---

## 5. Multi-Homing Security

Organizations connected to multiple ISPs should maintain proper routing policies.

---

# Spring Boot Protection

Spring Boot cannot prevent BGP Hijacking because it occurs at the Internet routing level.

However, applications should:

## Use HTTPS/TLS

Example:

```
User

↓

BGP Route

↓

Server

↓

TLS Certificate Validation
```

A fake server cannot easily impersonate the real application without a valid certificate.

---

## Use Strong Authentication

Never depend on:

- IP address.
    
- Network location.
    
- Routing path.
    

Use:

- OAuth2.
    
- JWT.
    
- mTLS.
    
- Strong identity verification.
    

---

## Availability Protection

Use:

- Multiple regions.
    
- Multiple cloud providers.
    
- CDN.
    
- Health monitoring.
    

---

# Infrastructure Protection

Enterprise-grade protection:

- Deploy RPKI validation.
    
- Use BGP route filtering.
    
- Monitor BGP announcements.
    
- Maintain accurate IP ownership records.
    
- Use multiple network providers.
    
- Use DDoS protection.
    
- Use encrypted communication.
    

Architecture:

```
Internet

↓

CDN

↓

DDoS Protection

↓

Firewall

↓

Load Balancer

↓

Spring Boot Application
```

---

# Best Practices

- Enable RPKI.
    
- Monitor your IP prefixes.
    
- Maintain BGP security policies.
    
- Use HTTPS everywhere.
    
- Use certificate transparency monitoring.
    
- Avoid IP-based authentication.
    
- Use redundant network providers.
    
- Keep network devices secure.
    
- Regularly audit routing configuration.
    

---

# Interview Questions

### 1. What is BGP Hijacking?

BGP Hijacking is an attack where false BGP route announcements redirect Internet traffic to an unauthorized network.

---

### 2. Which OSI layer does BGP Hijacking target?

Layer 3 (Network Layer).

---

### 3. What is BGP?

BGP (Border Gateway Protocol) is the routing protocol used to exchange routes between autonomous systems on the Internet.

---

### 4. What is an Autonomous System (AS)?

A collection of networks managed by one organization that follows a common routing policy.

Example:

- ISP
    
- Cloud provider
    
- Enterprise network
    

---

### 5. What is RPKI?

RPKI is a security framework that validates whether an AS is authorized to announce an IP prefix.

---

### 6. Can HTTPS prevent BGP Hijacking?

No.

HTTPS cannot stop routing attacks, but it protects data integrity and confidentiality.

---

### 7. Can Spring Boot prevent BGP Hijacking?

No.

The attack occurs before traffic reaches the application.

---

### 8. How do organizations protect against BGP Hijacking?

Using:

- RPKI
    
- Route filtering
    
- BGP monitoring
    
- Prefix validation
    
- Multiple providers
    

---

### 9. Difference between BGP Hijacking and DNS Spoofing?

|BGP Hijacking|DNS Spoofing|
|---|---|
|Changes Internet routing|Changes domain resolution|
|Works at network layer|Works at application layer|
|Affects IP paths|Affects domain-to-IP mapping|
|Targets routers|Targets DNS systems|

---

### 10. Why is BGP security difficult?

Because BGP was designed for connectivity and scalability, not strong security. Modern protections like RPKI were added later.

---

# References

- [[BGP]]
    
- [[Routing]]
    
- [[IP]]
    
- [[TCP-IP]]
    
- [[OSI Model]]
    
- [[DNS]]
    
- [[DNS Spoofing]]
    
- [[Firewall]]
    
- [[VPN]]
    
- [[HTTPS]]
    
- [[TLS]]
    
- [[Man-in-the-Middle (MITM)]]