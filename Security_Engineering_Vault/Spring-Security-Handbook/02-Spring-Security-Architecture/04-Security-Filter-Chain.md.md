---
title: Security Filter Chain
chapter: 4
category: Spring Security Foundation
tags:
  - Spring Security
  - Servlet
  - HTTP
  - Backend Architecture
  - Authentication
  - Authorization
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
  - Servlet API
previous: "[[Chapter-03-Servlet-Architecture]]"
next: "[[Chapter-05-Authentication-Architecture]]"
---

# Security Filter Chain

## Learning Objectives

After completing this chapter, you should understand:

- What a Servlet Filter is
- Why Spring Security is implemented using Filters
- How SecurityFilterChain works internally
- How HTTP requests travel through Spring Security
- Important Spring Security filter classes
- Authentication and authorization flow inside filters
- How to debug security issues in production
- How to design security configuration like a senior backend engineer

---

# 1. Why Security Filter Chain Exists

In enterprise applications, security cannot start inside controllers.

Bad design:
This does not scale.

Spring Security solves this by moving security before application code.

---

# 2. Simple Definition

## Security Filter Chain

Security Filter Chain is a sequence of Servlet Filters that process every incoming HTTP request before it reaches the application.

Simple flow:

```
Client   |   vSecurity Filters   |   vController
```

Spring Security acts like a security gateway inside the application.

---

# 3. Mental Model

Think of a company building entrance.

```
Employee arrives        |        vSecurity Gate        |        +----------------+        |                |        v                vVerify Identity     Check Permission        |        vEnter Building
```

Spring Security does the same:

```
Request | vAuthenticate | vAuthorize | vAllow Request
```

---

# 4. Big Picture Architecture

Complete request flow:

```
                 Client                   |                   |             HTTP Request                   |                   v        +-------------------+        | Servlet Container |        | Tomcat            |        +-------------------+                   |                   v        +-------------------+        | Filter Chain      |        +-------------------+                   |                   v        +-------------------+        | Spring Security   |        | Filters           |        +-------------------+                   |                   v        +-------------------+        | DispatcherServlet |        +-------------------+                   |                   v             Controller
```

---

# 5. What is a Servlet Filter?

A Filter is a component that intercepts requests before they reach a Servlet.

Example:

```
Request | vLogging Filter | vAuthentication Filter | vController
```

A Filter can:

- Modify request
- Validate request
- Reject request
- Add headers
- Log information

---

# 6. Why Spring Security Uses Filters

Security needs to happen before business logic.

Example:

Request:

```
DELETE /users/100Authorization:Bearer token
```

Flow:

```
Request | vJWT Authentication Filter | vValidate Token | vCheck Role | vUserController
```

If authentication fails:

```
Request | vSecurity Filter | v401 Unauthorized
```

Controller never executes.

---

# 7. SecurityFilterChain

Main interface:

```
org.springframework.security.web.SecurityFilterChain
```

It represents the collection of security filters applied to requests.

Example:

```
@BeanSecurityFilterChain securityFilterChain(        HttpSecurity http) throws Exception {    http        .authorizeHttpRequests(auth -> auth            .requestMatchers("/public/**")            .permitAll()            .anyRequest()            .authenticated()        );    return http.build();}
```

---

# 8. Internal Architecture

```
HTTP Request      |      vDelegatingFilterProxy      |      vFilterChainProxy      |      vSecurityFilterChain      |      vSecurity Filters      |      vDispatcherServlet
```

---

# 9. Important Internal Classes

# DelegatingFilterProxy

Package:

```
org.springframework.web.filter.DelegatingFilterProxy
```

Purpose:

Connects Servlet Container filters with Spring-managed beans.

Flow:

```
Tomcat | vDelegatingFilterProxy | vSpring Context | vSecurity Filter Chain
```

---

# FilterChainProxy

Package:

```
org.springframework.security.web.FilterChainProxy
```

This is the central Spring Security filter.

Responsibilities:

- Select correct SecurityFilterChain
- Execute filters
- Manage security processing

Flow:

```
Request | vFilterChainProxy | +---- SecurityFilterChain 1 | +---- SecurityFilterChain 2
```

---

# 10. Default Security Filters

Spring Security 6 includes many filters.

Common ones:

---

## SecurityContextHolderFilter

Purpose:

Loads SecurityContext.

Flow:

```
Request | vSecurityContext | vSecurityContextHolder
```

---

## UsernamePasswordAuthenticationFilter

Handles username/password authentication.

Example:

```
POST /loginusername=johnpassword=password        |        vAuthenticationManager
```

---

## BasicAuthenticationFilter

Handles HTTP Basic authentication.

Example:

```
Authorization:Basic base64(username:password)
```

---

## BearerTokenAuthenticationFilter

Used for JWT/OAuth2 resource servers.

Example:

```
Authorization:Bearer eyJhbGciOi...
```

---

## AuthorizationFilter

Checks permissions.

Example:

```
UserROLE_USERRequest:DELETE /adminRequired:ROLE_ADMINResult:403 Forbidden
```

---

# 11. Complete Request Lifecycle

Example:

```
Client | |GET /api/orders | vTomcat | vDelegatingFilterProxy | vFilterChainProxy | vSecurity Filters | | +-------------------+ |                   | v                   vAuthentication    Authorization | vSecurityContextHolder | vDispatcherServlet | vController | vResponse
```

---

# 12. Authentication Flow

Example:

Login request:

```
POST /loginusernamepassword
```

Flow:

```
UsernamePasswordAuthenticationFilter          |          vAuthenticationManager          |          vAuthenticationProvider          |          vUserDetailsService          |          vDatabase          |          vAuthentication Object          |          vSecurityContext
```

---

# 13. Authorization Flow

Example:

```
GET /admin/report
```

Flow:

```
Request | vIs User Authenticated? | vCheck Authorities | vROLE_ADMIN?      |      |   YES       NO    |         | Continue   403
```

---

# 14. Multiple Security Filter Chains

Enterprise applications may have multiple chains.

Example:

```
/api/**      |JWT Authentication-------------------/admin/**      |Session Authentication
```

Configuration:

```
@BeanSecurityFilterChain apiSecurity(        HttpSecurity http){    return http.build();}
```

Spring chooses the matching chain.

---

# 15. Production Example

Microservice:

```
Mobile App | vAPI Gateway | vOrder Service
```

Request:

```
GET /ordersAuthorization:Bearer JWT
```

Inside service:

```
Tomcat | vBearerTokenAuthenticationFilter | vJWT Validation | vAuthorizationFilter | vOrderController
```

---

# 16. Debugging Security Problems

## Problem: 401 Unauthorized

Check:

```
Authentication | vToken | vSecurityContext
```

Possible causes:

- Missing token
- Expired token
- Invalid credentials

---

## Problem: 403 Forbidden

Authentication succeeded.

Authorization failed.

Check:

```
User Authorities        |Required Role
```

---

## Problem: Controller not reached

Check:

```
Security Filters       |Filter Exceptions       |Request Matchers
```

---

# 17. Common Mistakes

## Mistake 1: Adding Security Logic in Controllers

Wrong:

```
if(user.isAdmin())
```

Better:

```
Security Configuration        |Authorization Rules
```

---

## Mistake 2: Wrong Filter Order

Filters execute in order.

Example:

```
JWT Filter      beforeAuthorization Filter
```

Wrong order causes failures.

---

## Mistake 3: Disabling Security Globally

Avoid:

```
.anyRequest().permitAll()
```

in production.

---

## Mistake 4: Forgetting Stateless APIs

REST APIs usually use:

```
JWTOAuth2API Keys
```

not server sessions.

---

# 18. Interview Questions

## Q1. Why does Spring Security use Filters?

Answer:

Because security must happen before controllers execute. Filters provide centralized request interception.

---

## Q2. What is SecurityFilterChain?

Answer:

A collection of security filters that process incoming HTTP requests.

---

## Q3. Difference between Filter and Controller?

Filter:

```
Infrastructure levelRuns before application logic
```

Controller:

```
Business request handling
```

---

## Q4. What is FilterChainProxy?

Answer:

The central Spring Security filter that delegates requests to SecurityFilterChain implementations.

---

# 19. Scenario Based Questions

## Scenario 1

JWT is valid but API returns 403.

Possible reason:

Authentication succeeded but authority is missing.

---

## Scenario 2

Public API requires login.

Check:

```
requestMatchers()permitAll()
```

---

## Scenario 3

Custom JWT filter is not executing.

Check:

- Filter registration
- Filter order
- Security configuration

---

# 20. Revision Notes

Remember:

```
Servlet Container        |Filters        |Spring Security Filter Chain        |DispatcherServlet        |Controller
```

Security happens before business logic.

Important classes:

```
DelegatingFilterProxyFilterChainProxySecurityFilterChainAuthenticationManagerAuthorizationFilterSecurityContextHolder
```

---

# Previous / Next Chapter

Previous:

[[Chapter-03-Servlet-Architecture]]

Next:

[[Chapter-05-Authentication-Architecture]]