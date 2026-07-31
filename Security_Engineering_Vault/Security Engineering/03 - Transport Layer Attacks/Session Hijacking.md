```markdown
# Session Hijacking

## Definition

**Session Hijacking** is a security attack where an attacker steals or takes control of a user's active session by obtaining the user's **session identifier** (such as a cookie or token).

Instead of stealing the username and password, the attacker steals the already authenticated session.

> **Simple Definition**
>
> "Session Hijacking means stealing a user's active login session and using it as the victim."

Example:

Normal:

```

User Login

↓

Server Creates Session ID

↓

User Uses Website

```

During attack:

```

User Session ID

↓

Attacker Steals It

↓

Attacker Becomes Logged-in User

```

---

# How it Works

## Normal Session Flow

Example:

### Step 1: User Login

```

User

↓

Username + Password

↓

Server

```

---

### Step 2: Server Creates Session

Server returns:

```

Session Cookie:

JSESSIONID=abc123

```

---

### Step 3: Browser Sends Cookie

Every request:

```

Browser

↓

Cookie: JSESSIONID=abc123

↓

Server

```

The server identifies the user using the session ID.

---

# During Session Hijacking

The attacker obtains:

```

JSESSIONID=abc123

```

Then sends:

```

Attacker Browser

↓

Cookie: JSESSIONID=abc123

↓

Server

```

The server thinks:

```

This is the real user

```

---

# Common Session Hijacking Techniques

## 1. Cookie Theft

Attacker steals session cookies.

Possible sources:

- Malware.
- Browser compromise.
- XSS attacks.

---

## 2. Packet Sniffing

Attacker captures unencrypted traffic.

Example:

```

HTTP Traffic

↓

Session Cookie

↓

Attacker

```

---

## 3. Man-in-the-Middle Attack

Attacker intercepts communication.

```

User

↓

Attacker

↓

Server

```

---

## 4. Session Fixation

Attacker forces a known session ID on a victim.

Example:

```

Attacker Session

↓

Victim Login

↓

Attacker Uses Same Session

```

---

## 5. Cross-Site Scripting (XSS)

Malicious JavaScript steals session information.

Example:

```

Browser

↓

Malicious Script

↓

Cookie Theft

```

---

# Attack Flow

```

User Login

↓

Server Creates Session Token

↓

Attacker Steals Token

↓

Attacker Sends Token

↓

Server Accepts Session

↓

Unauthorized Access

```

---

# Target Layer

Session Hijacking mainly targets:

## Layer 7 (Application Layer)

Reason:

- Sessions are managed by applications.
- Authentication tokens and cookies are application-level data.

Supporting layers:

- Layer 4 → TCP communication.
- Layer 6 → Encryption mechanisms.

---

# Prerequisites

For Session Hijacking:

- Active user session.
- Access to session identifier.
- Weak session management.
- Missing encryption.

Common vulnerable situations:

- HTTP instead of HTTPS.
- Long session expiration.
- Weak cookies.
- Poor token management.

---

# Real-world Example

A user logs into an e-commerce website.

Server creates:

```

Cookie:

SESSION=xyz789

```

Attacker steals:

```

SESSION=xyz789

```

The attacker opens:

```

shop.com/account

```

with the stolen cookie.

Result:

```

Attacker accesses user's account

```

Possible actions:

- View orders.
- Change address.
- Access payment information.

---

# Impact

Possible consequences:

- Account takeover.
- Unauthorized transactions.
- Data theft.
- Identity theft.
- Privacy violation.
- Financial loss.

For e-commerce systems:

- Fraudulent purchases.
- Customer data exposure.
- Reputation damage.

---

# Detection

Session hijacking detection is difficult because the attacker uses a valid session.

Possible indicators:

- Same account used from different locations.
- Impossible travel patterns.
- Multiple devices using one session.
- Suspicious IP changes.
- Unusual user behavior.

Security monitoring:

- Authentication logs.
- Session activity logs.
- SIEM alerts.

Example:

```

User Login:

India

↓

5 seconds later

↓

Europe

```

Possible suspicious activity.

---

# Prevention

## 1. Always Use HTTPS ✅

HTTPS protects session cookies from network interception.

```

Browser

↓

Encrypted TLS Connection

↓

Server

```

---

## 2. Secure Cookies

Use:

```

Secure

↓

Only HTTPS transmission

HttpOnly

↓

JavaScript cannot access cookie

SameSite

↓

Reduces CSRF attacks

```

Example:

```

Set-Cookie:

SESSION=abc123;

Secure;

HttpOnly;

SameSite=Strict

```

---

## 3. Session Expiration

Use:

- Short idle timeout.
- Maximum session lifetime.
- Automatic logout.

---

## 4. Regenerate Session After Login

Prevent session fixation.

Example:

```

Before Login:

Session A

After Login:

New Session B

```

---

## 5. Multi-Factor Authentication

Even if a session is compromised, MFA reduces account takeover risk.

---

## 6. Token Security

For JWT:

Use:

- Short expiration.
- Refresh tokens.
- Token rotation.

---

# Spring Boot Protection

Spring Boot applications must protect session management.

---

## 1. Enable HTTPS

Production architecture:

```

Client

↓

HTTPS

↓

Spring Boot Application

````

---

## 2. Spring Security Session Management

Example:

```java
http
.sessionManagement(session ->
    session
    .sessionFixation()
    .newSession()
);
````

Purpose:

Creates a new session after authentication.

---

## 3. Secure Cookies

application.properties:

```properties
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
```

---

## 4. Configure Session Timeout

Example:

```properties
server.servlet.session.timeout=15m
```

---

## 5. JWT Security

For REST APIs:

```
Client

↓

JWT Token

↓

Spring Security Filter

↓

Controller
```

Best practices:

- Short token expiry.
    
- Secure storage.
    
- Token validation.
    

---

## 6. CSRF Protection

Enable CSRF protection for session-based applications.

---

# Infrastructure Protection

Enterprise architecture:

```
Internet

↓

WAF

↓

Load Balancer

↓

API Gateway

↓

Spring Boot Application

↓

Database
```

Security controls:

- TLS termination.
    
- WAF rules.
    
- Rate limiting.
    
- Bot detection.
    
- SIEM monitoring.
    
- Identity management.
    

---

# Best Practices

- Use HTTPS everywhere.
    
- Never store sensitive data in cookies.
    
- Use Secure + HttpOnly cookies.
    
- Implement session timeout.
    
- Rotate session identifiers.
    
- Enable MFA.
    
- Monitor unusual login behavior.
    
- Use strong authentication.
    
- Protect JWT tokens.
    
- Avoid storing tokens in insecure locations.
    
- Implement logout functionality.
    
- Follow Zero Trust principles.
    

---

# Interview Questions

### 1. What is Session Hijacking?

Session Hijacking is an attack where an attacker steals a user's active session token and uses it to impersonate the user.

---

### 2. What is the main target of Session Hijacking?

Application sessions, cookies, and authentication tokens.

---

### 3. Which OSI layer does Session Hijacking target?

Mainly Layer 7 (Application Layer).

---

### 4. What is a session ID?

A unique identifier used by a server to recognize an authenticated user's session.

---

### 5. How does HTTPS prevent Session Hijacking?

HTTPS encrypts communication, preventing attackers from easily capturing session cookies.

---

### 6. What are HttpOnly and Secure cookie flags?

|Flag|Purpose|
|---|---|
|HttpOnly|Prevents JavaScript access|
|Secure|Sends cookie only over HTTPS|

---

### 7. What is Session Fixation?

An attack where an attacker forces a known session ID and uses it after the victim logs in.

---

### 8. Can Spring Boot prevent Session Hijacking?

Yes.

Using:

- Spring Security.
    
- Secure cookies.
    
- HTTPS.
    
- Session rotation.
    
- Timeout policies.
    

---

### 9. Difference between Authentication and Session Hijacking?

|Authentication|Session Hijacking|
|---|---|
|Proves user identity|Steals existing identity|
|Uses credentials|Uses session token|
|Happens during login|Happens after login|

---

### 10. How do you secure JWT tokens?

Using:

- Short expiration.
    
- Refresh token rotation.
    
- HTTPS.
    
- Secure storage.
    
- Token validation.
    

---

# References

- [[HTTP]]
    
- [[HTTPS]]
    
- [[TLS]]
    
- [[Cookie]]
    
- [[JWT]]
    
- [[OAuth2]]
    
- [[Spring Security]]
    
- [[CSRF]]
    
- [[XSS]]
    
- [[Man-in-the-Middle]]
    
- [[Packet Sniffing]]
    
- [[OSI Model]]
    
- [[TCP-IP]]