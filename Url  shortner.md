For interviews, you don't need to memorize every implementation detail. You should understand **what each technology is, why it's used, and where it fits**. Here's a concise cheat sheet.

# URL Shortener SaaS Architecture Cheat Sheet

---

# 1. Java 21

**What?**  
Programming language used for backend development.

**Why?**

- Fast
    
- Object-Oriented
    
- Secure
    
- LTS (Long Term Support)
    

**Interview**

> Java 21 is the latest LTS version providing modern language features, better performance, virtual threads, and improved memory management.

---

# 2. Spring Boot 3.x

**What?**  
Framework for building Java applications.

**Why?**

- Auto Configuration
    
- Embedded Tomcat
    
- Production Ready
    
- Less Boilerplate
    

Flow

```
Request
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

---

# 3. Spring Security

**Purpose**

Authentication + Authorization

Example

```
Login
↓

JWT Token

↓

Access Protected APIs
```

Interview

> Spring Security protects APIs using authentication and authorization mechanisms like JWT or OAuth2.

---

# 4. Spring Data JPA

**Purpose**

Communicates with database without writing SQL.

Example

Instead of

```sql
SELECT * FROM users;
```

You simply write

```java
userRepository.findAll();
```

---

# 5. PostgreSQL

**Purpose**

Relational Database

Stores

- Users
    
- URLs
    
- Analytics
    
- Roles
    

Interview

> PostgreSQL is an open-source relational database known for ACID compliance and reliability.

---

# 6. Flyway

**Purpose**

Database Version Control

Example

```
V1__Create_User.sql

V2__Create_Url.sql

V3__Add_Clicks.sql
```

Why?

Every developer gets the same database structure.

---

# 7. Lombok

Reduces boilerplate code.

Without Lombok

```java
Getter

Setter

Constructor

toString()

equals()
```

You write all manually.

With Lombok

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
```

Done.

---

# 8. MapStruct

Converts

```
Entity
↓

DTO
```

instead of manually copying fields.

Without

```java
dto.setName(user.getName());
dto.setEmail(user.getEmail());
```

With

```java
UserMapper.toDto(user)
```

---

# 9. Redis

Very fast in-memory database.

Used for

- Cache
    
- OTP
    
- Sessions
    
- Rate Limiting
    

Interview

> Redis stores frequently accessed data in memory, reducing database load and improving response time.

---

# 10. Maven

Build Tool

Handles

- Dependencies
    
- Packaging
    
- Build
    
- Plugins
    

Example

```xml
<dependency>
```

---

# Backend Layers

```
Client
↓

Controller

↓

Service

↓

Repository

↓

Database
```

---

# Controller

Receives HTTP Requests.

Example

```
GET /urls
```

---

# Service

Business Logic

Example

```
Generate Short URL

Validate User

Calculate Expiry
```

---

# Repository

Database Operations

Example

```
save()

findById()

delete()
```

---

# Entity

Represents Database Table

```
User

↓

users table
```

---

# DTO

Transfers Data

Never expose Entity directly.

```
Entity

↓

DTO

↓

JSON
```

---

# Mapper

Converts

```
Entity

↓

DTO

DTO

↓

Entity
```

---

# Config

Application Configurations

Examples

- Redis
    
- Swagger
    
- JWT
    
- CORS
    

---

# Exception

Handles Errors

Instead of

```
500 Internal Error
```

Return

```json
{
"message":"User Not Found"
}
```

---

# Validation

Checks User Input

Example

```java
@NotBlank

@Email

@Size
```

---

# Security

Handles

- Login
    
- JWT
    
- Roles
    
- Permissions
    

---

# Util

Helper Classes

Example

```
JWTUtil

DateUtil

RandomUtil
```

---

# Next.js 15

React Framework

Provides

- Routing
    
- SSR
    
- SEO
    
- API Integration
    

---

# TypeScript

JavaScript + Types

Example

```ts
name:string
```

instead of

```js
name
```

Prevents bugs.

---

# Tailwind CSS

Utility-first CSS

Example

```html
class="bg-blue-500 p-4 rounded"
```

---

# Shadcn UI

Ready-made Components

Examples

- Button
    
- Card
    
- Dialog
    
- Table
    

---

# React Hook Form

Handles

- Forms
    
- Validation
    
- Submission
    

Efficient and fast.

---

# Zod

Schema Validation

Example

```ts
email

password

username
```

Validation before API call.

---

# Axios

HTTP Client

```
Frontend

↓

Backend API
```

---

# TanStack Query

Manages Server State

Handles

- Fetching
    
- Caching
    
- Refetching
    
- Loading
    

---

# Docker

Packages application with all dependencies.

```
My App

+

Java

+

Redis

+

Postgres

=

One Container
```

---

# Docker Compose

Runs multiple containers.

Example

```
Backend

Frontend

Postgres

Redis
```

Start with

```
docker compose up
```

---

# GitHub Actions

CI/CD

Automatically

```
Push

↓

Build

↓

Test

↓

Deploy
```

---

# Swagger

Automatically generates API documentation.

Visit

```
localhost:8080/swagger-ui
```

See all APIs.

---

# Clean Architecture

```
Controller
↓

Service
↓

Repository
↓

Database
```

Rules

- Controller never talks to DB.
    
- Repository never handles business logic.
    
- Service contains all business rules.
    

---

# Feature-Based Frontend

```
features/

login/

dashboard/

url/

analytics/
```

Everything related to one feature stays together.

---

# application.yml

Stores configuration

```
Database

JWT

Redis

Port

Logging
```

---

# Docker Compose

Runs

```
Backend

Frontend

Redis

Postgres
```

using one command.

---

# Environment Variables (.env)

Never hardcode secrets.

```
DB_PASSWORD

JWT_SECRET

REDIS_HOST
```

---

# Swagger Configuration

Enables

```
API Docs

Testing APIs

Authorization
```

---

# Global Exception Handler

One place to handle all errors.

```
Not Found

Validation

Unauthorized

Internal Error
```

---

# Validation

Checks request before processing.

Examples

```
@NotBlank

@NotNull

@Email

@Pattern

@Size
```

---

# Logging

Records application events.

Levels

```
TRACE

DEBUG

INFO

WARN

ERROR
```

Example

```
INFO

User Logged In
```

---

# Response Wrapper

Instead of returning random JSON

Always return

```json
{
"success": true,
"message": "Created Successfully",
"data": {},
"errors": null,
"timestamp": "2026-07-08T12:30:00Z"
}
```

Benefits

- Consistent API responses
    
- Easier frontend handling
    
- Better debugging
    

---

# README

Explains

- Project Overview
    
- Installation
    
- Running the App
    
- Environment Variables
    
- API Documentation
    
- Docker Commands
    
- Folder Structure
    

---

# Typical Request Flow

```text
Client
   │
   ▼
Controller
   │
   ▼
Validation
   │
   ▼
Service
   │
   ▼
Repository
   │
   ▼
PostgreSQL
   │
   ▼
Repository
   │
   ▼
Service
   │
   ▼
Mapper (Entity ↔ DTO)
   │
   ▼
Response Wrapper
   │
   ▼
Client
```

## 5 Interview Questions You Should Be Ready For

1. **Why use DTO instead of Entity?**
    
    - To avoid exposing database models directly and to control the API payload.
        
2. **Why use Redis if PostgreSQL already stores data?**
    
    - PostgreSQL is persistent storage; Redis is an in-memory cache for much faster access to frequently used data.
        
3. **Why MapStruct instead of manual mapping?**
    
    - It generates mapping code at compile time, reducing boilerplate and improving performance.
        
4. **Why Flyway?**
    
    - It versions and automates database schema changes so every environment stays in sync.
        
5. **Why Clean Architecture?**
    
    - It separates responsibilities, making the application easier to maintain, test, and scale.
        

If you understand these concepts and can explain **what each component does, why it's used, and how it interacts with the others**, you'll have a solid foundation for both interviews and building a production-ready Spring Boot + Next.js application.