---
title: Servlet Architecture
chapter: 3
category: Spring Security Foundation
tags:
  - Servlet
  - HTTP
  - Spring Boot
  - Web Application Architecture
technology:
  - Java 21
  - Spring Boot 3
  - Spring Security 6
  - Servlet API
previous: "[[Chapter-02-Spring-Security-Architecture]]"
next: "[[Chapter-04-Security-Filter-Chain]]"
---

# Servlet Architecture

## Learning Objectives

After completing this chapter, you should understand:

- What the Servlet API is and why it exists
- How HTTP requests are processed inside Java backend applications
- The role of Servlet Container in Spring Boot applications
- Difference between Servlet, Filter, Listener, and Controller
- How Spring MVC fits on top of Servlet architecture
- Where Spring Security enters the request lifecycle
- How to debug request flow issues in enterprise applications
- How to explain Servlet architecture during backend interviews

---

# 1. Why Servlet Architecture Exists

Before frameworks like Spring MVC and Spring Boot, Java developers needed a standard way to build web applications.

Every application server had different ways of handling:

- HTTP requests
- URL routing
- Request parameters
- Sessions
- Responses

Example problem:
Application Server A

handleRequest()

Application Server B

processHttp()

Application Server C

executeWebRequest()

```
There was no common programming model.The Servlet specification solved this problem.It defined a standard contract:
```

HTTP Request

```
    |    v
```

Servlet

```
    |    v
```

HTTP Response

```
Any Java web server implementing the Servlet specification could execute the same application.---# 2. Simple Definition## ServletA Servlet is a Java component that receives HTTP requests and generates HTTP responses.It runs inside a Servlet Container.Example:
```

Client

|  
|  
HTTP Request

|  
v

Servlet Container

|  
v

Servlet

|  
v

HTTP Response

```
---# 3. Big Picture Mental ModelThink of a Servlet Container as a traffic controller.Thousands of requests arrive:
```

```
            Internet
```

Request 1 ----  
Request 2 -----  
Request 3 -------> Servlet Container  
Request 4 -----/  
Request 5 ----/

```
                 |                 |          Application Code
```

```
The container manages:- Request lifecycle- Thread creation- Servlet execution- Security filters- Sessions- Resource management---# 4. Servlet Ecosystem OverviewComplete architecture:
```

```
              Client                |                |          HTTP Request                |                v    +-----------------------+    | Servlet Container     |    |                       |    | Example: Tomcat       |    +-----------------------+                |                |    +-----------------------+    | Filter Chain          |    +-----------------------+                |                |    +-----------------------+    | Servlet               |    +-----------------------+                |                |    +-----------------------+    | Spring MVC            |    | DispatcherServlet     |    +-----------------------+                |                |    +-----------------------+    | Controller            |    +-----------------------+
```

```
---# 5. Servlet Container## What is a Servlet Container?A Servlet Container is a runtime environment that executes Servlets.Examples:- Tomcat- Jetty- UndertowSpring Boot commonly uses embedded Tomcat.Example:
```

Spring Boot Application

```
    |    v
```

Embedded Tomcat

```
    |    v
```

Servlet API

```
    |    v
```

Spring MVC

```
---# 6. Responsibilities of Servlet ContainerThe container manages:## 1. Servlet Lifecycle
```

Create Servlet

```
   |   v
```

Initialize

```
   |   v
```

Handle Requests

```
   |   v
```

Destroy

```
---## 2. Thread ManagementA single Servlet instance handles multiple requests.Example:
```

```
            Servlet
```

Request 1 ---> Thread 1

Request 2 ---> Thread 2

Request 3 ---> Thread 3

```
Important:Servlets must be thread-safe.---## 3. Request / Response ObjectsContainer creates:
```

HttpServletRequest

HttpServletResponse

```
Example:```javaprotected void doGet(    HttpServletRequest request,    HttpServletResponse response)
```

---

# 7. Core Servlet Components

## Servlet

Main component that handles requests.

Example:

```
public class OrderServlet extends HttpServlet {    protected void doGet(        HttpServletRequest request,        HttpServletResponse response    ) {        response.getWriter()                .write("Orders");    }}
```

---

## Filter

A Filter runs before or after a Servlet.

Example:

```
Request | vAuthentication Filter | vLogging Filter | vServlet | vResponse
```

Filters are extremely important in Spring Security.

---

## Listener

A Listener reacts to application events.

Examples:

- Application startup
- Session creation
- Context initialization

---

# 8. Spring Boot Inside Servlet Architecture

Spring Boot does not replace Servlet architecture.

It builds on top of it.

Architecture:

```
Spring Boot Application        |        vEmbedded Tomcat        |        vServlet Container        |        vDispatcherServlet        |        vController
```

---

# 9. DispatcherServlet

The most important Servlet in Spring MVC.

Class:

```
org.springframework.web.servlet.DispatcherServlet
```

It works as the Front Controller.

Meaning:

All requests come through it.

Example:

```
GET /users        |        vDispatcherServlet        |        +-------- UserControllerGET /orders        |        vDispatcherServlet        |        +-------- OrderController
```

---

# 10. Request Lifecycle in Spring Boot

Detailed flow:

```
Client | |HTTP Request | vTomcat | vFilter Chain | vDispatcherServlet | vHandlerMapping | vController | vService | vRepository | vDatabase | vResponse
```

---

# 11. Where Spring Security Fits

Spring Security works at the Filter level.

It does NOT start inside Controller.

Flow:

```
HTTP Request      |      vServlet Container      |      vSpring Security Filters      |      vDispatcherServlet      |      vController
```

Security gets the first opportunity to inspect requests.

---

# 12. Spring Security + Servlet Architecture

Complete enterprise flow:

```
Client | |JWT / Session / Credentials | vTomcat | vDelegatingFilterProxy | vSpring Security Filter Chain | vAuthentication | vAuthorization | vDispatcherServlet | vController | vBusiness Logic
```

---

# 13. Important Internal Classes

## HttpServletRequest

Represents incoming request.

Contains:

- Headers
- Parameters
- Cookies
- HTTP method
- URL

Example:

```
request.getHeader("Authorization");
```

---

## HttpServletResponse

Represents outgoing response.

Example:

```
response.setStatus(200);
```

---

## ServletContext

Represents application-wide information.

Used for:

- Shared configuration
- Application lifecycle

---

## DispatcherServlet

Spring MVC entry point.

---

## FilterChain

Represents ordered execution of filters.

Example:

```
Filter 1   |Filter 2   |Filter 3   |Servlet
```

---

# 14. Debugging Mental Model

When an API fails, ask:

## Problem:

Controller is not reached.

Check:

```
Client | vServlet Container | vFilters | vDispatcherServlet | XController
```

Possible causes:

- Security rejection
- Filter exception
- Invalid request mapping

---

## Problem:

Controller executes but response is wrong.

Check:

```
Controller | vService | vDatabase
```

---

# 15. Production Example

## Banking Application

Request:

```
GET /accounts/123Authorization:Bearer JWT_TOKEN
```

Flow:

```
Mobile App | vAPI Gateway | vTomcat | vSecurity Filter | vValidate JWT | vDispatcherServlet | vAccountController | vAccountService | vDatabase
```

---

# 16. Common Mistakes

## Mistake 1: Thinking Spring Security is a Controller Feature

Wrong:

```
Controller checks authentication
```

Correct:

```
Security Filters check before Controller
```

---

## Mistake 2: Ignoring Servlet Thread Model

A Servlet handles multiple requests.

Bad:

```
class MyServlet {    private String username;}
```

Because multiple threads share state.

---

## Mistake 3: Doing Heavy Work Inside Filters

Bad:

```
Filter | +---- Generate reports +---- Call multiple services
```

Filters should remain focused.

---

## Mistake 4: Not Understanding Embedded Servers

Spring Boot application:

```
java -jar application.jar
```

still runs:

```
Application | Embedded Tomcat | Servlet Container
```

---

# 17. Interview Questions

## Q1. What is Servlet?

Answer:

A Servlet is a Java component that handles HTTP requests and responses inside a Servlet Container.

---

## Q2. What is the role of Servlet Container?

Answer:

It manages Servlet lifecycle, request handling, threading, sessions, and communication between HTTP clients and application code.

---

## Q3. How does Spring MVC relate to Servlet?

Answer:

Spring MVC is built on top of Servlet API.

DispatcherServlet is a Servlet that acts as the front controller.

---

## Q4. Where does Spring Security execute?

Answer:

Spring Security executes through Servlet Filters before requests reach DispatcherServlet.

---

## Q5. Difference between Filter and Servlet?

Filter:

```
Intercepts requests
```

Servlet:

```
Processes application request
```

---

# 18. Scenario Based Questions

## Scenario 1

A request returns 403 before reaching Controller.

Where to debug?

Answer:

```
Spring Security Filter ChainAuthorization RulesAuthorities
```

---

## Scenario 2

A request never reaches application.

Check:

```
Tomcat logsFilter exceptionsServlet mapping
```

---

## Scenario 3

Two users see each other's data.

Possible issue:

- Shared Servlet state
- Incorrect session handling
- Missing authorization checks

---

# 19. Revision Notes

## Core Flow

```
HTTP Request    |    vServlet Container    |    vFilters    |    vDispatcherServlet    |    vController
```

---

## Remember

Servlet is the foundation.

Spring MVC sits on Servlet.

Spring Security uses Servlet Filters.

Architecture:

```
Servlet Container        |        vSecurity Filters        |        vDispatcherServlet        |        vControllers
```

---

# Previous / Next Chapter

Previous:

[[Chapter-02-Spring-Security-Architecture]]

Next:

[[Chapter-04-Security-Filter-Chain]]