---
title: DelegatingFilterProxy
chapter: 05
category: Spring Security Foundation
tags:
  - Spring Security
  - DelegatingFilterProxy
  - Servlet
  - Filter
  - HTTP
  - Backend Architecture
  - Spring Boot
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
  - Servlet API
previous: "[[Chapter-04-Security-Filter-Chain]]"
next: "[[Chapter-06-SecurityContext-Architecture]]"
---

# DelegatingFilterProxy

## Learning Objectives

After completing this chapter, you should understand:

- Why DelegatingFilterProxy exists
- How Servlet Filters and Spring Beans communicate
- How Spring Security connects with the Servlet container
- Internal request flow through DelegatingFilterProxy
- Difference between Servlet-managed Filters and Spring-managed Filters
- How FilterChainProxy works behind DelegatingFilterProxy
- How to debug security startup and request flow issues
- How to explain DelegatingFilterProxy in backend interviews

---

# 1. Why DelegatingFilterProxy Exists


Spring Security is built using Servlet Filters.

But there is an architectural problem.

Servlet containers like Tomcat create and manage Filters.

Spring manages Beans.


Two different worlds:
Servlet Container

```
    |    
    |    
    v
```

Servlet Filters

---

Spring Application Context

```
     v
```

Spring Beans

```
Problem:How does a Servlet Filter call a Spring Bean?Solution:
```

DelegatingFilterProxy

```
---# 2. Simple Definition## DelegatingFilterProxyDelegatingFilterProxy is a Servlet Filter provided by Spring that delegates incoming requests to a Spring-managed Filter bean.Simple meaning:
```

Servlet Container Filter

```
    |    v
```

Delegates work to

```
    |    v
```

Spring Bean

```
---# 3. Mental ModelThink of a receptionist in a company.The receptionist does not perform every task.The receptionist forwards requests to the correct department.
```

Visitor

|  
v

Reception Desk

|  
+------------+  
| |  
v v

HR Department

Security Department

```
DelegatingFilterProxy works the same way.
```

HTTP Request

|  
v

DelegatingFilterProxy

|  
v

Spring Security Filter Chain

```
---# 4. Big Picture ArchitectureComplete architecture:
```

```
             Client               |               |          HTTP Request               |               v    +----------------------+    | Servlet Container    |    | Tomcat               |    +----------------------+               |               v    +----------------------+    | DelegatingFilterProxy|    +----------------------+               |               v    +----------------------+    | Spring Application   |    | Context              |    +----------------------+               |               v    +----------------------+    | FilterChainProxy     |    +----------------------+               |               v    +----------------------+    | Security Filters     |    +----------------------+               |               v    DispatcherServlet
```

```
---# 5. Servlet Filter vs Spring Bean Filter## Servlet FilterManaged by:
```

Servlet Container

```
Example:
```

Tomcat

```
Lifecycle controlled by:
```

Servlet specification

```
---## Spring Security FilterManaged by:
```

Spring Application Context

```
Lifecycle controlled by:
```

Spring Framework

```
---# 6. The Problem Without DelegatingFilterProxyImagine:
```

Tomcat

|  
v

Security Filter

|  
?

AuthenticationManager

UserDetailsService

Application Services

```
The filter needs Spring dependencies.But Tomcat does not know Spring beans.Problem:
```

Servlet world

```
    X
```

Spring world

```
DelegatingFilterProxy bridges them.---# 7. Internal Request FlowDetailed flow:
```

HTTP Request

```
   |   v
```

Tomcat

```
   |   v
```

DelegatingFilterProxy

```
   |   v
```

ApplicationContext

```
   |   v
```

FilterChainProxy

```
   |   v
```

SecurityFilterChain

```
   |   v
```

Security Filters

```
   |   v
```

DispatcherServlet

```
---# 8. Important Internal Classes## DelegatingFilterProxyPackage:```javaorg.springframework.web.filter.DelegatingFilterProxy
```

Responsibilities:

- Receives request from Servlet container
- Finds Spring bean
- Delegates execution

---

## FilterChainProxy

Package:

```
org.springframework.security.web.FilterChainProxy
```

Responsibilities:

- Holds Spring Security filters
- Executes security logic
- Selects matching SecurityFilterChain

Flow:

```
DelegatingFilterProxy        |        vFilterChainProxy        |        vSecurity Filters
```

---

# 9. How Spring Boot Registers DelegatingFilterProxy

In modern Spring Boot applications:

You usually do not manually create it.

Spring Boot auto-configures:

```
Application Startup        |        vSpring Security Auto Configuration        |        vSecurity Filter Chain Bean        |        vDelegatingFilterProxy Registration
```

---

# 10. Security Filter Bean Flow

Example configuration:

```
@BeanSecurityFilterChain securityFilterChain(        HttpSecurity http) throws Exception {    http        .authorizeHttpRequests(auth ->            auth            .anyRequest()            .authenticated()        );    return http.build();}
```

Behind the scenes:

```
SecurityFilterChain Bean        |        vFilterChainProxy        |        vDelegatingFilterProxy        |        vTomcat
```

---

# 11. Application Startup Lifecycle

When Spring Boot starts:

```
Spring Boot | vCreate Application Context | vCreate Security Beans | vCreate FilterChainProxy | vRegister DelegatingFilterProxy | vStart Tomcat
```

---

# 12. Request Processing Example

Request:

```
GET /api/ordersAuthorization:Bearer JWT
```

Flow:

```
Client | vTomcat | vDelegatingFilterProxy | vFilterChainProxy | vBearerTokenAuthenticationFilter | vJWT Validation | vAuthorizationFilter | vDispatcherServlet | vOrderController
```

---

# 13. Why Not Put Spring Security Directly in Servlet Filter?

Because Spring Security needs:

- Dependency Injection
- Configuration
- Application Services
- Database access
- Authentication providers

Example:

```
Security FilterneedsAuthenticationManagerneedsUserDetailsServiceneedsDatabase
```

Spring manages these dependencies.

---

# 14. Debugging DelegatingFilterProxy Problems

## Problem 1: Security Filter Not Running

Check:

```
Request | vTomcat | vDelegatingFilterProxy
```

Possible causes:

- Security auto configuration disabled
- Filter registration issue
- Incorrect application startup

---

# Problem 2: Bean Not Found

Error:

```
No bean named springSecurityFilterChain
```

Meaning:

DelegatingFilterProxy cannot find the Spring Security filter chain.

Check:

- Security configuration
- Spring context startup
- Bean creation

---

# Problem 3: Request Reaches Controller Without Security

Check:

```
Tomcat | vDelegatingFilterProxy | vFilterChainProxy
```

Possible causes:

- Security filter chain missing
- Request matcher does not apply
- Incorrect configuration

---

# 15. Production Example

Microservice:

```
Mobile Client | vAPI Gateway | vOrder Service
```

Inside Order Service:

```
HTTP Request | vEmbedded Tomcat | vDelegatingFilterProxy | vSpring Security | vJWT Validation | vController
```

---

# 16. Common Mistakes

## Mistake 1: Thinking DelegatingFilterProxy Is The Security Filter

Incorrect:

```
DelegatingFilterProxydoes authentication
```

Correct:

```
DelegatingFilterProxydelegates toFilterChainProxyFilterChainProxydoes security processing
```

---

## Mistake 2: Creating Security Filters Manually

Usually unnecessary.

Spring Security manages:

- Filter creation
- Ordering
- Lifecycle

---

## Mistake 3: Ignoring Filter Order

Security depends on order.

Example:

```
Authentication Filter        beforeAuthorization Filter
```

Wrong order causes failures.

---

# 17. Interview Questions

## Q1. What is DelegatingFilterProxy?

Answer:

A Spring-provided Servlet Filter that delegates request processing to a Spring-managed Filter bean.

---

## Q2. Why does Spring Security use DelegatingFilterProxy?

Answer:

Because Servlet containers manage Filters while Spring manages Beans. DelegatingFilterProxy connects both systems.

---

## Q3. Difference between DelegatingFilterProxy and FilterChainProxy?

DelegatingFilterProxy:

```
Servlet → Spring bridge
```

FilterChainProxy:

```
Executes Spring Security filters
```

---

## Q4. Where does DelegatingFilterProxy run?

Answer:

Inside the Servlet container filter chain before DispatcherServlet.

---

# 18. Scenario Based Questions

## Scenario 1

Application starts but security is not applied.

Debug:

```
Tomcat | vDelegatingFilterProxy | vFilterChainProxy
```

---

## Scenario 2

JWT filter is never executed.

Check:

- FilterChainProxy configuration
- SecurityFilterChain matching
- Filter ordering

---

## Scenario 3

Security beans cannot be injected into a filter.

Cause:

Filter created by Tomcat instead of Spring.

Solution:

Use Spring-managed filter registration.

---

# 19. Revision Notes

Remember:

```
Tomcat    |    vDelegatingFilterProxy    |    vFilterChainProxy    |    vSecurityFilterChain    |    vSecurity Filters    |    vDispatcherServlet
```

Key idea:

> DelegatingFilterProxy is the bridge between Servlet architecture and Spring Security architecture.

Important classes:

```
DelegatingFilterProxyFilterChainProxySecurityFilterChainSecurityContextHolderAuthenticationManager
```

---

# Previous / Next Chapter

Previous:

[[Chapter-04-Security-Filter-Chain]]

Next:

[[Chapter-06-SecurityContext-Architecture]]