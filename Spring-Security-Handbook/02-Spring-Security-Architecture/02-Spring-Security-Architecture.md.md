---
title: Spring Security Architecture
chapter: 02
part: Spring Security Handbook
category: Backend Security

level:
  - Beginner
  - Intermediate
  - Advanced

technology:
  language:
    - Java 21+
  framework:
    - Spring Boot 3.x+
    - Spring Security 6.x+
  architecture:
    - REST API
    - Microservices

tags:
  - spring-security
  - spring-boot
  - backend
  - authentication
  - authorization
  - security-filter-chain
  - servlet

status: learning

previous:
  - "[[01-Spring-Security-Fundamentals]]"

next:
  - "[[03-Security-Filter-Chain]]"

related:
  - "[[Authentication]]"
  - "[[Authorization]]"
  - "[[SecurityContext]]"
  - "[[JWT Authentication]]"

created: 2026-06-26
updated: 2026-06-26

---

# Spring Security Architecture

> Spring Security Handbook  
> Beginner → Advanced Backend Engineer

## Learning Goal

Understand how Spring Security internally processes an HTTP request.

By the end of this chapter, you should understand:

- Why Spring Security uses filters
- How Tomcat communicates with Spring Security
- How DelegatingFilterProxy works
- How FilterChainProxy manages security filters
- How SecurityFilterChain is created
- How Authentication happens internally
- How Authorization happens internally
- How SecurityContext stores user information

Think like a backend engineer, not only a configuration user.

---


I will create this as an **Obsidian-ready engineering handbook chapter**. Because the requested chapter is large (architecture + lifecycle + classes + interview questions + revision notes), I will split it into multiple parts.

This is **Part 1**:

- Title
    
- Why Spring Security Architecture Exists
    
- Overall Architecture
    
- Servlet/Tomcat connection
    
- Servlet Filter concept
    
- DelegatingFilterProxy
    

---

```markdown
# Spring Security Architecture

> Spring Security Handbook  
> Chapter 2  
> Beginner → Advanced Backend Engineer

Technology Context:

- Java 21+
- Spring Boot 3.x+
- Spring Security 6.x+
- REST APIs
- Microservices Architecture

Previous Chapter:

```

01 Security Fundamentals

```

This chapter explains:

```

How Spring Security works internally

```

The goal is not only to configure security.

The goal is to understand:

- Request lifecycle
- Security filters
- Authentication flow
- Authorization flow
- Internal Spring Security architecture

---

# 1. Why Spring Security Architecture Exists

## Simple Definition

Spring Security architecture is the internal design that controls:

- How requests enter the application
- How users are authenticated
- How permissions are checked
- How security decisions are made

---

# Why Does This Architecture Exist?

A backend application receives thousands of requests.

Example:

```

GET /api/orders  
POST /api/payment  
DELETE /api/users/10

```

Every request needs security checks.

Without a security framework:

Developers would write:

```

Check login

Check password

Check roles

Check permissions

Create session

Validate tokens

````

inside every controller.

Example:

```java
@GetMapping("/orders")
public List<Order> getOrders(){

    if(user == null){
        throw new Exception();
    }

    if(!user.hasRole("USER")){
        throw new Exception();
    }

    return service.findOrders();
}
````

Problems:

- Duplicate code
    
- Easy to forget checks
    
- Hard to maintain
    
- Security bugs
    

---

# Real World Analogy

Imagine an airport.

Without security:

```
Passenger

   |
   v

Flight
```

Anyone can enter.

---

With security:

```
Passenger

   |
   v

Security Check

   |
   v

Identity Verification

   |
   v

Boarding Permission

   |
   v

Flight
```

Spring Security creates this security checkpoint before your application logic.

---

# Backend Mental Model

Think:

```
Application Code

     |
     |
     v

Security Gate

     |
     |
     v

Internet
```

Security happens BEFORE your controller.

---

# 2. Overall Spring Security Architecture

High-level view:

```
                Client
                  |
                  |
                  v

             HTTP Request


                  |
                  v


        +---------------------+
        |       Tomcat        |
        | Servlet Container   |
        +---------------------+

                  |
                  v


        Servlet Filter Chain


                  |
                  v


        +---------------------+
        | DelegatingFilterProxy|
        +---------------------+

                  |
                  v


        +---------------------+
        | FilterChainProxy    |
        | Spring Security     |
        +---------------------+

                  |
                  v


        SecurityFilterChain


                  |
                  v


        Authentication


                  |
                  v


        Authorization


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

---

# Architecture Flow Explanation

## Client

Examples:

- Browser
    
- Mobile app
    
- Another microservice
    

Sends:

```
HTTP Request
```

Example:

```
GET /api/orders

Authorization:
Bearer eyJhbGc...
```

---

# 3. Servlet Architecture Connection

## Simple Definition

Spring Security is built on top of Java Servlet architecture.

Servlet architecture provides:

- Request handling
    
- Response handling
    
- Filters
    
- Servlet lifecycle
    

Spring Security uses Servlet Filters to intercept requests.

---

# Mental Model

Servlet container:

```
Request

   |
   v

Filters

   |
   v

Servlet

   |
   v

Response
```

Spring Security is one of those filters.

---

# Internal Connection

```
Tomcat

   |
   |
   v

Servlet Filter Chain

   |
   |
   v

Spring Security Filters

   |
   |
   v

DispatcherServlet
```

---

# Production Example

A payment API request:

```
POST /payment
```

Before payment logic executes:

Spring Security checks:

```
Is user logged in?

Does user have permission?

Is token valid?
```

Only then:

```
PaymentService
```

runs.

---

# Common Mistakes

## Mistake

"Controller should handle authentication."

Wrong.

Security should happen before controller.

---

# Interview Questions

## Q1

Why does Spring Security use filters?

Answer:

Because filters execute before servlet/controller processing, allowing security checks before business logic.

---

## Q2

Where does Spring Security sit in a Spring Boot application?

Answer:

Between Tomcat servlet container and application controllers through the servlet filter chain.

---

# Revision Notes

Remember:

```
Tomcat receives request

↓

Filters run

↓

Spring Security checks

↓

Controller executes
```

---

# 4. Tomcat Role in Spring Security

## Simple Definition

Tomcat is the web server and servlet container that receives HTTP requests.

Spring Boot usually runs with embedded Tomcat.

---

# Why Does Tomcat Matter?

Tomcat:

- Opens HTTP port
    
- Receives requests
    
- Creates request objects
    
- Runs servlet filters
    
- Sends responses
    

---

# Mental Model

Tomcat is the building.

Spring Security is the security guard at the entrance.

Application code is inside the building.

---

# Diagram

```
Internet

   |
   v

Tomcat

   |
   v

Security Guard
(Spring Security)

   |
   v

Application
```

---

# Internal Classes

Tomcat works with:

```
HttpServletRequest

HttpServletResponse

Filter

Servlet
```

Spring Security integrates here.

---

# Request Lifecycle

Example:

```
Browser

  |
  v

Tomcat Connector

  |
  v

Servlet Container

  |
  v

Filter Chain

  |
  v

Spring Security

  |
  v

DispatcherServlet
```

---

# Production Example

Microservice:

```
order-service
```

Request:

```
GET /orders
```

Tomcat receives it.

Before:

```
OrderController
```

Spring Security validates identity.

---

# Common Mistakes

## Mistake

Thinking Spring Security is a separate server.

No.

It runs inside your application.

---

# Interview Questions

## Q1

Does Spring Security replace Tomcat?

Answer:

No.

Tomcat handles HTTP and servlet lifecycle. Spring Security handles security concerns through filters.

---

# Revision Notes

Tomcat:

```
Receives request

Runs filters

Calls servlet
```

Spring Security:

```
Uses filters inside Tomcat
```

---

# 5. Servlet Filter Concept

## Simple Definition

A Servlet Filter intercepts requests before they reach the application.

---

# Why Filters Exist

Common tasks:

- Logging
    
- Authentication
    
- Security checks
    
- Compression
    
- CORS handling
    

---

# Mental Model

Filter = checkpoint

```
Request

   |
   v

Filter

   |
   v

Application
```

---

# Multiple Filters

Real application:

```
Request

 |
 v

Logging Filter

 |
 v

Security Filter

 |
 v

CORS Filter

 |
 v

Controller
```

---

# Spring Security Uses Filters For:

- Authentication
    
- Authorization
    
- CSRF protection
    
- Security headers
    

---

# Internal Interface

Java Servlet:

```java
javax.servlet.Filter
```

Concept:

```java
doFilter(
 request,
 response,
 chain
)
```

The filter decides:

Continue?

or

Stop?

---

# Example

Authentication failure:

```
Request

   |
   v

Security Filter

   |
   v

Invalid token

   |
   v

401 Unauthorized
```

Controller never runs.

---

# Production Example

JWT API:

Request:

```
Authorization: Bearer token
```

Filter:

```
Extract token

Validate token

Create Authentication object
```

---

# Common Mistakes

## Mistake

Putting security logic in every controller.

Better:

Use filters.

---

# Interview Questions

## Q1

Why are filters useful for security?

Answer:

Because they provide a central place to process every request before business logic.

---

# Revision Notes

Filter:

```
Request interceptor

Runs before controller

Can stop request
```

---

# 6. DelegatingFilterProxy

## Simple Definition

DelegatingFilterProxy is the bridge between Tomcat Filter system and Spring Security.

---

# Why It Exists

Tomcat knows:

```
Servlet Filters
```

Spring knows:

```
Spring Beans
```

They are different worlds.

DelegatingFilterProxy connects them.

---

# Mental Model

Tomcat:

"I need a filter."

Spring:

"I have security filters."

Bridge:

```
DelegatingFilterProxy
```

---

# Diagram

```
Tomcat

   |
   v

Servlet Filter

   |
   v

DelegatingFilterProxy

   |
   v

Spring Application Context

   |
   v

Spring Security
```

---

# Internal Role

It delegates request processing to:

```
FilterChainProxy
```

---

# Request Flow

```
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

Security Filters
```

---

# Production Example

Every API call:

```
GET /orders
```

passes through:

```
DelegatingFilterProxy
```

before reaching:

```
OrderController
```

---

# Common Mistakes

## Mistake

Calling Spring Security directly from controllers.

Wrong architecture.

---

# Interview Questions

## Q1

What is DelegatingFilterProxy?

Answer:

A servlet filter that delegates processing to Spring-managed security filters.

---

# Revision Notes

Remember:

```
Tomcat Filter World

        |

DelegatingFilterProxy

        |

Spring Security World
```

```

---

Next part will continue with:

- FilterChainProxy
- SecurityFilterChain
- Security Filters
- SecurityContext flow
- AuthenticationManager
- AuthenticationProvider
- Authorization flow
- Full request lifecycle
- Production architecture
- Interview preparation
- Revision sheet
```