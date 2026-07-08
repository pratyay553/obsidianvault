---
title: "Authentication vs Authorization"
chapter: 02
series: "Spring Security Handbook"
topic: "Authentication vs Authorization"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: "Intermediate"
tags:
  - java
  - spring-boot
  - spring-security
  - authentication
  - authorization
  - security
  - backend
  - rest-api
  - jwt
created: 2026-06-26
status: learning
prerequisites:
  - Java fundamentals
  - Spring Boot basics
  - REST API basics
related:
  - SecurityContextHolder
  - Authentication
  - Authorization
  - JWT
  - OAuth2
---
---


# Chapter 2: Authentication vs Authorization

> Previous Chapter: [[01-Authentication-Flow]]

---

## 🎯 How to Use This Notebook

Each concept below follows the same pattern: **definition → analogy → backend example → Spring internals → common mistake → say-it-like-this spoken answer.** Read it top to bottom once to build understanding. Then use [Section 9](https://claude.ai/chat/9b085ae4-8072-4aba-ad1e-bf67a6e7d5df#9-quick-revision-sheet) and the Q&A bank ([Section 10](https://claude.ai/chat/9b085ae4-8072-4aba-ad1e-bf67a6e7d5df#10-interview-qa-bank-natural-spoken-style)) the night before an interview — those two sections alone should let you explain this topic confidently without sounding memorized.

---

## 1. Learning Objectives

After this chapter you should be able to:

- Clearly separate Authentication ("who are you") from Authorization ("what can you do")
- Explain why backend systems always check identity _before_ permissions
- Describe how Spring Security represents a logged-in user internally
- Distinguish Role vs Permission and justify why permission-based design scales better
- Explain `Authentication`, `Principal`, `Credentials`, and `GrantedAuthority` and how they relate
- Correctly use HTTP `401` vs `403` and justify the difference under interview pressure
- Walk through a complete request lifecycle from HTTP request to controller execution
- Debug common authentication/authorization failures methodically

---

## 2. Why This Concept Exists

Every backend request needs two separate questions answered, in this exact order:

```
Question 1: "Who is this person?"          → Authentication
Question 2: "What can this person do?"      → Authorization
```

**Example — login:**

```
User sends: username=john, password=password123
System verifies: Is this really John?
→ Authentication
```

**Example — after login:**

```
John is logged in. Can John:
  GET    /users
  DELETE /users/10
  CREATE /products
→ Authorization
```

### The mistake everyone makes early in their career

> "User logged in = user can access everything."

This is **wrong**. Authentication only proves identity. It says nothing about what that identity is allowed to do — that's a completely separate check, performed separately, often per-endpoint.

---

## 3. Simple Definitions

### Authentication

> Answers: **WHO ARE YOU?**

Verifies identity using username/password, JWT, OAuth2, or an API key.

```
POST /login
username=admin
password=secret
→ Authenticated successfully
```

### Authorization

> Answers: **WHAT ARE YOU ALLOWED TO DO?**

Checked _after_ authentication, on a per-action basis.

```
Authenticated user: username=john, role=USER
Request: DELETE /users/10
System checks: Does USER have DELETE permission?
→ No → 403 Forbidden
```

---

## 4. Real-World Analogy

Entering a company building:

**Authentication** — the security guard asks for your ID card. You prove "I am Rahul." Identity confirmed.

**Authorization** — once inside, can Rahul enter the CEO's room? The server room? A normal office? Your **access level** decides — not your identity alone.

> **Say it like this:** "Authentication is like showing your ID card to a security guard. Once the guard knows who you are, authorization decides which rooms you can actually enter. Showing the ID doesn't mean you can go everywhere — different people get different access."

---

## 5. The Backend Engineer's Mental Model

```
                HTTP Request
                      │
                      ▼
              ┌───────────────┐
              │ Who is calling? │   ← Authentication
              └────────┬────────┘
                       ▼
              ┌───────────────────┐
              │ What can they do?  │   ← Authorization
              └────────┬────────────┘
                       ▼
                 Business Logic
```

**Example:**

```
GET /api/orders
Authorization: Bearer eyJhbGci...
```

Backend thinking, step by step:

```
Step 1 (Authentication): Is this token valid?
Step 2 (Authorization):  Does this user have ORDER_READ permission?
```

---

## 6. Internal Spring Security Connection

Spring Security keeps the authenticated user's information in one central object:

```
Authentication  →  stored inside  →  SecurityContextHolder
```

### Login-to-storage flow

```
User
  │  Login Request
  ▼
AuthenticationManager
  │
  ▼
AuthenticationProvider
  │
  ▼
UserDetailsService
  │
  ▼
Authentication object created
  │
  ▼
SecurityContextHolder
```

### Authentication architecture (diagram)

```
Client
  │  username/password
  ▼
Authentication Filter
  │
  ▼
AuthenticationManager
  │
  ▼
AuthenticationProvider
  │
  ▼
UserDetailsService
  │
  ▼
Database
```

### Authorization architecture (diagram)

```
Request
  │
  ▼
Authenticated User (from SecurityContext)
  │
  ▼
GrantedAuthority (roles / permissions)
  │
  ▼
Authorization check
  │
  ▼
Allow / Deny
```

---

## 7. Full Request Lifecycle

Example request:

```
GET /api/admin/users
Authorization: Bearer token
```

```
HTTP Request
     │
     ▼
SecurityFilterChain
     │
     ▼
JWT Authentication Filter   (validates the token)
     │
     ▼
Authentication object created
     │
     ▼
SecurityContextHolder       (current user now known)
     │
     ▼
Authorization Check         (does this user have the right role/authority?)
     │
     ▼
Controller                  (only reached if both checks pass)
     │
     ▼
Response
```

> **Say it like this:** "When a request comes in, it first goes through the security filter chain. If JWT is used, the filter validates the token and Spring builds an Authentication object, which gets stored in the SecurityContextHolder. Only after that does Spring check whether the user has the right role or permission for this specific endpoint. The controller only ever sees requests that already passed both checks — so my controller code stays clean."

---

## 8. Spring Security Classes Involved

### `Authentication`

Package: `org.springframework.security.core.Authentication`

Holds: **Principal, Credentials, Authorities, authenticated status.**

```java
Authentication authentication =
    SecurityContextHolder.getContext().getAuthentication();
```

### `Principal`

Represents the logged-in user — usually a `UserDetails` object.

```java
UserDetails user = (UserDetails) authentication.getPrincipal();
```

### `Credentials`

Sensitive proof of identity — password, JWT, API key. **Removed from memory after successful authentication** for security.

### `GrantedAuthority`

Represents a granted permission: `ROLE_ADMIN`, `READ_USER`, `DELETE_ORDER`.

```java
authentication.getAuthorities();
```

### Roles vs Permissions

||Role|Permission|
|---|---|---|
|Granularity|High-level grouping|Fine-grained action|
|Example|`ROLE_ADMIN`, `ROLE_USER`|`USER_DELETE`, `ORDER_CREATE`|
|Checked with|`hasRole("ADMIN")`|`hasAuthority("USER_DELETE")`|

```
ROLE_ADMIN contains →  USER_DELETE, ORDER_DELETE, REPORT_VIEW
```

---

## 9. Quick Revision Sheet

Read this right before an interview.

- **Authentication = Who are you? Authorization = What can you do?**
- Authentication always happens **first** — you can't check permissions for an unknown identity.
- Spring stores the current user in `Authentication`, kept inside `SecurityContextHolder`.
- `Authentication` contains: **Principal, Credentials, Authorities, authenticated flag.**
- `Principal` = the logged-in user (usually `UserDetails`). `Credentials` = sensitive proof (password/token), wiped after login.
- `GrantedAuthority` = a single granted permission or role string.
- **Roles** = coarse grouping (`ADMIN`, `USER`). **Permissions** = fine-grained actions (`DELETE_USER`). Prefer permissions for scalability — don't create `ROLE_DELETE_USER` style roles.
- `401 Unauthorized` = not authenticated (missing/invalid/expired token).
- `403 Forbidden` = authenticated, but lacks permission for this action.
- JWT/token validation happens in the **filter chain, before the controller** — controllers never check tokens themselves.
- A user can be **authenticated and still not authorized** — these are two independent checks, and authorization is re-checked on every request, not just once at login.

---

## 10. Interview Q&A Bank (Natural, Spoken-Style)

These answers are written the way you'd actually say them in an interview — clear and logical, not copied from documentation. Perfectly fine to explain in simple English.

**Q1. What is the difference between Authentication and Authorization?** Authentication means checking who the user is — when I enter username and password, the app verifies whether I'm really that user. Authorization comes after — it checks what I'm allowed to do. Logging in successfully doesn't mean I can delete users or access an admin page; the app checks my role or permission before allowing that.

**Q2. User logged in successfully but gets 403. Why?** That means authentication already succeeded — the system knows who the user is. The problem is in authorization: the user probably doesn't have the required role or permission for that specific endpoint.

**Q3. Why do we need both Authentication and Authorization?** Because they solve different problems. Authentication only tells the app who the user is — but not every user should get the same access. In an e-commerce app, customers can place orders, but only admins can delete products. So after identifying the user, we still need to check what they're allowed to do.

**Q4. Can Authorization happen before Authentication?** No. The system first needs to know who the user is. Without an identity, there's nothing to check permissions against.

**Q5. Explain Authentication in Spring Security.** When a login request comes in, Spring checks the username/password or JWT. If it's correct, Spring builds an `Authentication` object and stores it in `SecurityContextHolder`. From then on, every request can identify the current user from that object.

**Q6. What is the `Authentication` object?** It stores information about the logged-in user — principal, credentials, authorities, and whether the user is currently authenticated. Spring uses this object on every request to know who's calling.

**Q7. Why doesn't Spring Security just store the username?** Because the username alone isn't enough — Spring also needs roles/permissions and sometimes other details like email or ID. Bundling everything into one `Authentication` object makes it easy to access later.

**Q8. What is Principal?** The current logged-in user — typically a `UserDetails` object, from which I can get the username and other details.

**Q9. What are Credentials?** The proof used to verify identity — usually the password during login. After successful authentication, Spring typically clears it from memory for security.

**Q10. Why are credentials removed after authentication?** Because the password is sensitive. There's no need to keep it once login is done — if someone got access to application memory, they shouldn't be able to see it.

**Q11. Walk me through the request flow.** The request first goes through the `SecurityFilterChain`. If JWT is used, the JWT filter validates the token. If valid, Spring creates an `Authentication` object and stores it in `SecurityContextHolder`. The request then reaches the controller — but before the controller method runs, Spring checks whether the user has permission for that endpoint. If yes, it continues; if not, it returns 403.

**Q12. Where is the JWT actually verified?** Inside the security filter, before the request ever reaches the controller — so the controller only sees requests from already-authenticated users, keeping controller code clean.

**Q13. What is `SecurityContextHolder`?** It's where Spring keeps the current authenticated user's information. Whenever I need the logged-in user, I read it from there instead of re-parsing the token.

**Q14. Difference between Role and Permission?** A role is a group of permissions — e.g. `ADMIN` might internally include `DELETE_USER`, `CREATE_PRODUCT`, `VIEW_REPORT`. Using permissions gives more flexibility because different roles can share the same permission.

**Q15. Why not create a role for every action?** If every action becomes its own role, role management becomes a mess. It's easier to keep a small set of roles (`ADMIN`, `USER`, `MANAGER`) and assign fine-grained permissions to each — much easier to maintain as the app grows.

**Q16. Difference between 401 and 403?** 401 means the user isn't authenticated — no token, or it's invalid/expired. 403 means the user _is_ authenticated, but doesn't have permission for that specific resource.

**Q17. Valid JWT sent, but still getting 401. What do you check?** First, whether the `Authorization` header is actually present. Then whether the token is expired, whether the signature is correct, and finally whether the JWT filter itself is wired correctly into the filter chain.

**Q18. User has ADMIN role but still gets 403. What do you check?** First whether the role is stored correctly in the database/token. Then whether the code is checking `hasRole()` vs `hasAuthority()` correctly — a common bug is the role stored as `ROLE_ADMIN` but the code checking for plain `ADMIN`, causing a mismatch. Also worth checking the security config itself.

**Q19. Why is authentication done once but authorization checked every request?** Authentication proves identity, so once logged in, we already know who the user is. But different APIs have different access rules — one might be public, another needs `USER`, another needs `ADMIN`. So authorization has to be evaluated per request, per endpoint.

**Q20. Explain this to a non-technical person.** I'd use a building example: authentication is like showing your ID card to a security guard. Once the guard knows who you are, authorization decides which rooms you're allowed to enter. Showing the card doesn't mean you can go everywhere — different people get different access.

---

## 11. Deeper / Logical Interview Questions (For Senior-Track Prep)

These probe reasoning rather than definitions — good for 2+ years experience or system-design-flavored interviews. Think through your own answer before checking the hint.

**Conceptual reasoning**

1. A user logs in successfully but gets `403 Forbidden`. What can you immediately conclude? _(Hint: authentication succeeded, authorization failed — check role/permission mapping.)_
2. A client calls a protected endpoint with no JWT at all — should it be `401` or `403`? Why?
3. Bob steals Alice's valid JWT and uses it. Has "authentication" technically succeeded? What security problem does this reveal? _(Hint: yes technically — this is why token expiry, refresh rotation, and HTTPS matter.)_
4. Can authorization ever happen before authentication? Why not?
5. Why does Spring Security deliberately separate authentication and authorization into different stages instead of doing both in one step?

**Internals** 6. Where does Spring Security store the logged-in user's info after login, and why centralize it there instead of passing the user through every method? 7. Why doesn't `Authentication` just store a username string? 8. Why is `Principal` kept separate from `Credentials`? 9. Why are credentials typically wiped after authentication completes? 10. If `authentication.isAuthenticated()` is `true`, does that guarantee access to every endpoint? Why not?

**Request flow** 11. Trace `GET /api/orders` with a Bearer JWT end-to-end through Spring Security until the controller runs. 12. At what exact stage is the JWT verified — before, inside, or after the controller? Why does that matter for security? 13. Who creates the `Authentication` object, and at what point in the chain? 14. When exactly does `SecurityContextHolder` get populated? 15. Why doesn't the controller need to manually verify tokens?

**Role vs Permission design** 16. Why is `ROLE_ADMIN` + permissions like `DELETE_ORDER` a better design than creating `ROLE_DELETE_ORDER`? 17. Can two different roles share the same permission? Give a real example. _(e.g. both `MANAGER` and `ADMIN` might share `VIEW_REPORT`.)_ 18. At 500+ endpoints, why does permission-based authorization scale better than pure role-based checks?

**Architecture & design** 19. What problems would appear if authentication and authorization were merged into a single step? 20. Why is authentication typically a one-time event per session/token, while authorization is evaluated on nearly every request? 21. An admin's role is revoked in the database while they still hold a valid JWT. Should access be revoked immediately? Compare this for JWT-based vs session-based authentication. _(Hint: stateless JWTs can't be revoked without extra infrastructure — token blacklists, short expiry + refresh tokens, or switching to session-based revocation.)_

**Debugging** 22. `hasRole("ADMIN")` is configured, the user has `ADMIN` — but access is still denied. What would you check? _(Authority prefix mismatch — `ROLE_ADMIN` vs `ADMIN` — is the classic cause.)_ 23. JWT is valid, user exists, password is correct, yet every request returns 401. What components would you inspect first? 24. Every endpoint returns 403, even for admins. Where do you start debugging? 25. `authentication.getPrincipal()` returns a plain `String` instead of `UserDetails` — why might that happen? _(Hint: often happens with certain auth providers/anonymous authentication where no custom `UserDetails` was set.)_

**System design** 26. A banking app needs: view balance, transfer money, approve transfers, freeze accounts. Would you rely on roles alone? Why or why not? 27. A hospital system has Doctor, Nurse, Receptionist, Lab Technician, Administrator — with overlapping permissions. How would you model roles vs permissions here? 28. If your company adds OAuth2 login tomorrow, does your authorization logic need to change? Why or why not? _(Hint: ideally no — authorization should be decoupled from how identity was established.)_ 29. Can authentication exist without authorization? Can authorization exist without authentication? Give an example of each. _(A public read-only endpoint can skip both; an internal service-to-service call might authenticate via mTLS but have no fine-grained authorization at all.)_ 30. What's the single biggest security risk when authentication is implemented correctly but authorization is implemented poorly? _(Hint: privilege escalation — any authenticated user can perform actions meant for higher-privileged roles, simply by guessing or calling internal endpoints directly.)_

---

## 12. One Answer to Rule Them All

If asked **"Explain Authentication vs Authorization in your own words"**, this single answer covers the whole chapter cleanly:

> "Authentication and authorization solve two different problems. Authentication checks who the user is — username, password, or a JWT token — and once that's verified, Spring Security builds an Authentication object and stores it in the SecurityContextHolder. Authorization happens after that, on almost every request, checking whether this already-identified user has the right role or permission for the specific action they're trying to do. If identity verification fails, the response is 401. If identity is fine but the permission is missing, it's 403. I try to design permissions rather than dozens of specific roles, because that scales much better as the application grows."

---

## Navigation

- Previous Chapter: [[01-Authentication-Flow]]
- Next Chapter: [[03-Authentication-Flow-Internals]]