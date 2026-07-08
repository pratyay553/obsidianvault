---
title: "Authentication Flow"
chapter: 01
series: "Spring Security Handbook"
topic: "Authentication Lifecycle and Internal Flow"
technology:
  - Java 21
  - Spring Boot 3.x
  - Spring Security 6.x
level: "Intermediate"
tags:
  - spring-security
  - authentication
  - authenticationmanager
  - authenticationprovider
  - userdetailsservice
  - passwordencoder
  - securitycontext
  - backend-engineering
created: 2026-06-30
status: learning
previous:
  - "[[Chapter-0-Spring-Security-Fundamentals]]"
next:
  - "[[Chapter-02-Authentication-vs-Authorization]]"
---


# 1. Learning Objectives

After completing this chapter, I should understand:

- How login works internally in Spring Security
- Complete authentication request flow
- Role of AuthenticationManager
- Role of AuthenticationProvider
- Role of UserDetailsService
- Role of PasswordEncoder
- How Authentication object is created
- How SecurityContextHolder stores logged-in user information
- Difference between authentication before and after success

---

# 2. What is Authentication Flow?

Authentication flow means:

```

How Spring Security verifies a user's identity

````

Simple example:

User enters:

```json
{
 "username":"rahul",
 "password":"12345"
}
````

Spring Security needs to answer:

```
Is this really Rahul?

Is the password correct?
```

If yes:

```
User becomes authenticated
```

---

# 3. High Level Authentication Flow

```
Client

 |
 |
 v

Username + Password

 |
 |
 v

AuthenticationManager

 |
 |
 v

AuthenticationProvider

 |
 |
 v

UserDetailsService

 |
 |
 v

Database

 |
 |
 v

UserDetails

 |
 |
 v

Password Verification

 |
 |
 v

Authentication Object

 |
 |
 v

SecurityContextHolder

 |
 |
 v

Controller
```

---

# 4. Step-by-Step Flow

## Step 1: User Sends Login Request

Example:

```
POST /login
```

Request:

```json
{
 "username":"rahul",
 "password":"password123"
}
```

At this point:

```
User is NOT authenticated
```

Because Spring has not verified anything yet.

---

# Step 2: Spring Creates Authentication Request

Spring creates an object:

```java
UsernamePasswordAuthenticationToken
```

Initially:

```
Principal:
username

Credentials:
password

Authenticated:
false
```

Example:

```
Authentication

username = rahul

password = password123

authenticated = false
```

---

# 5. AuthenticationManager

## What is AuthenticationManager?

AuthenticationManager is the main entry point for authentication.

It receives:

```
Authentication request
```

and decides:

```
Who should authenticate this request?
```

Example:

```java
authenticationManager.authenticate(authentication);
```

---

# Why do we need AuthenticationManager?

Because Spring supports many authentication types:

Example:

```
Username + Password

JWT

OAuth2

LDAP

API Key
```

AuthenticationManager does not implement every logic.

It delegates.

---

# 6. AuthenticationProvider

## What is AuthenticationProvider?

AuthenticationProvider performs the actual authentication.

It checks:

```
Are these credentials valid?
```

Example:

Username/password provider:

```
Receive username/password

        |

        v

Find user

        |

        v

Check password

        |

        v

Return authenticated user
```

---

# AuthenticationManager vs AuthenticationProvider

|Component|Responsibility|
|---|---|
|AuthenticationManager|Coordinates authentication|
|AuthenticationProvider|Performs actual verification|

Simple example:

```
Manager = Restaurant Manager

Provider = Chef
```

Manager decides who handles the work.

Chef actually does the work.

---

# 7. UserDetailsService

## Purpose

UserDetailsService loads user information.

Interface:

```java
UserDetails loadUserByUsername(String username);
```

---

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
```

---

Example:

Database:

```
users table

id:101

username:rahul

password:$2a$10...
```

UserDetails:

```
username = rahul

password = hashed password

authorities = ROLE_USER
```

---

# 8. PasswordEncoder

## Why PasswordEncoder?

Passwords should not be stored directly.

Wrong:

Database:

```
password123
```

Correct:

Database:

```
$2a$10$8Fjd82...
```

---

During login:

User enters:

```
password123
```

Database contains:

```
hashed password
```

PasswordEncoder checks:

```
Does this password match this hash?
```

---

Example:

```java
passwordEncoder.matches(
 rawPassword,
 storedPassword
);
```

---

# 9. Authentication Success

If everything is correct:

```
Username

+

Password

        |

        v

Verified User

        |

        v

Authentication Object
```

Now:

```
authenticated = true
```

Example:

```java
Authentication {

principal = Rahul

authorities = ROLE_USER

authenticated = true

}
```

---

# 10. SecurityContextHolder

After successful authentication:

Spring stores:

```
Authentication Object
```

inside:

```
SecurityContextHolder
```

Flow:

```
Authentication

        |

        v

SecurityContext

        |

        v

SecurityContextHolder
```

---

Example:

```java
Authentication auth =
SecurityContextHolder
.getContext()
.getAuthentication();
```

Now application knows:

```
Current logged-in user = Rahul
```

---

# 11. Complete Internal Flow

```
Request

 |
 v

UsernamePasswordAuthenticationToken

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

UserDetails

 |
 v

PasswordEncoder

 |
 v

Authentication Object

 |
 v

SecurityContextHolder

 |
 v

Controller
```

---

# 12. Authentication Failure Flow

Wrong password:

```
User enters password

        |

        v

Database password check

        |

        v

Mismatch

        |

        v

AuthenticationException

        |

        v

401 Unauthorized
```

---

# 13. Real Production Example

E-commerce login:

User:

```
rahul@gmail.com
```

Password:

```
secret
```

Flow:

1. Request reaches Spring Security
    
2. AuthenticationManager receives request
    
3. AuthenticationProvider starts verification
    
4. UserDetailsService loads Rahul from DB
    
5. PasswordEncoder checks password
    
6. Authentication created
    
7. SecurityContextHolder stores Rahul
    
8. User can access APIs
    

---

# 14. Common Developer Mistakes

## Mistake 1

Thinking Controller handles login.

Wrong:

```
Controller
 |
 Login logic
```

Better:

```
Security Filter

 |

AuthenticationManager

 |

Controller
```

---

## Mistake 2

Saving plain passwords.

Wrong:

```
password123
```

Correct:

```
hashed password
```

---

## Mistake 3

Using database User directly everywhere.

Better:

```
User Entity

        |

        v

UserDetails

        |

        v

Spring Security
```

---

# 15. Interview Questions

## Q1. Explain Spring Security authentication flow.

Answer:

> User sends credentials. Spring creates an Authentication request. AuthenticationManager receives it and delegates to AuthenticationProvider. Provider loads user information using UserDetailsService, verifies password using PasswordEncoder, and creates an authenticated Authentication object. Finally, Spring stores it in SecurityContextHolder.

---

## Q2. Why do we need AuthenticationProvider?

Answer:

> Because different applications authenticate users differently. AuthenticationProvider contains the actual authentication logic.

---

## Q3. What is the role of UserDetailsService?

Answer:

> It loads user information from the data source and provides it to Spring Security in UserDetails format.

---

## Q4. Why use PasswordEncoder?

Answer:

> To compare passwords securely without storing the original password.

---

# 16. Final Mental Model

Remember:

```
AuthenticationManager

=
Who handles authentication?
```

```
AuthenticationProvider

=
How authentication happens?
```

```
UserDetailsService

=
Where user information comes from?
```

```
PasswordEncoder

=
Is password correct?
```

```
SecurityContextHolder

=
Who is currently logged in?
```

---

Next Chapter:

[[Chapter-02-Authentication-vs-Authorization]]

Topics:

- Authentication vs Authorization
    
- 401 vs 403
    
- Roles
    
- Authorities
    
- Permission-based security
    
- Real production scenarios