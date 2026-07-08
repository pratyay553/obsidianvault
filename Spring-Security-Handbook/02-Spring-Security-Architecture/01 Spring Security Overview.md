---
title: "Chapter-01 Spring Security Overview"
series: "Spring Boot Backend Engineering Handbook"
chapter: 1
technology:
  java: "21"
  spring_boot: "3.x"
  spring_security: "6.x"
tags:
  - spring-security
  - spring-boot
  - backend-engineering
  - authentication
  - authorization
status: draft
previous: "[[Previous Chapter]]"
next: "[[Chapter-02 Spring Security Configuration]]"
---

# Chapter-01 Spring Security Overview

## Learning Objectives

After completing this chapter, you should understand:

- Why application security exists in backend systems
- The difference between authentication and authorization
- How Spring Security fits into a Spring Boot application
- The internal architecture of Spring Security 6.x
- How HTTP requests move through the security pipeline
- Important Spring Security classes involved in request processing
- Common enterprise security patterns
- Mistakes developers make while implementing security
- How to answer Spring Security interview questions

---

# 1. Why This Concept Exists

Modern backend applications are exposed to:

- Public APIs
- Mobile applications
- Web clients
- Third-party integrations
- Internal enterprise systems

Without security controls, anyone could:

- Access private data
- Call restricted APIs
- Modify resources
- Impersonate users
- Perform administrative actions

A backend system needs two fundamental questions answered:

## Question 1

"Who are you?"

This is:

**Authentication**

Example:
**User enters:

username: john  
password: secret123

System verifies:

john exists  
password matches

Result:

Authenticated User

```
---## Question 2"What are you allowed to do?"This is:**Authorization**Example:
```

User:

john

Roles:

ROLE_USER

Request:

DELETE /admin/users

Security decision:

ROLE_ADMIN required

Result:

Access Denied

```
---# 2. Simple Definition## Spring SecuritySpring Security is a framework that provides authentication, authorization, and protection mechanisms for Spring applications.It integrates with Spring Boot applications by inserting security filters into the HTTP request processing pipeline.Simple view:
```

Incoming Request

```
    |    v
```

Spring Security

```
    |    +---- Authentication    |    +---- Authorization    |    v
```

Controller

```
---# 3. Mental ModelThink of Spring Security as a security checkpoint at a company entrance.
```

```
            Company Building
```

Request  
|  
v

+---------------------+  
| Security Gate |  
| |  
| 1. Check Identity |  
| 2. Check Permission |  
| |  
+---------------------+

```
    |    v
```

Internal Application

```
The gate decides:- Is this person known?- Are they allowed inside?- What areas can they access?---# 4. Spring Security Architecture Overview
```

Client  
|  
|  
HTTP Request  
|  
v

+--------------------------------+  
| Servlet Container |  
| |  
| Filter Chain |  
+--------------------------------+

```
      |      
      v
```

+--------------------------------+  
| SecurityFilterChain |  
| |  
| Authentication Filters |  
| Authorization Filters |  
+--------------------------------+

```
      |      
      v
```

+--------------------------------+  
| AuthenticationManager |  
+--------------------------------+

```
      |      
      v
```

+--------------------------------+  
| AuthenticationProvider |  
+--------------------------------+

```
      |      
      v
```

+--------------------------------+  
| UserDetailsService |  
+--------------------------------+

```
      |     
      v
```

Database / LDAP / External System

```
---# 5. Important Internal Spring Security Classes## SecurityFilterChainPackage:
```

org.springframework.security.web.SecurityFilterChain

```
Purpose:Defines security rules for incoming requests.Example:```java@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)        throws Exception {    http        .authorizeHttpRequests(auth -> auth            .requestMatchers("/public/**")            .permitAll()            .anyRequest()            .authenticated()        );    return http.build();}
```

---

# FilterChainProxy

Internal class:

```
org.springframework.security.web.FilterChainProxy
```

Purpose:

Entry point for Spring Security filters.

It delegates requests to the correct security filter chain.

Diagram:

```
HTTP Request     |     vFilterChainProxy     |     vSecurityFilterChain     |     vSecurity Filters
```

---

# DelegatingFilterProxy

Spring Security integrates with Servlet filters using:

```
DelegatingFilterProxy
```

Purpose:

Connects the servlet container with Spring-managed security filters.

Flow:

```
Tomcat | vDelegatingFilterProxy | vSpring Security Filter Chain
```

---

# AuthenticationManager

Package:

```
org.springframework.security.authentication.AuthenticationManager
```

Purpose:

Main authentication API.

It receives:

```
Authentication Request
```

and returns:

```
Authenticated Object
```

Example:

```
username/password        |        vAuthenticationManager        |        vAuthenticated User
```

---

# AuthenticationProvider

Responsible for actual authentication logic.

Examples:

- Database username/password
- LDAP
- JWT validation
- OAuth2 login

Flow:

```
AuthenticationManager        |        vAuthenticationProvider        |        vUserDetailsService        |        vDatabase
```

---

# UserDetailsService

Purpose:

Loads user information.

Example:

```
public interface UserDetailsService {    UserDetails loadUserByUsername(String username);}
```

Usually:

```
username   |   vDatabase Query   |   vUserDetails
```

---

# SecurityContextHolder

Stores authenticated user information.

Example:

```
Request | vAuthentication | vSecurityContext | vSecurityContextHolder
```

Access example:

```
Authentication auth =    SecurityContextHolder        .getContext()        .getAuthentication();
```

---

# 6. HTTP Request Lifecycle

Complete lifecycle:

```
Client | | | GET /api/orders | vServlet Container | vDelegatingFilterProxy | vFilterChainProxy | vSecurity Filters | | +----------------------+ |                      | v                      vAuthentication       Authorization | | vSecurityContextHolder | | vDispatcherServlet | | vController | | vResponse
```

---

# 7. Authentication Lifecycle

Example:

```
POST /loginusernamepassword        |        vUsernamePasswordAuthenticationFilter        |        vAuthenticationManager        |        vDaoAuthenticationProvider        |        vUserDetailsService        |        vDatabase        |        vAuthentication Object        |        vSecurityContext
```

---

# 8. Authorization Lifecycle

Example:

```
GET /admin/users        |        vIs User Authenticated?        |        vCheck Authorities        |        vROLE_ADMIN?     /       \   YES       NO    |         |Allow     AccessDeniedException
```

---

# 9. Production Examples

## Example 1: Banking API

Requirement:

```
GET /accounts/{id}
```

Rules:

```
Customer can see own accountAdmin can see all accounts
```

Security:

Authentication:

```
JWT Token
```

Authorization:

```
ROLE_CUSTOMERROLE_ADMIN
```

---

## Example 2: E-Commerce Platform

Roles:

```
CUSTOMERSELLERADMIN
```

Permissions:

```
CUSTOMER- Place orderSELLER- Add productADMIN- Delete user
```

---

## Example 3: Microservices

Architecture:

```
Client | vAPI Gateway | vAuthentication Service | vOrder Service | vPayment Service
```

Usually:

- OAuth2
- JWT
- Service-to-service authentication

---

# 10. Common Mistakes

## Mistake 1: Confusing Authentication and Authorization

Wrong:

"User has ROLE_ADMIN so authentication is done."

Correct:

Authentication:

```
Who is the user?
```

Authorization:

```
What can the user do?
```

---

## Mistake 2: Disabling Security Temporarily

Bad:

```
http.csrf(csrf -> csrf.disable()).authorizeHttpRequests(auth ->    auth.anyRequest().permitAll());
```

Production risk:

Everything becomes public.

---

## Mistake 3: Storing Plain Passwords

Wrong:

```
password = "hello123"
```

Correct:

Use:

- BCrypt
- PasswordEncoder

---

## Mistake 4: Putting Business Rules Inside Filters

Bad:

```
Security Filter    |    +---- Calculate order price
```

Filters should handle security concerns only.

---

## Mistake 5: Using Deprecated Configuration

Avoid:

```
extends WebSecurityConfigurerAdapter
```

Spring Security 6 uses:

```
@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)
```

---

# 11. Interview Questions

## Q1. What is Spring Security?

Answer:

Spring Security is a framework that provides authentication, authorization, and protection against common security attacks in Spring applications.

---

## Q2. Difference between Authentication and Authorization?

Authentication:

```
Identity verification
```

Authorization:

```
Permission checking
```

---

## Q3. What is SecurityFilterChain?

Answer:

A chain of security filters responsible for processing incoming HTTP requests.

---

## Q4. What does AuthenticationManager do?

Answer:

It coordinates authentication by delegating authentication requests to AuthenticationProviders.

---

## Q5. Where does Spring Security store authenticated user information?

Answer:

Inside:

```
SecurityContextHolder
```

---

# 12. Scenario Based Questions

## Scenario 1

A user logs in successfully but cannot access:

```
GET /admin/report
```

Possible reasons:

- Missing ROLE_ADMIN authority
- Incorrect authorization rule
- Wrong granted authority format

---

## Scenario 2

Every API returns 401.

Possible reasons:

- Missing authentication token
- Invalid JWT
- Security filter misconfiguration

---

## Scenario 3

User can access another user's data.

Possible issue:

Authorization logic missing.

Example:

Bad:

```
GET /orders/101
```

Only checks:

```
USER is authenticated
```

Should check:

```
USER owns order 101
```

---

# 13. Revision Notes

## Core Concepts

```
Authentication    |    vWho are you?Authorization    |    vWhat can you access?
```

---

## Important Classes

```
SecurityFilterChainFilterChainProxyAuthenticationManagerAuthenticationProviderUserDetailsServiceSecurityContextHolder
```

---

## Request Flow

```
Request -> Security Filters -> Authentication -> Authorization -> Controller -> Response
```

---

## Modern Spring Security Style

Use:

```
SecurityFilterChain Bean
```

Avoid:

```
WebSecurityConfigurerAdapter
```

---

# Previous / Next Chapter

Previous:

[[Previous Chapter]]

Next:
User enters:

username: john  
password: secret123

System verifies:

john exists  
password matches

Result:

Authenticated User

```
---## Question 2"What are you allowed to do?"This is:**Authorization**Example:
```

User:

john

Roles:

ROLE_USER

Request:

DELETE /admin/users

Security decision:

ROLE_ADMIN required

Result:

Access Denied

```
---# 2. Simple Definition## Spring SecuritySpring Security is a framework that provides authentication, authorization, and protection mechanisms for Spring applications.It integrates with Spring Boot applications by inserting security filters into the HTTP request processing pipeline.Simple view:
```

Incoming Request

```
    |    v
```

Spring Security

```
    |    +---- Authentication    |    +---- Authorization    |    v
```

Controller

```
---# 3. Mental ModelThink of Spring Security as a security checkpoint at a company entrance.
```

```
            Company Building
```

Request  
|  
v

+---------------------+  
| Security Gate |  
| |  
| 1. Check Identity |  
| 2. Check Permission |  
| |  
+---------------------+

```
    |    v
```

Internal Application

```
The gate decides:- Is this person known?- Are they allowed inside?- What areas can they access?---# 4. Spring Security Architecture Overview
```

Client  
|  
|  
HTTP Request  
|  
v

+--------------------------------+  
| Servlet Container |  
| |  
| Filter Chain |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| SecurityFilterChain |  
| |  
| Authentication Filters |  
| Authorization Filters |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| AuthenticationManager |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| AuthenticationProvider |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| UserDetailsService |  
+--------------------------------+

```
      |      v
```

Database / LDAP / External System

```
---# 5. Important Internal Spring Security Classes## SecurityFilterChainPackage:
```

org.springframework.security.web.SecurityFilterChain

```
Purpose:Defines security rules for incoming requests.Example:```java@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)        throws Exception {    http        .authorizeHttpRequests(auth -> auth            .requestMatchers("/public/**")            .permitAll()            .anyRequest()            .authenticated()        );    return http.build();}
```

---

# FilterChainProxy

Internal class:

```
org.springframework.security.web.FilterChainProxy
```

Purpose:

Entry point for Spring Security filters.

It delegates requests to the correct security filter chain.

Diagram:

```
HTTP Request     |     vFilterChainProxy     |     vSecurityFilterChain     |     vSecurity Filters
```

---

# DelegatingFilterProxy

Spring Security integrates with Servlet filters using:

```
DelegatingFilterProxy
```

Purpose:

Connects the servlet container with Spring-managed security filters.

Flow:

```
Tomcat | vDelegatingFilterProxy | vSpring Security Filter Chain
```

---

# AuthenticationManager

Package:

```
org.springframework.security.authentication.AuthenticationManager
```

Purpose:

Main authentication API.

It receives:

```
Authentication Request
```

and returns:

```
Authenticated Object
```

Example:

```
username/password        |        vAuthenticationManager        |        vAuthenticated User
```

---

# AuthenticationProvider

Responsible for actual authentication logic.

Examples:

- Database username/password
- LDAP
- JWT validation
- OAuth2 login

Flow:

```
AuthenticationManager        |        vAuthenticationProvider        |        vUserDetailsService        |        vDatabase
```

---

# UserDetailsService

Purpose:

Loads user information.

Example:

```
public interface UserDetailsService {    UserDetails loadUserByUsername(String username);}
```

Usually:

```
username   |   vDatabase Query   |   vUserDetails
```

---

# SecurityContextHolder

Stores authenticated user information.

Example:

```
Request | vAuthentication | vSecurityContext | vSecurityContextHolder
```

Access example:

```
Authentication auth =    SecurityContextHolder        .getContext()        .getAuthentication();
```

---

# 6. HTTP Request Lifecycle

Complete lifecycle:

```
Client | | | GET /api/orders | vServlet Container | vDelegatingFilterProxy | vFilterChainProxy | vSecurity Filters | | +----------------------+ |                      | v                      vAuthentication       Authorization | | vSecurityContextHolder | | vDispatcherServlet | | vController | | vResponse
```

---

# 7. Authentication Lifecycle

Example:

```
POST /loginusernamepassword        |        vUsernamePasswordAuthenticationFilter        |        vAuthenticationManager        |        vDaoAuthenticationProvider        |        vUserDetailsService        |        vDatabase        |        vAuthentication Object        |        vSecurityContext
```

---

# 8. Authorization Lifecycle

Example:

```
GET /admin/users        |        vIs User Authenticated?        |        vCheck Authorities        |        vROLE_ADMIN?     /       \   YES       NO    |         |Allow     AccessDeniedException
```

---

# 9. Production Examples

## Example 1: Banking API

Requirement:

```
GET /accounts/{id}
```

Rules:

```
Customer can see own accountAdmin can see all accounts
```

Security:

Authentication:

```
JWT Token
```

Authorization:

```
ROLE_CUSTOMERROLE_ADMIN
```

---

## Example 2: E-Commerce Platform

Roles:

```
CUSTOMERSELLERADMIN
```

Permissions:

```
CUSTOMER- Place orderSELLER- Add productADMIN- Delete user
```

---

## Example 3: Microservices

Architecture:

```
Client | vAPI Gateway | vAuthentication Service | vOrder Service | vPayment Service
```

Usually:

- OAuth2
- JWT
- Service-to-service authentication

---

# 10. Common Mistakes

## Mistake 1: Confusing Authentication and Authorization

Wrong:

"User has ROLE_ADMIN so authentication is done."

Correct:

Authentication:

```
Who is the user?
```

Authorization:

```
What can the user do?
```

---

## Mistake 2: Disabling Security Temporarily

Bad:

```
http.csrf(csrf -> csrf.disable()).authorizeHttpRequests(auth ->    auth.anyRequest().permitAll());
```

Production risk:

Everything becomes public.

---

## Mistake 3: Storing Plain Passwords

Wrong:

```
password = "hello123"
```

Correct:

Use:

- BCrypt
- PasswordEncoder

---

## Mistake 4: Putting Business Rules Inside Filters

Bad:

```
Security Filter    |    +---- Calculate order price
```

Filters should handle security concerns only.

---

## Mistake 5: Using Deprecated Configuration

Avoid:

```
extends WebSecurityConfigurerAdapter
```

Spring Security 6 uses:

```
@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)
```

---

# 11. Interview Questions

## Q1. What is Spring Security?

Answer:

Spring Security is a framework that provides authentication, authorization, and protection against common security attacks in Spring applications.

---

## Q2. Difference between Authentication and Authorization?

Authentication:

```
Identity verification
```

Authorization:

```
Permission checking
```

---

## Q3. What is SecurityFilterChain?

Answer:

A chain of security filters responsible for processing incoming HTTP requests.

---

## Q4. What does AuthenticationManager do?

Answer:

It coordinates authentication by delegating authentication requests to AuthenticationProviders.

---

## Q5. Where does Spring Security store authenticated user information?

Answer:

Inside:

```
SecurityContextHolder
```

---

# 12. Scenario Based Questions

## Scenario 1

A user logs in successfully but cannot access:

```
GET /admin/report
```

Possible reasons:

- Missing ROLE_ADMIN authority
- Incorrect authorization rule
- Wrong granted authority format

---

## Scenario 2

Every API returns 401.

Possible reasons:

- Missing authentication token
- Invalid JWT
- Security filter misconfiguration

---

## Scenario 3

User can access another user's data.

Possible issue:

Authorization logic missing.

Example:

Bad:

```
GET /orders/101
```

Only checks:

```
USER is authenticated
```

Should check:

```
USER owns order 101
```

---

# 13. Revision Notes

## Core Concepts

```
Authentication    |    vWho are you?Authorization    |    vWhat can you access?
```

---

## Important Classes

```
SecurityFilterChainFilterChainProxyAuthenticationManagerAuthenticationProviderUserDetailsServiceSecurityContextHolder
```

---

## Request Flow

```
Request -> Security Filters -> Authentication -> Authorization -> Controller -> Response
```

---

## Modern Spring Security Style

Use:

```
SecurityFilterChain Bean
```

Avoid:

```
WebSecurityConfigurerAdapter
```

---

# Previous / Next Chapter

Previous:

[[Previous Chapter]]

Next:
User:

john

Roles:

ROLE_USER

Request:

DELETE /admin/users

Security decision:

ROLE_ADMIN required

Result:

Access Denied

```
---# 2. Simple Definition## Spring SecuritySpring Security is a framework that provides authentication, authorization, and protection mechanisms for Spring applications.It integrates with Spring Boot applications by inserting security filters into the HTTP request processing pipeline.Simple view:
```

Incoming Request

```
    |    v
```

Spring Security

```
    |    +---- Authentication    |    +---- Authorization    |    v
```

Controller

```
---# 3. Mental ModelThink of Spring Security as a security checkpoint at a company entrance.
```

```
            Company Building
```

Request  
|  
v

+---------------------+  
| Security Gate |  
| |  
| 1. Check Identity |  
| 2. Check Permission |  
| |  
+---------------------+

```
    |    v
```

Internal Application

```
The gate decides:- Is this person known?- Are they allowed inside?- What areas can they access?---# 4. Spring Security Architecture Overview
```

Client  
|  
|  
HTTP Request  
|  
v

+--------------------------------+  
| Servlet Container |  
| |  
| Filter Chain |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| SecurityFilterChain |  
| |  
| Authentication Filters |  
| Authorization Filters |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| AuthenticationManager |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| AuthenticationProvider |  
+--------------------------------+

```
      |      v
```

+--------------------------------+  
| UserDetailsService |  
+--------------------------------+

```
      |      v
```

Database / LDAP / External System

```
---# 5. Important Internal Spring Security Classes## SecurityFilterChainPackage:
```

org.springframework.security.web.SecurityFilterChain

```
Purpose:Defines security rules for incoming requests.Example:```java@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)        throws Exception {    http        .authorizeHttpRequests(auth -> auth            .requestMatchers("/public/**")            .permitAll()            .anyRequest()            .authenticated()        );    return http.build();}
```

---

# FilterChainProxy

Internal class:

```
org.springframework.security.web.FilterChainProxy
```

Purpose:

Entry point for Spring Security filters.

It delegates requests to the correct security filter chain.

Diagram:

```
HTTP Request     |     vFilterChainProxy     |     vSecurityFilterChain     |     vSecurity Filters
```

---

# DelegatingFilterProxy

Spring Security integrates with Servlet filters using:

```
DelegatingFilterProxy
```

Purpose:

Connects the servlet container with Spring-managed security filters.

Flow:

```
Tomcat | vDelegatingFilterProxy | vSpring Security Filter Chain
```

---

# AuthenticationManager

Package:

```
org.springframework.security.authentication.AuthenticationManager
```

Purpose:

Main authentication API.

It receives:

```
Authentication Request
```

and returns:

```
Authenticated Object
```

Example:

```
username/password        |        vAuthenticationManager        |        vAuthenticated User
```

---

# AuthenticationProvider

Responsible for actual authentication logic.

Examples:

- Database username/password
- LDAP
- JWT validation
- OAuth2 login

Flow:

```
AuthenticationManager        |        vAuthenticationProvider        |        vUserDetailsService        |        vDatabase
```

---

# UserDetailsService

Purpose:

Loads user information.

Example:

```
public interface UserDetailsService {    UserDetails loadUserByUsername(String username);}
```

Usually:

```
username   |   vDatabase Query   |   vUserDetails
```

---

# SecurityContextHolder

Stores authenticated user information.

Example:

```
Request | vAuthentication | vSecurityContext | vSecurityContextHolder
```

Access example:

```
Authentication auth =    SecurityContextHolder        .getContext()        .getAuthentication();
```

---

# 6. HTTP Request Lifecycle

Complete lifecycle:

```
Client | | | GET /api/orders | vServlet Container | vDelegatingFilterProxy | vFilterChainProxy | vSecurity Filters | | +----------------------+ |                      | v                      vAuthentication       Authorization | | vSecurityContextHolder | | vDispatcherServlet | | vController | | vResponse
```

---

# 7. Authentication Lifecycle

Example:

```
POST /loginusernamepassword        |        vUsernamePasswordAuthenticationFilter        |        vAuthenticationManager        |        vDaoAuthenticationProvider        |        vUserDetailsService        |        vDatabase        |        vAuthentication Object        |        vSecurityContext
```

---

# 8. Authorization Lifecycle

Example:

```
GET /admin/users        |        vIs User Authenticated?        |        vCheck Authorities        |        vROLE_ADMIN?     /       \   YES       NO    |         |Allow     AccessDeniedException
```

---

# 9. Production Examples

## Example 1: Banking API

Requirement:

```
GET /accounts/{id}
```

Rules:

```
Customer can see own accountAdmin can see all accounts
```

Security:

Authentication:

```
JWT Token
```

Authorization:

```
ROLE_CUSTOMERROLE_ADMIN
```

---

## Example 2: E-Commerce Platform

Roles:

```
CUSTOMERSELLERADMIN
```

Permissions:

```
CUSTOMER- Place orderSELLER- Add productADMIN- Delete user
```

---

## Example 3: Microservices

Architecture:

```
Client | vAPI Gateway | vAuthentication Service | vOrder Service | vPayment Service
```

Usually:

- OAuth2
- JWT
- Service-to-service authentication

---

# 10. Common Mistakes

## Mistake 1: Confusing Authentication and Authorization

Wrong:

"User has ROLE_ADMIN so authentication is done."

Correct:

Authentication:

```
Who is the user?
```

Authorization:

```
What can the user do?
```

---

## Mistake 2: Disabling Security Temporarily

Bad:

```
http.csrf(csrf -> csrf.disable()).authorizeHttpRequests(auth ->    auth.anyRequest().permitAll());
```

Production risk:

Everything becomes public.

---

## Mistake 3: Storing Plain Passwords

Wrong:

```
password = "hello123"
```

Correct:

Use:

- BCrypt
- PasswordEncoder

---

## Mistake 4: Putting Business Rules Inside Filters

Bad:

```
Security Filter    |    +---- Calculate order price
```

Filters should handle security concerns only.

---

## Mistake 5: Using Deprecated Configuration

Avoid:

```
extends WebSecurityConfigurerAdapter
```

Spring Security 6 uses:

```
@BeanSecurityFilterChain securityFilterChain(HttpSecurity http)
```

---

# 11. Interview Questions

## Q1. What is Spring Security?

Answer:

Spring Security is a framework that provides authentication, authorization, and protection against common security attacks in Spring applications.

---

## Q2. Difference between Authentication and Authorization?

Authentication:

```
Identity verification
```

Authorization:

```
Permission checking
```

---

## Q3. What is SecurityFilterChain?

Answer:

A chain of security filters responsible for processing incoming HTTP requests.

---

## Q4. What does AuthenticationManager do?

Answer:

It coordinates authentication by delegating authentication requests to AuthenticationProviders.

---

## Q5. Where does Spring Security store authenticated user information?

Answer:

Inside:

```
SecurityContextHolder
```

---

# 12. Scenario Based Questions

## Scenario 1

A user logs in successfully but cannot access:

```
GET /admin/report
```

Possible reasons:

- Missing ROLE_ADMIN authority
- Incorrect authorization rule
- Wrong granted authority format

---

## Scenario 2

Every API returns 401.

Possible reasons:

- Missing authentication token
- Invalid JWT
- Security filter misconfiguration

---

## Scenario 3

User can access another user's data.

Possible issue:

Authorization logic missing.

Example:

Bad:

```
GET /orders/101
```

Only checks:

```
USER is authenticated
```

Should check:

```
USER owns order 101
```

---

# 13. Revision Notes

## Core Concepts

```
Authentication    |    vWho are you?Authorization    |    vWhat can you access?
```

---

## Important Classes

```
SecurityFilterChainFilterChainProxyAuthenticationManagerAuthenticationProviderUserDetailsServiceSecurityContextHolder
```

---

## Request Flow

```
Request -> Security Filters -> Authentication -> Authorization -> Controller -> Response
```

---

## Modern Spring Security Style

Use:

```
SecurityFilterChain Bean
```

Avoid:

```
WebSecurityConfigurerAdapter
```

---

# Previous / Next Chapter

Previous:

[[Previous Chapter]]

Next:
