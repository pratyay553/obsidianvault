---
title: FilterChainProxy
chapter: 06
category: Spring Security Foundation
tags:
  - Spring Security
  - FilterChainProxy
  - SecurityFilterChain
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
previous: "[[Chapter-05-DelegatingFilterProxy]]"
next: "[[Chapter-07-Security-Filters]]"
---

# FilterChainProxy

## Learning Objectives

After completing this chapter, you should understand:

- What FilterChainProxy does internally
- Why Spring Security needs FilterChainProxy
- Difference between DelegatingFilterProxy and FilterChainProxy
- How SecurityFilterChain selection works
- How multiple security chains work in enterprise applications
- How HTTP requests move through Spring Security internally
- How to debug authentication and authorization issues
- How senior backend engineers reason about security request flow

---

# 1. Why FilterChainProxy Exists


In Servlet architecture, the container understands Filters.

Example:
HTTP Request


  |  
  v
  

Servlet Filter

  |  
  v

Servlet

Spring Security has many security responsibilities:- Authentication- Authorization- CSRF protection- Session management- Exception handling- Security context managementPutting all security logic into one Filter would become complicated.Problem:
```

One Giant Security Filter

+--------------------------+  
| Authentication |  
| Authorization |  
| CSRF |  
| Session Management |  
| Exception Handling |  
| JWT Validation |  
+--------------------------+

```
Solution:Create multiple security filters and manage them centrally.That manager is:
```

FilterChainProxy

```
---# 2. Simple Definition## FilterChainProxyFilterChainProxy is the central Spring Security Servlet Filter that delegates incoming requests to the appropriate SecurityFilterChain.Simple view:
```

HTTP Request

```
  |  v
```

FilterChainProxy

```
  |  v
```

Security Filters

```
  |  v
```

Application

```
---# 3. Mental ModelThink of an airport security system.
```

Passenger

|  
v

Security Entrance

|  
+----------------+  
| |  
v v

Domestic Gate International Gate

```
FilterChainProxy decides:"Which security pipeline should process this request?"---# 4. Complete Spring Security Architecture
```

Client

|  
v

Servlet Container (Tomcat)

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
---# 5. FilterChainProxy Internal RoleFilterChainProxy performs three main jobs:## 1. Receive RequestExample:
```

GET /api/orders

```
---## 2. Select Matching SecurityFilterChainExample:
```

Request:

/api/orders

Matches:

API Security Chain

```
---## 3. Execute Security FiltersExample:
```

JWT Filter

```
  |
```

Authorization Filter

```
  |
```

Exception Handling Filter

```
---# 6. Internal ClassClass:```javaorg.springframework.security.web.FilterChainProxy``` id="n9j5r8"Responsibilities:- Store SecurityFilterChain objects- Match requests- Execute filters- Continue request processing---# 7. SecurityFilterChain RelationshipFilterChainProxy contains one or more SecurityFilterChain objects.Architecture:
```

FilterChainProxy

```
    |    |    +----------------+    |    v
```

SecurityFilterChain 1

```
    |    |    +----------------+    |    v
```

SecurityFilterChain 2

```
---# 8. Single Security Filter ChainMost applications have one chain.Example:
```

All Requests

```
  |  v
```

SecurityFilterChain

```
  |  v
```

Authentication

```
  |
```

Authorization

```
---# 9. Multiple Security Filter ChainsEnterprise applications often need different security rules.Example:
```

/api/**

```
  |  v
```

JWT Authentication

---

/admin/**

```
  |  v
```

Session Authentication

```
Example configuration:```java@BeanSecurityFilterChain apiSecurity(        HttpSecurity http) throws Exception {    http        .securityMatcher("/api/**")        .authorizeHttpRequests(auth ->            auth.anyRequest()                .authenticated()        );    return http.build();}``` id="0slvpg"---# 10. Request Matching ProcessExample request:
```

GET /api/orders

```
Flow:
```

Request

|  
v

FilterChainProxy

|  
|  
+---------------------+  
|  
v

Does Chain 1 match?

|  
|  
No

|  
v

Does Chain 2 match?

|  
|  
Yes

|  
v

Execute Chain 2

```
---# 11. Detailed Request Lifecycle
```

Client

|  
v

HTTP Request

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
---# 12. Security Filters Inside FilterChainProxyA typical chain:
```

SecurityContextHolderFilter

```
    |
```

LogoutFilter

```
    |
```

UsernamePasswordAuthenticationFilter

```
    |
```

BearerTokenAuthenticationFilter

```
    |
```

AuthorizationFilter

```
    |
```

ExceptionTranslationFilter

```
Order matters.---# 13. Authentication FlowExample JWT request:
```

GET /orders

Authorization:

Bearer token

```
Flow:
```

Request

|  
v

BearerTokenAuthenticationFilter

|  
v

AuthenticationManager

|  
v

AuthenticationProvider

|  
v

SecurityContextHolder

```
---# 14. Authorization FlowAfter authentication:
```

User Identity

```
   |
```

Authorities

```
   |
```

AuthorizationFilter

```
   |
```

Access Decision

```
Example:
```

User:

ROLE_USER

Request:

DELETE /admin/user

Required:

ROLE_ADMIN

Result:

403 Forbidden

```
---# 15. Difference Between DelegatingFilterProxy and FilterChainProxy## DelegatingFilterProxyPurpose:Bridge between Servlet container and Spring.
```

Tomcat

|

DelegatingFilterProxy

|

Spring Bean

```
---## FilterChainProxyPurpose:Execute Spring Security filters.
```

FilterChainProxy

|

Security Filters

```
---# 16. Production ExampleMicroservice:
```

Mobile App

|  
v

API Gateway

|  
v

Order Service

```
Request:
```

GET /orders

Authorization:

Bearer JWT

```
Processing:
```

Tomcat

|  
v

DelegatingFilterProxy

|  
v

FilterChainProxy

|  
v

BearerTokenAuthenticationFilter

|  
v

AuthorizationFilter

|  
v

OrderController

```
---# 17. Debugging FilterChainProxy Problems## Problem 1: Security Filters Not RunningCheck:
```

Request

|  
v

DelegatingFilterProxy

|  
v

FilterChainProxy

```
Possible causes:- SecurityFilterChain bean missing- Incorrect matcher- Security disabled---# Problem 2: Wrong Security Chain SelectedExample:
```

/api/orders

matches

/admin chain

```
Check:- securityMatcher()- Request patterns- Chain ordering---# Problem 3: Authentication Works But Authorization FailsCheck:
```

Authentication

```
    |
```

Authorities

```
    |
```

Authorization Rules

```
---# 18. Common Mistakes## Mistake 1: Confusing FilterChainProxy With SecurityFilterChainWrong:
```

FilterChainProxy = one security rule

```
Correct:
```

FilterChainProxy

contains

multiple SecurityFilterChain objects

```
---## Mistake 2: Ignoring Filter OrderingExample:Wrong:
```

Authorization Filter

```
    |
```

Authentication Filter

```
Correct:
```

Authentication Filter

```
    |
```

Authorization Filter

```
---## Mistake 3: Creating Custom Filters Without Understanding OrderCustom filters must be placed correctly.Example:```javahttp.addFilterBefore(    jwtFilter,    UsernamePasswordAuthenticationFilter.class);
```

---

# 19. Interview Questions

## Q1. What is FilterChainProxy?

Answer:

FilterChainProxy is the central Spring Security filter responsible for selecting and executing the appropriate SecurityFilterChain.

---

## Q2. Difference between FilterChainProxy and DelegatingFilterProxy?

Answer:

DelegatingFilterProxy connects Servlet container with Spring.

FilterChainProxy executes Spring Security filters.

---

## Q3. Can an application have multiple SecurityFilterChain beans?

Answer:

Yes. Enterprise applications commonly use multiple chains for different URL patterns.

---

## Q4. Why is filter order important?

Answer:

Because later filters depend on information created by previous filters, such as authentication details.

---

# 20. Scenario Based Questions

## Scenario 1

JWT token is valid but controller is never reached.

Debug:

```
BearerTokenAuthenticationFilter        |AuthorizationFilter        |Controller``` id="6j1p8s"---## Scenario 2Admin endpoint uses user authentication rules.Check:
```

SecurityFilterChain ordering

Request matcher priority

```
---## Scenario 3Custom JWT filter does not execute.Check:- Filter registration- addFilterBefore()- Matching SecurityFilterChain---# 21. Revision NotesRemember:
```

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

```
Key idea:> FilterChainProxy is the brain that controls Spring Security filter execution.Important classes:
```

DelegatingFilterProxy

FilterChainProxy

SecurityFilterChain

AuthenticationManager

AuthorizationFilter

SecurityContextHolder

```
---# Previous / Next ChapterPrevious:[[Chapter-05-DelegatingFilterProxy]]Next:[[Chapter-07-Security-Filters]]
```