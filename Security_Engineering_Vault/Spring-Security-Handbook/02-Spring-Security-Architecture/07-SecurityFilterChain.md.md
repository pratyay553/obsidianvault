---
title: SecurityFilterChain
chapter: 07
category: Spring Security Foundation
tags:
  - Spring Security
  - SecurityFilterChain
  - FilterChainProxy
  - Servlet
  - HTTP
  - Backend Architecture
  - Authentication
  - Authorization
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
previous: "[[Chapter-06-FilterChainProxy]]"
next: "[[Chapter-08-Security-Filters]]"
---


# SecurityFilterChain


## Learning Objectives


After completing this chapter, you should understand:


- What SecurityFilterChain represents
- How SecurityFilterChain works internally
- How Spring Security builds security rules
- Relationship between FilterChainProxy and SecurityFilterChain
- How request matching works
- How multiple SecurityFilterChain beans work
- How to configure security using Spring Security 6 style
- How senior backend engineers design security boundaries


---


# 1. Why SecurityFilterChain Exists


Spring Security needs a way to define:

- Which URLs need authentication
- Which URLs are public
- Which filters should run
- What security rules should apply


Example:


A banking application:

/login

Public

---

/accounts/**

Authenticated User

---

/admin/**

Admin Only

```
Different endpoints need different security rules.Spring Security represents these rules using:SecurityFilterChain---# 2. Simple Definition## SecurityFilterChainSecurityFilterChain is an ordered collection of Spring Security filters that process HTTP requests according to configured security rules.Simple view:
```

HTTP Request

```
  |  v
```

SecurityFilterChain

```
  |  v
```

Security Filters

```
  |  v
```

Application

```
---# 3. Mental ModelThink of SecurityFilterChain as a security pipeline.Example:Request:
```

GET /admin/users

```
Pipeline:
```

Request

|  
v

Authentication Check

|  
v

Authorization Check

|  
v

CSRF Check

|  
v

Application

```
Each SecurityFilterChain defines the pipeline.---# 4. Complete ArchitectureRequest flow:
```

Client

|  
v

Tomcat

|  
v

DelegatingFilterProxy

|  
v

FilterChainProxy

|  
v

SecurityFilterChain

|  
v

Security Filters

|  
v

DispatcherServlet

|  
v

Controller

```
---# 5. Internal ClassInterface:```javaorg.springframework.security.web.SecurityFilterChain
```

Purpose:

Defines:

- Filters involved
- Request matching rules
- Security behavior

---

# 6. Relationship With FilterChainProxy

Important:

FilterChainProxy manages SecurityFilterChain objects.

Architecture:

```
FilterChainProxy        |        |        +----------------+        |        vSecurityFilterChain 1        |        |        +----------------+        |        vSecurityFilterChain 2
```

Flow:

```
Request | vFilterChainProxy | vFind Matching SecurityFilterChain | vExecute Filters
```

---

# 7. Default Security Configuration

Spring Boot automatically creates a default security chain.

Example:

```
Application Startup        |        vSpring Security Auto Configuration        |        vSecurityFilterChain Bean        |        vFilterChainProxy
```

---

# 8. Creating SecurityFilterChain

Modern Spring Security 6 configuration:

```
@BeanSecurityFilterChain securityFilterChain(        HttpSecurity http) throws Exception {    http        .authorizeHttpRequests(auth ->            auth            .requestMatchers("/public/**")            .permitAll()            .anyRequest()            .authenticated()        );    return http.build();}
```

Important:

Spring Security 6 uses:

```
SecurityFilterChain Bean
```

Not:

```
WebSecurityConfigurerAdapter
```

---

# 9. Request Matching

SecurityFilterChain decides:

"Does this chain handle this request?"

Example:

Request:

```
GET /api/orders
```

Rule:

```
/api/**
```

Match:

```
Yes
```

Security filters execute.

---

# 10. Multiple SecurityFilterChain

Enterprise applications commonly have multiple chains.

Example:

Public APIs:

```
/public/**No authentication
```

Mobile APIs:

```
/api/**JWT Authentication
```

Admin:

```
/admin/**Admin Authentication
```

Architecture:

```
FilterChainProxy        |        +---- Public SecurityFilterChain        |        +---- API SecurityFilterChain        |        +---- Admin SecurityFilterChain
```

---

# 11. Multiple Chain Example

Example:

```
@BeanSecurityFilterChain apiSecurity(        HttpSecurity http) throws Exception {    http        .securityMatcher("/api/**")        .authorizeHttpRequests(auth ->            auth.anyRequest()                .authenticated()        );    return http.build();}
```

Another chain:

```
@BeanSecurityFilterChain adminSecurity(        HttpSecurity http) throws Exception {    http        .securityMatcher("/admin/**")        .authorizeHttpRequests(auth ->            auth.anyRequest()                .hasRole("ADMIN")        );    return http.build();}
```

---

# 12. SecurityFilterChain Selection Process

Example:

Request:

```
GET /admin/users
```

Flow:

```
Request | vFilterChainProxy | vCheck SecurityFilterChain 1 | vMatch?No | vCheck SecurityFilterChain 2 | vMatch?Yes | vExecute Chain
```

---

# 13. Filter Order Inside SecurityFilterChain

A SecurityFilterChain contains filters in order.

Example:

```
SecurityContextHolderFilter        |LogoutFilter        |UsernamePasswordAuthenticationFilter        |BearerTokenAuthenticationFilter        |AuthorizationFilter
```

Order matters.

---

# 14. Authentication Flow

Example:

JWT Request:

```
GET /ordersAuthorization:Bearer TOKEN
```

Flow:

```
Request | vBearerTokenAuthenticationFilter | vAuthenticationManager | vAuthenticationProvider | vSecurityContextHolder
```

---

# 15. Authorization Flow

After authentication:

```
User | vAuthorities | vAuthorizationFilter | vAccess Decision
```

Example:

```
User:ROLE_USERRequest:DELETE /admin/userRequired:ROLE_ADMINResult:403 Forbidden
```

---

# 16. Production Example

Microservice:

```
Mobile App | vAPI Gateway | vOrder Service
```

Request:

```
GET /ordersAuthorization:Bearer JWT
```

Processing:

```
Tomcat | vDelegatingFilterProxy | vFilterChainProxy | vSecurityFilterChain | vJWT Authentication | vAuthorization | vOrderController
```

---

# 17. Debugging SecurityFilterChain Problems

## Problem 1: Request Not Matching Chain

Example:

```
Request:/api/ordersConfigured:/admin/**
```

Result:

Security rules do not apply.

Check:

- securityMatcher()
- request patterns

---

## Problem 2: Wrong Chain Executed

Example:

```
/api/adminmatchesgeneral API chaininstead ofadmin chain
```

Check:

- Chain order
- Matcher priority

---

## Problem 3: Authentication Works But Authorization Fails

Check:

```
Authentication        |Authorities        |Authorization Rules
```

---

# 18. Common Mistakes

## Mistake 1: Creating Too Many Chains

Bad:

```
100 SecurityFilterChain beans
```

Problem:

Hard to maintain.

Better:

Group related endpoints.

---

## Mistake 2: Wrong Matcher Order

Example:

```
/**before/admin/**
```

The general rule catches everything.

---

## Mistake 3: Mixing Business Logic With Security

Bad:

```
Controller | vCheck Role Manually
```

Better:

```
SecurityFilterChain | vAuthorization Rules
```

---

# 19. Interview Questions

## Q1. What is SecurityFilterChain?

Answer:

SecurityFilterChain is a collection of security filters that defines how requests are secured.

---

## Q2. Relationship between FilterChainProxy and SecurityFilterChain?

Answer:

FilterChainProxy manages SecurityFilterChain objects and selects the correct chain for each request.

---

## Q3. Can we create multiple SecurityFilterChain beans?

Answer:

Yes.

Different chains can secure different parts of an application.

---

## Q4. What replaced WebSecurityConfigurerAdapter?

Answer:

Spring Security 6 uses SecurityFilterChain beans.

---

# 20. Scenario Based Questions

## Scenario 1

JWT authentication works for /api but not /admin.

Check:

```
SecurityFilterChain matcher        |Authorization rules
```

---

## Scenario 2

Public endpoint asks for login.

Check:

```
requestMatchers()permitAll()
```

---

## Scenario 3

Admin user gets 403.

Check:

```
Granted AuthoritiesROLE namingAuthorization rules
```

---

# 21. Revision Notes

Remember:

```
Request | vFilterChainProxy | vSelect SecurityFilterChain | vExecute Security Filters | vController
```

Key idea:

SecurityFilterChain defines how a group of requests should be protected.

Important classes:

```
SecurityFilterChainFilterChainProxyHttpSecurityAuthenticationManagerAuthorizationFilterSecurityContextHolder
```

---

# Previous / Next Chapter

Previous:

[[Chapter-06-FilterChainProxy]]

Next:

[[Chapter-08-Security-Filters]]