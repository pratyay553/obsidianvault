---
title: "Authentication Flow in Spring Security 6.x"

chapter: 1

tags:
  - spring-security
  - spring-boot
  - java21
  - authentication
  - backend
  - handbook

status: draft

java_version: "21"

spring_boot_version: "3.x"

spring_security_version: "6.x"

previous: "00-Introduction-To-Spring-Security.md"

next: "02-Authorization-And-Method-Security.md"

created: "2026-06-30"
---
## 1. Learning Objectives

By the end of this chapter you should be able to:

- Explain, in your own words, what "authentication" means inside the Spring Security filter chain (not just "checking a password").
- Trace a single login request from the browser down to the `SecurityContextHolder`, naming every internal class involved.
- Configure authentication in Spring Boot 3.x / Spring Security 6.x using `SecurityFilterChain` beans — no `WebSecurityConfigurerAdapter`.
- Identify the difference between an `Authentication` object that is "unauthenticated" versus "authenticated."
- Recognize and avoid the most common authentication misconfigurations seen in production codebases.
- Answer mid-to-senior level interview questions on this topic without hand-waving.

## 2. Why This Concept Exists

Every backend service that isn't fully public needs to answer one question before doing anything else: **"Who is making this request?"**

Without a standardized authentication layer, every team would reinvent password hashing, session handling, token parsing, and credential storage — usually badly. Spring Security exists to centralize that problem into a well-tested, pluggable pipeline so that application code never has to manually parse an `Authorization` header or compare password hashes by hand.

Authentication is the **first gate** in that pipeline. It exists to convert "an anonymous HTTP request" into "a request tied to a known identity," so that everything downstream (authorization, auditing, multi-tenancy, rate limiting per user, etc.) has something concrete to work with.

# 3. Simple Definition

Instead of:

> Authentication is the process of verifying who a request claims to be...

Write:

> Authentication simply means **verifying the identity of the user**.
> 
> The user claims,
> 
> > "I am Rahul."
> 
> Spring Security asks,
> 
> > "Can you prove it?"
> 
> If the proof (password, JWT token, API key, certificate, etc.) is valid, Spring Security creates an **Authentication object** and stores it in the **SecurityContextHolder**.
> 
> From that point onward, the application knows **who is making the request**.

---

# 4. Mental Model

Use an Indian example.

## ATM Card Example

Imagine you go to an ATM.

1. You insert your ATM card.
2. You enter your PIN.
3. The bank checks whether the PIN is correct.
4. If correct, the ATM lets you access your account.

Notice something important.

The ATM checks your PIN **only once**.

After verification, it already knows who you are.

Similarly,

- username/password
- JWT token
- API key

are only used to verify your identity.

Once verified, Spring Security stores your identity inside the **SecurityContextHolder**.

Every other part of the application simply checks that stored identity.

---

# 5. High-Level Flow

```
Client   │   ▼Spring Security Filter   │   ▼AuthenticationManager   │   ▼AuthenticationProvider   │   ▼UserDetailsService   │   ▼PasswordEncoder   │Password Correct?   │   ▼Authenticated User   │   ▼SecurityContextHolder   │   ▼Controller
```

---

# 6. Important Classes (Easy Version)

Instead of giving only definitions, explain them like an interviewer expects.

|Class|Easy Explanation|
|---|---|
|SecurityFilterChain|The list of security rules and filters for incoming requests.|
|UsernamePasswordAuthenticationFilter|Reads username and password from the login request.|
|AuthenticationManager|Doesn't authenticate itself. It decides **which AuthenticationProvider should do it.**|
|ProviderManager|Default implementation of AuthenticationManager.|
|AuthenticationProvider|Actually verifies the credentials.|
|UserDetailsService|Loads user information from the database.|
|PasswordEncoder|Compares the entered password with the encrypted password stored in the database.|
|Authentication|Represents the current user. Before login it is unauthenticated; after successful login it becomes authenticated.|
|SecurityContext|Holds the current Authentication object.|
|SecurityContextHolder|Gives access to the SecurityContext anywhere in the application.|

---

# 7. Authentication Flow (Interview Style)

## Step 1

The user sends username and password.

Example:

```
POST /loginusername = rahulpassword = 123456
```

---

## Step 2

`UsernamePasswordAuthenticationFilter` intercepts the request.

It creates an **unauthenticated Authentication object**.

```
UsernamePasswordAuthenticationTokenprincipal = rahulcredentials = 123456authenticated = false
```

Notice that Spring still doesn't know whether the password is correct.

---

## Step 3

The filter sends this object to the `AuthenticationManager`.

```
AuthenticationManager.authenticate(authentication)
```

Think of the AuthenticationManager as a manager in an office.

It usually doesn't verify credentials itself.

Instead, it forwards the work to the appropriate AuthenticationProvider.

---

## Step 4

`ProviderManager` looks at all registered AuthenticationProviders.

For username/password login, it usually selects

```
DaoAuthenticationProvider
```

because it knows how to authenticate username and password.

---

## Step 5

`DaoAuthenticationProvider` calls

```
UserDetailsService.loadUserByUsername()
```

This method loads the user from the database.

Example:

```
Username : rahulPassword :$2a$10$...Roles :ROLE_USER
```

---

## Step 6

Spring now compares passwords.

```
PasswordEncoder.matches(enteredPassword,storedEncryptedPassword)
```

If they match,

authentication succeeds.

Otherwise,

Spring throws `BadCredentialsException`.

---

## Step 7

A new Authentication object is created.

```
principal = UserDetailscredentials = nullauthorities = ROLE_USERauthenticated = true
```

Notice that

```
credentials = null
```

Spring removes the raw password for security reasons.

---

## Step 8

The Authentication object is stored inside

```
SecurityContextHolder
```

Now every part of the application can know

```
Who is the current user?
```

without asking for the password again.

---

## Step 9

The request reaches the controller.

Inside the controller,

```
SecurityContextHolder↓SecurityContext↓Authentication↓Current User
```

---

# Difference Between Authenticated and Unauthenticated

Before Login

```
Authenticationusername = rahulpassword = 123456authenticated = false
```

Spring still doesn't trust the user.

---

After Login

```
Authenticationprincipal = UserDetailsroles = ROLE_USERcredentials = nullauthenticated = true
```

Now Spring trusts the user.

---

# Common Mistakes (Simplified)

### ❌ Using WebSecurityConfigurerAdapter

It is removed in Spring Security 6.

Always use

```
SecurityFilterChain
```

---

### ❌ Saving plain passwords

Never store

```
123456
```

Store

```
$2a$10...
```

using BCrypt.

---

### ❌ Mixing Authentication and Authorization

Authentication answers

> Who are you?

Authorization answers

> What are you allowed to do?

These are different steps.

---

### ❌ Expecting Password After Login

```
authentication.getCredentials()
```

returns

```
null
```

after successful authentication.

This is intentional.

---

### ❌ Using SecurityContextHolder inside @Async

By default,

`SecurityContextHolder` is thread-local.

The new thread doesn't automatically receive the security context.

---

# Interview Questions (Indian Style Answers)

## 1. What is Authentication?

**Answer**

> Authentication is the process of verifying the identity of a user.
> 
> In Spring Security, after successful verification, an Authentication object is created and stored inside SecurityContextHolder so the application knows who is making the request.

---

## 2. Difference between Authentication and Authorization?

**Answer**

Authentication means

> "Who are you?"

Authorization means

> "What are you allowed to access?"

Authentication always happens before authorization.

---

## 3. What is AuthenticationManager?

**Answer**

AuthenticationManager is responsible for starting the authentication process.

It doesn't usually verify credentials itself.

Instead, it delegates the work to an AuthenticationProvider.

---

## 4. What is AuthenticationProvider?

**Answer**

AuthenticationProvider performs the actual authentication.

For username/password login, `DaoAuthenticationProvider` loads the user using `UserDetailsService` and verifies the password using `PasswordEncoder`.

---

## 5. Why do we need UserDetailsService?

**Answer**

`UserDetailsService` is responsible for loading user information from the database or another data source.

Spring Security calls its `loadUserByUsername()` method during authentication.

---

## 6. Why are credentials removed after login?

**Answer**

For security reasons.

Once the password has been verified, Spring Security no longer needs it, so it clears the credentials to reduce the risk of exposing sensitive information.

---

## 7. What is SecurityContextHolder?

**Answer**

`SecurityContextHolder` stores the current authenticated user's `Authentication` object for the duration of the request.

Any part of the application can access the logged-in user's details from it.