---
title: Request Lifecycle
chapter: 08
category: Spring Security Foundation
tags:
  - Spring Security
  - Request Lifecycle
  - Servlet
  - HTTP
  - Filter Chain
  - Backend Architecture
  - Authentication
  - Authorization
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
  - Servlet API
previous: "[[Chapter-07-SecurityFilterChain]]"
next: "[[Chapter-09-Authentication-Flow]]"
---


# Request Lifecycle


## Learning Objectives


After completing this chapter, you should understand:


- Complete HTTP request flow inside Spring Boot applications
- Where Spring Security enters the request pipeline
- How Servlet Container processes requests
- How Filters execute before Controllers
- Authentication lifecycle
- Authorization lifecycle
- How SecurityContext is created and used
- How to debug request failures in production


---


# 1. Why Understanding Request Lifecycle Matters


A backend engineer should know:


"How does a request travel from client to database and back?"


Without understanding lifecycle, debugging becomes guessing.


Example problem:


User reports:


"API returns 403"


Possible locations:

- API Gateway
- Tomcat
- Security Filter
- Authentication
- Authorization
- Controller
- Service


Request lifecycle helps locate the failure.


---


# 2. Simple Request Flow


Basic Spring Boot flow:

Client

|  
v

HTTP Request

|  
v

Servlet Container

|  
v

Filters

|  
v

DispatcherServlet

|  
v

Controller

|  
v

Service

|  
v

Repository

|  
v

Database

```
---# 3. Complete Spring Security Request FlowWith Spring Security:
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
Important:Spring Security executes before Controller.---# 4. Request Lifecycle Components## 1. ClientExamples:- Browser- Mobile application- Another microserviceSends:
```

GET /orders

Authorization:

Bearer JWT

```
---# 2. Servlet ContainerUsually:
```

Embedded Tomcat

```
Responsibilities:- Accept HTTP request- Create request/response objects- Manage threads- Execute filters- Invoke servlet---# 3. DelegatingFilterProxyBridge between:Servlet ContainerandSpring SecurityFlow:
```

Tomcat

|  
v

DelegatingFilterProxy

|  
v

Spring Security

```
---# 4. FilterChainProxyMain Spring Security entry point.Responsibilities:- Select SecurityFilterChain- Execute filters- Continue request processing---# 5. SecurityFilterChainDefines:- Which filters execute- Which URLs are protected- Authentication rules- Authorization rules---# 6. Security FiltersExamples:
```

Authentication Filter

JWT Filter

Authorization Filter

Exception Handling Filter

```
---# 7. DispatcherServletSpring MVC entry point.Responsibilities:- Find Controller- Bind request data- Execute controller method---# 5. Complete Internal Flow Diagram
```

```
             Client                |                v          HTTP Request                |                v          Tomcat Server                |                v    DelegatingFilterProxy                |                v         FilterChainProxy                |                v      SecurityFilterChain                |                v      Authentication Filters                |                v      Authorization Filters                |                v      DispatcherServlet                |                v          Controller                |                v          Service                |                v         Repository                |                v          Database
```

```
---# 6. Authentication Request LifecycleExample:User login:
```

POST /login

username

password

```
Flow:
```

Request

|  
v

UsernamePasswordAuthenticationFilter

|  
v

AuthenticationManager

|  
v

AuthenticationProvider

|  
v

UserDetailsService

|  
v

Database

|  
v

Authentication Object

|  
v

SecurityContextHolder

```
---# 7. JWT Authentication LifecycleModern REST APIs usually use JWT.Request:
```

GET /orders

Authorization:

Bearer eyJhbGciOi...

```
Flow:
```

Request

|  
v

BearerTokenAuthenticationFilter

|  
v

Extract Token

|  
v

Validate JWT

|  
v

AuthenticationManager

|  
v

SecurityContextHolder

|  
v

Controller

```
---# 8. Authorization LifecycleAuthentication answers:"Who are you?"Authorization answers:"What can you access?"Flow:
```

Authenticated User

```
    |    v
```

Granted Authorities

```
    |    v
```

AuthorizationFilter

```
    |    v
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
---# 9. SecurityContext LifecycleSecurityContext stores authentication information.Flow:
```

Request

|  
v

SecurityContextHolderFilter

|  
v

SecurityContext

|  
v

Authentication Object

|  
v

Controller

```
Example:```javaAuthentication authentication =    SecurityContextHolder        .getContext()        .getAuthentication();
```

---

# 10. Successful Request Flow

Example:

```
GET /profileAuthorization:Bearer Token
```

Flow:

```
Client | vTomcat | vSecurity Filters | vJWT Validation | vAuthentication Success | vAuthorization Success | vController | vResponse
```

---

# 11. Failed Authentication Flow

Example:

Invalid JWT.

Flow:

```
Request | vBearerTokenAuthenticationFilter | vJWT Validation | vAuthentication Failed | vAuthenticationEntryPoint | v401 Unauthorized
```

---

# 12. Failed Authorization Flow

Example:

User is logged in but lacks permission.

Flow:

```
Request | vAuthentication Success | vAuthorization Check | vAccess Denied | v403 Forbidden
```

---

# 13. Thread Model

Servlet containers use threads.

Example:

```
Request 1   |   vThread 1----------------Request 2   |   vThread 2
```

Each request gets a thread.

SecurityContext is associated with the current request thread.

---

# 14. Production Microservice Example

Architecture:

```
Mobile App | vAPI Gateway | vOrder Service | vDatabase
```

Request:

```
GET /ordersAuthorization:Bearer JWT
```

Internal flow:

```
Tomcat | vSecurity Filters | vJWT Validation | vAuthorization | vOrderController | vOrderService | vDatabase
```

---

# 15. Debugging Request Lifecycle

## Problem 1

Controller is not reached.

Check:

```
Client | vFilters | vDispatcherServlet | XController
```

Possible causes:

- Authentication failure
- Authorization failure
- Filter exception

---

## Problem 2

Controller executes but user is null.

Check:

```
SecurityContextHolderAuthentication Object
```

Possible causes:

- Security context not populated
- Authentication filter missing

---

## Problem 3

API returns 403.

Check:

```
Authentication        |Authorities        |Authorization Rules
```

---

# 16. Common Mistakes

## Mistake 1: Checking Security Inside Controller

Bad:

```
Controller | vif(user.isAdmin())
```

Better:

```
SecurityFilterChain | vAuthorization Rules
```

---

## Mistake 2: Assuming Authentication Means Authorization

Example:

User authenticated:

YES

Permission:

NO

Result:

403 Forbidden

---

## Mistake 3: Ignoring Filter Order

Example:

Authorization requires authentication.

Correct order:

```
Authentication Filter        |Authorization Filter
```

---

# 17. Interview Questions

## Q1. Explain Spring Security request lifecycle.

Answer:

Request enters Servlet Container, passes through Spring Security filters, authentication happens, authorization is checked, then request reaches DispatcherServlet and Controller.

---

## Q2. Where does Spring Security execute?

Answer:

Inside Servlet Filter chain before DispatcherServlet.

---

## Q3. Difference between authentication and authorization?

Authentication:

Who are you?

Authorization:

What can you access?

---

## Q4. What happens when JWT is invalid?

Answer:

Authentication fails and Spring Security returns 401 Unauthorized.

---

# 18. Scenario Based Questions

## Scenario 1

API returns 401.

Debug:

```
Token |Authentication Filter |AuthenticationManager
```

---

## Scenario 2

API returns 403.

Debug:

```
User Authorities |Authorization Rules
```

---

## Scenario 3

Controller is not executed.

Debug:

```
Security Filters |DispatcherServlet
```

---

# 19. Revision Notes

Complete request lifecycle:

```
Client | vTomcat | vDelegatingFilterProxy | vFilterChainProxy | vSecurityFilterChain | vSecurity Filters | vDispatcherServlet | vController | vService | vDatabase
```

Remember:

Authentication happens before Controller.

Authorization happens before Controller.

Security is a pipeline, not a controller feature.

---

# Previous / Next Chapter

Previous:

[[Chapter-07-SecurityFilterChain]]

Next:

[[Chapter-09-Authentication-Flow]]