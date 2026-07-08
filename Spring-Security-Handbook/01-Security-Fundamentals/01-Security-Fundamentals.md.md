---
title: Spring Security Fundamentals
part: Spring Security Handbook
level: Beginner to Advanced
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
tags:
  - spring-security
  - backend
  - authentication
  - authorization
status: learning
---

# Chapter 0: Spring Security Fundamentals

> Beginner → Advanced Backend Engineer Java 21+ · Spring Boot 3.x+ · Spring Security 6.x+ · REST APIs · Microservices

---

## 🎯 How to Use This Notebook

This chapter is written to be **read before an interview, not during one**. Every section follows the same pattern:

- **Definition** — the one-line truth
- **Real-world analogy** — so you never forget it
- **Backend example** — so you can code it
- **Common mistake** — so you don't repeat it
- **Say it like this** — a natural, spoken answer you can actually say out loud to an interviewer, without sounding like you memorized documentation

Read top to bottom once. Then jump straight to [Section 6: Quick Revision Sheet](https://claude.ai/chat/9b085ae4-8072-4aba-ad1e-bf67a6e7d5df#6-quick-revision-sheet) the night before an interview.

---

## 1. Why This Concept Exists

Every backend application has to answer three questions before it does anything else:

```
Who is calling me?
What is this caller allowed to do?
Can I trust this request?
```

Without security, a backend looks like this:

```
   Client
     │
     ▼
   API  ───────► Database
```

Anyone can call any endpoint, see anyone else's data, or change anything. Spring Security exists so that a **standard, well-tested layer** sits between the client and your code, answering those three questions consistently — instead of every team writing their own (usually buggy) login and permission logic.

> **Say it like this:** "Before frameworks like Spring Security, every team wrote their own login and permission logic, and that usually had bugs. Spring Security gives a standard, tested way to handle identity and permissions, so I can focus on business logic instead of reinventing security."

---

## 2. The Mental Model

A request never goes straight to your business logic. It passes through a security layer first.

```
                 Client
                   │
                   │  HTTP Request
                   ▼
         ┌───────────────────┐
         │   Security Layer    │   "Who are you?"
         │                      │   "What can you do?"
         └──────────┬───────────┘
                     │  ✅ approved only
                     ▼
         ┌───────────────────┐
         │     Controller       │
         └──────────┬───────────┘
                     ▼
         ┌───────────────────┐
         │   Service Layer      │  (business logic)
         └──────────┬───────────┘
                     ▼
         ┌───────────────────┐
         │     Database          │
         └───────────────────┘
```

|Layer|Responsibility|Example|
|---|---|---|
|Client|Sends the request|`GET /api/orders` with `Authorization: Bearer <jwt>`|
|Security Layer|Checks identity + permission|"Is this JWT valid? Does the user have `ORDER_READ`?"|
|Controller|Receives only approved requests|`@GetMapping("/orders")`|
|Service|Business logic|calculate price, apply discount|
|Database|Stores users, roles, permissions, data|—|

> **Say it like this:** "I think of Spring Security like an airport. Security checks your ticket and ID before you even reach the gate. Similarly, every request first passes through Spring Security's filter layer, which checks identity and permission before it ever reaches my controller."

---

## 3. Core Concept: Application Security

### Definition

Protecting your software and its data from unauthorized access, leaks, and misuse.

### Real-world analogy

A bank protects account balances, transaction history, and customer identity — not everything is public, and not everyone gets the same access.

### Backend example — the classic mistake

```
❌ Insecure:
GET /users/101
```

Any logged-in user can change `101` to any other ID and view someone else's data.

```
✅ Secure:
GET /users/profile
```

The backend identifies the **current logged-in user** from the security context and returns only their own data — the ID is never trusted from the URL.

### Common mistake

Believing "the security team handles security." In reality, backend engineers write the APIs, queries, and access rules — security is part of the job, not a separate department's problem.

> **Say it like this:** "Application security isn't just a checklist someone else handles — every API I write, every query I design, decides what data is exposed. So I treat it as part of normal backend engineering, not an afterthought."

---

## 4. Core Concept: Authentication

### Definition

> Authentication answers: **"Who are you?"**

It verifies identity — nothing more.

### Real-world analogy

Airport security checking your passport before you're allowed past the gate.

### Backend example

```
POST /login
{
  "email": "rahul@test.com",
  "password": "secret"
}
```

Backend steps:

1. Find the user by email
2. Verify the password (hashed, never plain text)
3. Issue a session or a JWT representing "this is now Rahul"

### Spring Security's view of identity

```
Authentication
      │
      ▼
  Principal
      │
      ▼
  User (your domain object)
```

### Common authentication methods

|Method|Typical Use|
|---|---|
|Username + Password|Traditional login forms|
|JWT|Stateless REST APIs, microservices|
|OAuth2|"Login with Google", enterprise SSO|

### JWT flow (the one you'll be asked about most)

```
Login request
     │
     ▼
Validate credentials
     │
     ▼
Generate JWT
     │
     ▼
Client stores & sends JWT on every request
     │
     ▼
Backend validates token on each request
```

### Common mistake

Thinking "logged in" = "can do everything." Authentication only proves identity — it says nothing about permissions.

> **Say it like this:** "Authentication only answers 'who is this person?' It doesn't decide what they're allowed to do — that's a separate step. I keep those two ideas mentally separate, because mixing them up is a common source of security bugs."

---

## 5. Core Concept: Authorization

### Definition

> Authorization answers: **"What are you allowed to do?"** — and it always happens _after_ authentication.

### Real-world analogy

An employee badge gets you into the office (authentication). It does **not** mean you can walk into the server room (authorization).

### Backend example

```
DELETE /admin/users/10
```

|User|Result|
|---|---|
|Admin|✅ Allowed|
|Customer|❌ 403 Forbidden|

### Spring Security's permission model

- **Roles** — e.g. `ROLE_ADMIN`, `ROLE_USER`
- **Authorities / Permissions** — e.g. `ORDER_READ`, `ORDER_DELETE`

### Common mistake — frontend-only checks

```js
// ❌ This is NOT security, only UI convenience
if (user.isAdmin) {
  showDeleteButton();
}
```

A hacker doesn't need the button — they can call `DELETE /admin/users/10` directly with a tool like Postman. **The backend must enforce the check**, not just hide UI elements.

> **Say it like this:** "Hiding a button on the frontend is a UX decision, not a security control. Anyone can call the API directly with Postman or curl. So every sensitive action has to be checked again on the backend, regardless of what the UI shows."

---

### Authentication vs Authorization — Side by Side

||Authentication|Authorization|
|---|---|---|
|Question|Who are you?|What can you do?|
|Order|Happens first|Happens after authentication|
|Example|Logging in|Calling an admin-only API|
|Spring object|`Authentication`|`GrantedAuthority`|
|Failure response|`401 Unauthorized`|`403 Forbidden`|

### Important: you can be authenticated but not authorized

```
Authenticated  ✅
Authorized     ❌
```

A regular customer logged into a banking app is **authenticated** — but if they try `/admin/users`, they're **not authorized**, and the backend returns 403.

> **Say it like this:** "A user can be fully logged in — authenticated — and still get blocked from a specific action because they don't have the right role. That's a 403, not a 401. The login worked; the permission check failed."

---

## 6. Quick Revision Sheet

Read this section right before walking into the interview.

- **Security exists** because frontends can't be trusted — anyone can call your API directly.
- **Security layer sits before the controller** — bad requests never reach business logic.
- **Authentication = identity** ("who are you") → failure = `401`
- **Authorization = permission** ("what can you do") → failure = `403`
- Authorization **always comes after** authentication — you can't check permissions for an unknown user.
- **Frontend hides buttons; backend enforces rules.** Never trust the client.
- **Never store plain-text passwords** — always hash (e.g. BCrypt).
- **Never trust IDs from the URL** for "my own data" endpoints — use the authenticated principal instead.
- A user can be **authenticated and still not authorized** — these are two separate checks.
- Spring Security centralizes auth so controllers stay focused on business logic only.

---

## 7. Interview Q&A Bank (Natural, Spoken-Style Answers)

These are written the way you'd actually _say_ them out loud — clear and logical, not memorized textbook lines.
		   
**Q1. Why can't we rely on the frontend to protect sensitive APIs?** The frontend runs on the user's machine, so it can't be trusted. Even if a button is hidden, anyone can call the API directly with Postman or curl. Real protection has to happen on the backend.

**Q2. Why did Spring Security become popular?** Before it, developers wrote login, session handling, and permission checks manually every time — leading to duplicate code and security mistakes. Spring Security standardizes that into a tested framework.

**Q3. Why is the security layer placed before the controller?** So unauthorized requests never reach business logic. It's like airport security — they check your ticket before you reach the gate, not after.

**Q4. What happens if authentication is checked inside every controller instead of centrally?** The code gets duplicated everywhere, and it's easy for one developer to forget the check in one place, leaving that endpoint insecure.

**Q5. What is Application Security?** Protecting the application and its data so only the right people can access or change it — e.g. only an account owner sees their own balance.

**Q6. Is authentication enough on its own?** No — it only tells you who the user is. You still need authorization to know what that user is allowed to do.

**Q7. What is Authorization?** Checking whether an already-identified user has permission for a specific action — e.g. both Admin and Customer can log in, but only Admin can delete users.

**Q8. Why does authorization happen after authentication?** Because you can't decide someone's permissions until you know who they are.

**Q9. A user is logged in but gets a 403 — why?** Authentication succeeded (they're a valid user), but authorization failed — they don't have permission for that specific resource.

**Q10. Difference between 401 and 403?** 401 means you're not authenticated at all. 403 means you are authenticated, but you're not allowed to do this specific thing.

**Q11. Why isn't hiding the Delete button enough?** Because the frontend is client-side and can be bypassed entirely — the backend has to verify the role before performing the delete, every time.

**Q12. Why shouldn't passwords be stored as plain text?** If the database leaks, every password is instantly exposed. Hashing means even a leaked database doesn't reveal the real password.

**Q13. Why is `GET /users/{id}` risky?** If the ID is taken directly from the URL without checking ownership, a user could change it and view someone else's data. The backend should use the authenticated user's identity instead, not a client-supplied ID.

**Q14. Can an authenticated user still be an attacker?** Yes — being logged in doesn't mean someone won't try to access another user's data or an admin endpoint. That's exactly why authorization checks exist separately.

**Q15. If someone sends a fake or invalid JWT, what should happen?** Spring Security should reject it before the request ever reaches the controller, returning 401 Unauthorized.

**Q16. Why don't controllers check passwords directly?** Authentication is a cross-cutting concern shared by every endpoint, so it's handled centrally by Spring Security — controllers stay focused only on business logic.

**Q17. Why separate security from business logic?** It keeps the codebase clean and maintainable — business logic doesn't get cluttered with repeated identity and permission checks.

**Q18. A customer manually changes `/profile` to `/admin` in the URL — what should happen?** Spring Security checks their role before the request reaches the controller. Without the Admin role, it's blocked with a 403, and the controller code never even runs.

---

## 8. One Answer to Rule Them All

If an interviewer just says **"Explain how Spring Security works, in your own words"**, this is a safe, complete, natural answer:

> "I think of Spring Security as a gate in front of my application. Every request first passes through a security layer that checks two things: who the user is — that's authentication — and what they're allowed to do — that's authorization. Authentication always happens first, because you can't check permissions for someone you haven't identified. If identity fails, the response is 401. If identity is fine but permission is missing, it's 403. Only requests that pass both checks reach my controller, which keeps my business logic clean and means I never have to trust anything coming from the frontend."

---

## Navigation

- Previous Chapter: _(none — this is the starting chapter)_
- Next Chapter: [[01-Authentication-Flow]]