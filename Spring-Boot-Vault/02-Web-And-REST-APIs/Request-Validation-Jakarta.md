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

# 📌 Request Validation (Jakarta Bean Validation)

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Jakarta Bean Validation lets you declare constraints directly on DTO fields with annotations like `@NotBlank` and `@Size`, and Spring MVC enforces them automatically when you add `@Valid` to a controller parameter — no manual null-checking required.

## 🧱 Key Concepts & Mechanics

### Common Constraint Annotations

| Annotation | Applies To | Description |
|------------|-----------|-------------|
| `@NotNull` | Any | Must not be null |
| `@NotBlank` | String | Not null, not empty, not whitespace |
| `@NotEmpty` | String, Collection | Not null and not empty |
| `@Size(min, max)` | String, Collection | Length/size within range |
| `@Min(value)` / `@Max(value)` | Number | Numeric range |
| `@Email` | String | Valid email format |
| `@Pattern(regexp)` | String | Matches regex |
| `@Positive` / `@PositiveOrZero` | Number | Sign constraint |
| `@Valid` | Object field | Cascades validation to nested object |
| `@Validated` | Class/method | Spring extension — enables group validation |

### Validation Trigger Points
- `@Valid` on `@RequestBody` — validates inbound JSON DTOs
- `@Validated` on `@Service` or `@Repository` — method-level validation
- `@ConfigurationProperties` + `@Validated` — validates config at startup

### Validation Groups
- Define marker interfaces as groups
- Use `@Validated(CreateGroup.class)` vs `@Validated(UpdateGroup.class)` to apply different rules per operation

### Custom Validators
1. Create `@Constraint` annotation
2. Implement `ConstraintValidator<MyAnnotation, TargetType>`

## 💻 Production Code Example

```java
// ── DTO with constraints ──────────────────────────────────────────────
public record CreateUserRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be 3–50 characters")
    String username,

    @NotBlank @Email(message = "Must be a valid email address")
    String email,

    @NotBlank
    @Pattern(regexp = "^(?=.*[A-Z])(?=.*\\d).{8,}$",
             message = "Password must be 8+ chars with uppercase and digit")
    String password,

    @Valid                          // cascades validation into nested object
    @NotNull
    AddressRequest address
) {}

public record AddressRequest(
    @NotBlank String street,
    @NotBlank String city,
    @Pattern(regexp = "\\d{5}(-\\d{4})?") String postalCode
) {}

// ── Controller ────────────────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserDto create(@Valid @RequestBody CreateUserRequest req) {
        // req is guaranteed valid here
        return userService.create(req);
    }
}

// ── Custom validator ──────────────────────────────────────────────────
@Target({ FIELD, PARAMETER })
@Retention(RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already registered";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component
public class UniqueEmailValidator
        implements ConstraintValidator<UniqueEmail, String> {

    private final UserRepository repo;

    public UniqueEmailValidator(UserRepository repo) {
        this.repo = repo;
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext ctx) {
        if (email == null) return true; // defer to @NotBlank
        return !repo.existsByEmail(email);
    }
}

// ── Global error handler (see also Global-Exception-Handling.md) ──────
@RestControllerAdvice
public class ValidationErrorHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Validation Failed");
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(fe -> errors.put(fe.getField(), fe.getDefaultMessage()));
        problem.setProperty("errors", errors);
        return problem;
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `@NotEmpty` on a String to disallow whitespace-only values.
  **Fix:** `@NotEmpty` allows `"   "`. Use `@NotBlank` for strings.

- **Mistake:** Forgetting `@Valid` on nested objects — child constraints are silently skipped.
  **Fix:** Annotate nested DTO fields with both `@Valid` and `@NotNull`.

- **Mistake:** Writing custom validation logic in the controller before calling the service.
  **Fix:** Express constraints declaratively on DTOs; let `@Valid` enforce them.

- **Mistake:** Database calls inside a `ConstraintValidator` without understanding transaction scope.
  **Fix:** Validators run outside service transactions by default. Mark the validator method with `@Transactional(readOnly = true)` if it queries the DB.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`?
   - **A:** `@NotNull` — not null. `@NotEmpty` — not null AND not empty string/collection. `@NotBlank` — not null, not empty, and not whitespace only. For strings, `@NotBlank` is almost always what you want.

2. **Q:** How do you return structured validation errors from a Spring Boot API?
   - **A:** Handle `MethodArgumentNotValidException` in a `@RestControllerAdvice`. Iterate `ex.getBindingResult().getFieldErrors()` to build a map of field → message, and return it with a 400 status (ideally as RFC 7807 `ProblemDetail`).

3. **Q:** How do you write a custom constraint validator?
   - **A:** Create a `@Constraint` annotation pointing to a `ConstraintValidator<A, T>` implementation. The `isValid()` method returns `false` when the constraint is violated. Register the validator as a `@Component` so it can inject Spring beans.

4. **Q:** How do you validate method parameters in a `@Service` (not just `@RequestBody`)?
   - **A:** Annotate the class with `@Validated` and use constraint annotations on method parameters. Spring AOP intercepts calls and throws `ConstraintViolationException` on failure.

## 🔗 Related Notes
- [[Controllers-And-Request-Mapping]]
- [[Global-Exception-Handling]]
- [[RESTful-API-Design]]
