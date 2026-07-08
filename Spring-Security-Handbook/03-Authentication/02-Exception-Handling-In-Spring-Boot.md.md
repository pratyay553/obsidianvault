---
title: "Exception Handling in Spring Boot"
chapter: 02
tags:
  - spring-boot
  - spring-framework
  - exception-handling
  - rest-api
  - controller-advice
  - exception-handler
  - backend-engineering
  - enterprise-api-design
status: learning
java_version: "21"
spring_boot_version: "3.x"
spring_framework_version: "6.x"
previous:
  - "[[Chapter-01-Authentication-Flow]]"
next:
  - "[[Chapter-03-Identity-Principal-Credentials]]"
created: 2026-06-30
---

# Chapter-02-Exception-Handling-In-Spring-Boot

> Enterprise Backend Engineering Handbook  
> Java 21 · Spring Boot 3.x · Spring Framework 6.x · Spring MVC · REST APIs
---

---

# 1. Learning Objectives

After completing this chapter, I should understand:

- Why exception handling is required in backend systems
    
- Difference between checked and unchecked exceptions
    
- How Spring Boot handles exceptions internally
    
- How to create custom exceptions
    
- How `@ControllerAdvice` and `@ExceptionHandler` work
    
- How to create consistent API error responses
    
- How exception handling works in production systems
    
- How enterprise applications separate business failures and technical failures
    

---

# 2. Why Exception Handling Exists

Every backend application deals with failures.

Examples:

- User does not exist
    
- Database is unavailable
    
- Payment service fails
    
- Invalid input is received
    
- External API is down
    

A backend developer cannot assume:

```
Everything will always work
```

A production system must handle failures gracefully.

---

# 3. The Problem Without Proper Exception Handling

Imagine an API:

```
GET /users/100
```

Database lookup:

```
User with id 100 does not exist
```

Without proper handling:

Client receives:

```json
{
 "timestamp":"2026-06-30",
 "exception":"NullPointerException",
 "stackTrace":"...."
}
```

Problems:

- Internal information exposed
    
- Difficult for frontend developers
    
- Security risk
    
- Poor user experience
    

---

# 4. Why Raw Stack Trace Is Bad

Stack trace contains:

- Class names
    
- Database details
    
- Internal implementation
    
- File paths
    

Example:

Bad response:

```text
java.lang.NullPointerException
at com.company.user.UserService.java:45
```

A client does not need this.

Client needs:

```json
{
 "message":"User not found",
 "status":404
}
```

---

# 5. Simple Definition

Exception handling is:

```
The process of catching application failures and converting them into meaningful responses for clients without crashing the application.
```

Simple interview answer:

> Exception handling means managing errors in a controlled way so that the application can return proper responses instead of failing unexpectedly.

---

# 6. Real World Mental Model

Imagine a restaurant.

Flow:

```
Customer places order

        |

        v

Kitchen prepares food

        |

        v

Problem happens

        |

        v

Manager handles problem

        |

        v

Customer receives proper response
```

Mapping:

|Restaurant|Backend|
|---|---|
|Customer order|API request|
|Kitchen problem|Exception|
|Manager|Exception handler|
|Customer message|API response|

---

# 7. Backend Exception Flow

Normal flow:

```
Client

   |

   v

Controller

   |

   v

Service

   |

   v

Database
```

Failure flow:

```
Client

   |

   v

Controller

   |

   v

Service

   |

   v

Exception thrown

   |

   v

Global Exception Handler

   |

   v

Error Response
```

---

# 8. Spring MVC Internal Exception Flow

```
Client Request

        |

        v

DispatcherServlet

        |

        v

Controller

        |

        v

Exception

        |

        v

HandlerExceptionResolver

        |

        v

@ControllerAdvice

        |

        v

ResponseEntity

        |

        v

Client
```

---

# 9. Checked vs Unchecked Exceptions

Java has two major types.

---

# 9.1 Checked Exception

Compiler forces handling.

Example:

```java
try {

}
catch(IOException e){

}
```

Examples:

- IOException
    
- SQLException
    

Usually used for:

```
Recoverable external problems
```

---

# 9.2 Unchecked Exception

Runtime exceptions.

Example:

```java
throw new RuntimeException();
```

Examples:

- NullPointerException
    
- IllegalArgumentException
    
- Custom business exceptions
    

Spring applications commonly use:

```
RuntimeException
```

for business errors.

---

# 10. Why Spring Applications Prefer RuntimeException

Example:

Business rule:

```
User does not exist
```

This is not a programming error.

It is an application condition.

So we create:

```java
UserNotFoundException
```

instead of forcing every method to catch exceptions.

---

# 11. Custom Exceptions

Example:

```java
public class UserNotFoundException 
        extends RuntimeException {


    public UserNotFoundException(String message) {
        super(message);
    }

}
```

Usage:

```java
public User findUser(Long id){

    return repository.findById(id)
        .orElseThrow(
          () -> new UserNotFoundException(
              "User not found"
          )
        );
}
```

---

# 12. Why Create Custom Exceptions?

Bad:

```java
throw new RuntimeException();
```

Problem:

Nobody knows what happened.

Better:

```java
throw new UserNotFoundException();
```

Benefits:

- Clear meaning
    
- Easy handling
    
- Better API responses
    
- Cleaner code
    

---

# 13. @ExceptionHandler

Purpose:

Handles specific exceptions.

Example:

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<?> handleUserNotFound(
        UserNotFoundException ex){

    return ResponseEntity
        .status(404)
        .body(ex.getMessage());

}
```

Meaning:

```
If this exception occurs,
execute this method.
```

---

# 14. @ControllerAdvice

Problem:

If every controller has exception handling:

```
UserController

OrderController

PaymentController
```

Duplicate code.

Solution:

Global exception handler.

---

Example:

```java
@ControllerAdvice
public class GlobalExceptionHandler {


}
```

Now all controllers use it.

---

# 15. @RestControllerAdvice

Combination of:

```
@ControllerAdvice

+

@ResponseBody
```

Used for REST APIs.

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

---

# 16. Complete Global Handler Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler {


@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ApiError> 
handleUserNotFound(
UserNotFoundException ex){


ApiError error =
new ApiError(
404,
ex.getMessage()
);


return ResponseEntity
.status(404)
.body(error);

}

}
```

---

# 17. Creating Standard Error Response

Production APIs usually return a common structure.

Example:

```json
{
 "timestamp":"2026-06-30T10:20:00",
 "status":404,
 "error":"USER_NOT_FOUND",
 "message":"User does not exist",
 "path":"/users/100"
}
```

---

# 18. ResponseEntity

`ResponseEntity` represents complete HTTP response.

It contains:

- Status code
    
- Headers
    
- Body
    

Example:

```java
return ResponseEntity
.status(404)
.body(error);
```

Produces:

```
HTTP 404

Body:
error object
```

---

# 19. ProblemDetail (Spring Framework 6)

Spring 6 introduced:

```java
ProblemDetail
```

It follows RFC style error responses.

Example:

```java
ProblemDetail problem =
ProblemDetail.forStatus(404);

problem.setDetail(
"User not found"
);
```

Response:

```json
{
 "status":404,
 "detail":"User not found"
}
```

---

# 20. Important Spring Classes

## DispatcherServlet

Responsibility:

```
Front controller of Spring MVC
```

It receives requests and sends them to controllers.

---

## HandlerExceptionResolver

Responsibility:

```
Finds who can handle an exception
```

---

## ExceptionHandlerExceptionResolver

Handles:

```
@ExceptionHandler methods
```

---

## ResponseEntityExceptionHandler

Base class provided by Spring.

Useful for:

- Validation errors
    
- MVC exceptions
    
- Standard Spring errors
    

---

## ControllerAdvice

Global exception handling for controllers.

---

## RestControllerAdvice

REST API version of ControllerAdvice.

---

## ExceptionHandler

Maps exception type to handling method.

---

# 21. Request Lifecycle Example

Request:

```
GET /users/100
```

Flow:

## Step 1

Request reaches:

```
DispatcherServlet
```

---

## Step 2

Controller executes:

```java
userService.getUser(100);
```

---

## Step 3

Service:

```java
throw new UserNotFoundException();
```

---

## Step 4

Exception moves upward.

```
Service

↓

Controller

↓

Spring MVC
```

---

## Step 5

Spring searches handlers.

Finds:

```java
@ExceptionHandler(UserNotFoundException.class)
```

---

## Step 6

Creates response:

```json
{
 "message":"User not found"
}
```

---

## Step 7

Client receives:

```
HTTP 404
```

---

# 22. Business Exception vs Technical Exception

## Business Exception

Application rule failure.

Examples:

```
UserNotFoundException

InsufficientBalanceException

OrderAlreadyCancelledException
```

---

## Technical Exception

System failure.

Examples:

```
Database down

Network timeout

Server unavailable
```

---

# 23. Production Design Approach

Typical enterprise structure:

```
controller

service

exception

handler

response
```

Example:

```
exception/

 UserNotFoundException.java

handler/

 GlobalExceptionHandler.java

response/

 ApiError.java
```

---

# 24. Common Mistakes

## Mistake 1

Returning stack traces.

Wrong:

```java
return exception.toString();
```

---

## Mistake 2

Handling exceptions in every controller.

Problem:

Duplicate code.

---

## Mistake 3

Using generic exceptions everywhere.

Wrong:

```java
RuntimeException
```

Better:

```java
UserNotFoundException
```

---

# 25. Interview Questions

## Q1. Why do we need exception handling?

Answer:

> Because production applications can fail. Exception handling allows us to convert failures into meaningful and predictable responses instead of exposing internal errors.

---

## Q2. Why use @ControllerAdvice?

Answer:

> It provides centralized exception handling so all controllers follow the same error response format.

---

## Q3. Difference between @ExceptionHandler and @ControllerAdvice?

Answer:

> @ExceptionHandler handles a specific exception. @ControllerAdvice provides global exception handling across controllers.

---

## Q4. Why create custom exceptions?

Answer:

> Custom exceptions represent business scenarios clearly and make error handling easier.

---

# Final Mental Model

Remember:

```
Controller

calls

Service

something fails

Exception created

↓

Global Handler catches it

↓

API returns proper response
```

Spring Boot does not remove failures.

It helps us handle failures professionally.

Next Chapter:

[[Chapter-03-Identity-Principal-Credentials]]