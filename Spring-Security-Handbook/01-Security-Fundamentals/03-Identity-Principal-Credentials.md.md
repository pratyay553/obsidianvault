---
title: Identity, Principal, and Credentials
chapter: 3
series: Spring Security Handbook
topic: Identity, Principal, Credentials
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: Intermediate
tags:
  - java
  - spring-boot
  - spring-security
  - authentication
  - principal
  - credentials
  - userdetails
  - securitycontext
  - backend
  - rest-api
  - jwt
created: 2026-06-26
status: learning
previous_chapter:
  - Authentication vs Authorization
next_chapter:
  - Authentication Flow Internals
concepts:
  - Identity
  - Principal
  - Credentials
  - Authentication
  - SecurityContextHolder
  - UserDetails
  - UserDetailsService
  - GrantedAuthority
  - Roles
  - Authorities
related:
  - SecurityFilterChain
  - AuthenticationManager
  - AuthenticationProvider
  - JWT
  - OAuth2
---
```markdown
---
title: "Identity, Principal, and Credentials"
chapter: 03
series: "Spring Security Handbook"
topic: "Identity, Principal, Credentials"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: "Intermediate"
tags:
  - spring-security
  - authentication
  - identity
  - principal
  - credentials
  - userdetails
  - securitycontext
  - granted-authority
  - roles
  - permissions
  - backend-engineering
created: 2026-06-26
status: learning
previous:
  - "[[Chapter-02-Authentication-vs-Authorization]]"
related:
  - "[[SecurityContextHolder]]"
  - "[[AuthenticationManager]]"
  - "[[AuthenticationProvider]]"
  - "[[JWT Authentication]]"
---
```

# Chapter-03-Identity-Principal-Credentials

# 1. Learning Objectives

After completing this chapter, I should understand:

- What identity means in a backend system
    
- Difference between User, Identity, and Principal
    
- Why Spring Security uses the Principal concept
    
- How credentials are used only for verification
    
- Why passwords/tokens should not be stored inside authentication context
    
- Internal structure of the `Authentication` object
    
- How `SecurityContextHolder` stores the current authenticated user
    
- Purpose of `UserDetails` and `UserDetailsService`
    
- Difference between Roles and Authorities
    
- How these concepts appear in production systems
    

Technology:

- Java 21
    
- Spring Boot 3.x
    
- Spring Security 6.x
    

---

# 2. Why Identity Exists

Every secure system needs to know:

```
Who is making this request?
```

Example:

An e-commerce API receives:

```
GET /orders/1001
```

The backend must know:

```
Is this request from:

John?
Admin?
Seller?
Anonymous user?
```

Without identity:

- No personalization
    
- No ownership checks
    
- No access control
    

Identity is the foundation of authentication and authorization.

---

# 3. Identity Concept

## Real World

A person has:

```
Person

|
|
Employee ID

|
|
Role
```

Example:

```
Person:
Rahul Sharma

Employee ID:
EMP-1024

Role:
Manager
```

The company does not identify Rahul by his name only.

It uses a unique identifier.

---

## Backend Concept

Backend equivalent:

```
User

|

Identity

|

Principal
```

Example:

Database:

```
users table

id:
101

username:
rahul

email:
rahul@test.com
```

Spring Security converts this into:

```
Principal
```

The Principal represents:

```
The currently authenticated user
```

---

# 4. Principal

## What is Principal?

Principal means:

```
The entity whose identity has been verified
```

In Spring Security:

```java
Authentication authentication =
SecurityContextHolder
.getContext()
.getAuthentication();


Object principal =
authentication.getPrincipal();
```

Usually:

```
principal = UserDetails
```

---

# Why Spring Security uses Principal?

Because authentication can come from different sources:

Username/password:

```
Database User
```

JWT:

```
Token Payload
```

OAuth2:

```
Google User
```

Spring needs a common representation.

That common representation is:

```
Principal
```

---

# Internal Representation

Example:

```java
Authentication authentication =
new UsernamePasswordAuthenticationToken(
        userDetails,
        password,
        authorities
);
```

Internally:

```
Authentication

      |
      |
      +---- Principal

      |
      |
      +---- Credentials

      |
      |
      +---- Authorities
```

---

# 5. Credentials

Credentials prove identity.

Examples:

## Password

Login request:

```json
{
 "username":"john",
 "password":"secret"
}
```

Password is used to verify identity.

---

## JWT Token

Example:

```
Authorization:

Bearer eyJhbGciOiJIUzI1NiJ9
```

JWT proves:

```
This user was already authenticated
```

---

## API Key

Example:

```
X-API-KEY:
abc123xyz
```

Used for:

- Service-to-service communication
    
- External integrations
    

---

# Why credentials are removed after authentication?

Security reason.

Before authentication:

```
Username
Password

        |
        |
Verification
```

After authentication:

```
User Identity

Authorities

Roles
```

Password is no longer needed.

Keeping credentials increases risk.

Example:

Bad:

```
Authentication

{
 principal: john,
 credentials: password123
}
```

Better:

```
Authentication

{
 principal: john,
 credentials: null
}
```

---

# 6. Authentication Object Deep Dive

Spring Security central object:

```
Authentication
```

Structure:

```java
Authentication {

    principal;

    credentials;

    authorities;

    authenticated;

}
```

---

## Principal

Who is logged in.

Example:

```
UserDetails(username=john)
```

---

## Credentials

Proof used during authentication.

Example:

```
password
JWT token
```

Usually removed.

---

## Authorities

Permissions assigned.

Example:

```
ROLE_ADMIN

ORDER_READ

USER_DELETE
```

---

## Authenticated

Boolean value:

```
true

=
identity verified
```

Example:

```java
authentication.isAuthenticated();
```

---

# 7. SecurityContextHolder connection

Spring stores current user information here.

Flow:

```
Client Request


        |

        v


Authentication


        |

        v


SecurityContext


        |

        v


SecurityContextHolder


        |

        v


Controller
```

Example:

```java
Authentication auth =
SecurityContextHolder
.getContext()
.getAuthentication();


String username =
auth.getName();
```

---

# 8. UserDetails

## UserDetails interface

Package:

```
org.springframework.security.core.userdetails.UserDetails
```

Represents authenticated user information.

Example:

```java
public interface UserDetails {

 String getUsername();

 String getPassword();

 Collection<? extends GrantedAuthority>
 getAuthorities();

}
```

---

# Why Spring created UserDetails?

Because users can come from different places:

Database:

```
MySQL User
```

LDAP:

```
Enterprise User
```

OAuth:

```
Google User
```

Spring needs a common contract.

That contract:

```
UserDetails
```

---

# Production usage

Example:

Entity:

```
UserEntity
```

converted into:

```
CustomUserDetails
```

Example:

```java
public class CustomUserDetails 
implements UserDetails {

 private User user;

}
```

---

# 9. UserDetailsService

Purpose:

```
Load user information
```

Interface:

```java
UserDetails loadUserByUsername(
String username
);
```

---

# Database flow

Request:

```
username=john
```

Flow:

```
AuthenticationProvider

        |

        v

UserDetailsService

        |

        v

Database

        |

        v

UserDetails

        |

        v

Authentication
```

---

Example:

```java
@Service
class CustomUserDetailsService 
implements UserDetailsService {


public UserDetails loadUserByUsername(
String username){

 return repository
 .findByUsername(username);

}

}
```

---

# 10. Roles vs Authorities

## Role

High-level grouping.

Example:

```
ADMIN

USER

SELLER
```

Usage:

```java
hasRole("ADMIN")
```

---

## Authority

Fine-grained permission.

Example:

```
USER_CREATE

ORDER_DELETE

PRODUCT_UPDATE
```

Usage:

```java
hasAuthority("ORDER_DELETE")
```

---

Example:

```
ADMIN

 |
 |
 +---- USER_CREATE

 +---- USER_DELETE

 +---- REPORT_VIEW
```

---

# 11. Production Example

## E-commerce Application

Users:

## USER

Can:

```
View products

Create orders

View own orders
```

Authorities:

```
PRODUCT_READ

ORDER_CREATE
```

---

## SELLER

Can:

```
Create products

Update inventory
```

Authorities:

```
PRODUCT_CREATE

PRODUCT_UPDATE
```

---

## ADMIN

Can:

```
Manage users

View reports

Delete accounts
```

Authorities:

```
USER_DELETE

REPORT_VIEW
```

---

# 12. Common Mistakes

## Mistake 1

Using username as identity everywhere.

Better:

Use:

```
User ID
```

because username can change.

---

## Mistake 2

Saving password inside SecurityContext.

Wrong:

```
credentials=password
```

---

## Mistake 3

Confusing Principal and User.

User:

```
Database object
```

Principal:

```
Security identity representation
```

---

## Mistake 4

Using only roles for everything.

Large systems usually use:

```
Roles
+
Permissions
```

---

# 13. Interview Questions

## Q1. What is Principal in Spring Security?

Answer:

Principal represents the currently authenticated user.

---

## Q2. Difference between User and Principal?

Answer:

User:

Database entity.

Principal:

Security representation of authenticated identity.

---

## Q3. What is stored inside Authentication?

Answer:

- Principal
    
- Credentials
    
- Authorities
    
- Authentication status
    

---

## Q4. Why does Spring use UserDetails?

Answer:

To provide a common user representation regardless of where users come from.

---

## Q5. Why are credentials removed after authentication?

Answer:

To reduce security risk and avoid storing sensitive information unnecessarily.

---

# 14. Scenario Questions

## Scenario 1

A user logs in successfully but cannot delete an order.

Reason:

Authentication:

```
SUCCESS
```

Authorization:

```
FAILED
```

Missing authority:

```
ORDER_DELETE
```

---

## Scenario 2

JWT is valid but API returns 403.

Possible reason:

Token user does not have required authority.

---

## Scenario 3

Two users have same username in different systems.

Solution:

Use unique identity:

```
userId
```

not username.

---

# 15. Revision Sheet

Remember:

```
Identity
=
Who the user is
```

```
Principal
=
Authenticated identity representation
```

```
Credentials
=
Proof of identity
```

```
Authentication
=
Principal + Credentials + Authorities
```

```
SecurityContextHolder
=
Current logged-in user storage
```

```
UserDetails
=
Spring Security user contract
```

```
UserDetailsService
=
Loads users
```

```
Roles
=
High level access groups
```

```
Authorities
=
Fine-grained permissions
```

Final mental model:

```
Request

 |
 v

Credentials

 |
 v

Authentication

 |
 v

Principal

 |
 v

SecurityContextHolder

 |
 v

Authorization

 |
 v

Controller
```