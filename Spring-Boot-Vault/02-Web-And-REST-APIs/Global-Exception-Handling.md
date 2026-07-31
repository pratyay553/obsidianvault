---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Intermediate
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Global Exception Handling

## 💡 1-Sentence Elevator Pitch (For Interviews)
> `@RestControllerAdvice` provides a single, centralized place to catch exceptions thrown anywhere in the request pipeline and translate them into consistent, structured HTTP error responses — eliminating try/catch blocks from every controller.

## 🧱 Key Concepts & Mechanics

### The Mechanism
- `@ControllerAdvice` / `@RestControllerAdvice` — beans that intercept exceptions thrown by any `@Controller`
- `@ExceptionHandler(ExceptionType.class)` — maps a specific exception class to a handler method
- Spring picks the most specific matching handler when exceptions form a hierarchy

### RFC 7807 — Problem Details (Spring Boot 3.x)
- Spring Boot 3 ships `ProblemDetail` as a first-class error response format
- Enable globally: `spring.mvc.problemdetails.enabled=true` in `application.yml`
- Fields: `type` (URI), `title`, `status`, `detail`, `instance`, + custom extensions via `setProperty()`

### Error Response Strategy
| Situation | Status | Handler |
|-----------|--------|---------|
| Validation failure | 400 | `MethodArgumentNotValidException` |
| Resource not found | 404 | Custom `ResourceNotFoundException` |
| Business rule violation | 409 / 422 | Custom domain exceptions |
| Auth token invalid | 401 | Security filter (not ControllerAdvice) |
| Unexpected error | 500 | `Exception` catch-all |

### Exception Hierarchy Design
- Define base `AppException(HttpStatus status, String code, String detail)`
- Domain exceptions extend it: `ResourceNotFoundException`, `ConflictException`, `BusinessRuleException`
- One handler in `@RestControllerAdvice` catches `AppException` and maps to `ProblemDetail`

## 💻 Production Code Example

```java
// ── Domain exception hierarchy ────────────────────────────────────────
public class AppException extends RuntimeException {
    private final HttpStatus status;
    private final String errorCode;

    public AppException(HttpStatus status, String errorCode, String message) {
        super(message);
        this.status = status;
        this.errorCode = errorCode;
    }
    // getters...
}

public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resource, Object id) {
        super(HttpStatus.NOT_FOUND, "RESOURCE_NOT_FOUND",
              resource + " not found: " + id);
    }
}

public class ConflictException extends AppException {
    public ConflictException(String message) {
        super(HttpStatus.CONFLICT, "CONFLICT", message);
    }
}

// ── Global handler ────────────────────────────────────────────────────
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // Catch all domain exceptions — single handler for the whole hierarchy
    @ExceptionHandler(AppException.class)
    public ProblemDetail handleAppException(AppException ex,
                                             HttpServletRequest req) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            ex.getStatus(), ex.getMessage());
        problem.setTitle(ex.getStatus().getReasonPhrase());
        problem.setInstance(URI.create(req.getRequestURI()));
        problem.setProperty("errorCode", ex.getErrorCode());
        return problem;
    }

    // Jakarta validation failures
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex,
                                           HttpServletRequest req) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Validation Failed");
        problem.setInstance(URI.create(req.getRequestURI()));
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(fe -> errors.put(fe.getField(), fe.getDefaultMessage()));
        problem.setProperty("errors", errors);
        return problem;
    }

    // Service-level constraint violations
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleConstraintViolation(ConstraintViolationException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Constraint Violation");
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getConstraintViolations()
          .forEach(cv -> errors.put(cv.getPropertyPath().toString(),
                                    cv.getMessage()));
        problem.setProperty("errors", errors);
        return problem;
    }

    // Unexpected catch-all — never expose stack trace
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ProblemDetail handleUnexpected(Exception ex, HttpServletRequest req) {
        log.error("Unhandled exception on {}", req.getRequestURI(), ex);
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        problem.setTitle("Internal Server Error");
        problem.setDetail("An unexpected error occurred. Please try again later.");
        return problem;
    }
}

// ── Usage in service ──────────────────────────────────────────────────
@Service
public class UserService {
    public UserDto findById(UUID id) {
        return repo.findById(id)
            .map(mapper::toDto)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Returning `200 OK` with `{ "success": false, "error": "..." }` error bodies.
  **Fix:** Use proper HTTP status codes + RFC 7807 `ProblemDetail` for a machine-readable, standardized format.

- **Mistake:** Exposing stack traces in error responses (e.g., default Spring Whitelabel error page in prod).
  **Fix:** Catch `Exception` in the advice and return a generic message; log the full trace server-side.

- **Mistake:** Security-related exceptions (401/403) not being caught by `@RestControllerAdvice`.
  **Fix:** Spring Security exceptions are thrown before the dispatcher servlet reaches the advice. Handle them in `AuthenticationEntryPoint` and `AccessDeniedHandler`.

- **Mistake:** Duplicating exception handling logic across multiple controllers.
  **Fix:** One global `@RestControllerAdvice` handles all exceptions; controllers stay clean.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is `@RestControllerAdvice` and how does it differ from `@ControllerAdvice`?
   - **A:** `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`. Handler methods return objects serialized to JSON instead of view names. Use `@RestControllerAdvice` for REST APIs.

2. **Q:** Why doesn't `@RestControllerAdvice` catch Spring Security exceptions?
   - **A:** Security exceptions are thrown in servlet filters before the request reaches the Spring MVC dispatcher, so the exception handler chain is never invoked. Handle them via `HttpSecurity.exceptionHandling().authenticationEntryPoint(...)` and `.accessDeniedHandler(...)`.

3. **Q:** What is RFC 7807 and why use it?
   - **A:** RFC 7807 (Problem Details for HTTP APIs) defines a standard JSON structure for error responses: `type`, `title`, `status`, `detail`, `instance`. It gives clients a machine-readable, consistent format to parse errors across different APIs.

4. **Q:** How do you prioritize between two `@ExceptionHandler` methods that could both match?
   - **A:** Spring picks the most specific (closest in class hierarchy) match. If both are equally specific, the one declared first in the class wins.

## 🔗 Related Notes
- [[Request-Validation-Jakarta]]
- [[Spring-Security-Architecture-Filters]]
- [[RESTful-API-Design]]
