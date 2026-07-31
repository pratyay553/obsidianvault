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

# 📌 Configuration & Properties

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Boot's externalized configuration system lets you bind environment-specific values (from `application.yml`, environment variables, or secrets) into type-safe Java records/classes via `@ConfigurationProperties`, keeping your code free of magic strings.

## 🧱 Key Concepts & Mechanics

### Property Source Priority (highest → lowest)
1. Command-line arguments (`--server.port=9090`)
2. `SPRING_*` environment variables / OS env vars
3. `application-{profile}.yml` (active profile)
4. `application.yml`
5. `@PropertySource` annotations
6. Default values in `@Value`

### Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@Value("${prop.key}")` | Inject single property; brittle for large configs |
| `@ConfigurationProperties(prefix = "app")` | Bind a property subtree to a typed class |
| `@EnableConfigurationProperties` | Register `@ConfigurationProperties` beans |
| `@Profile("prod")` | Activate bean only on a specific profile |
| `@PropertySource("classpath:custom.properties")` | Add extra property file |

### Profiles
- Active profile set via `spring.profiles.active=prod` or `SPRING_PROFILES_ACTIVE=prod`
- `application-prod.yml` overrides `application.yml` for active profile keys
- `@Profile("!prod")` — bean active when profile is NOT prod

### `@ConfigurationProperties` vs `@Value`
- `@ConfigurationProperties` — type-safe, validated, IDE autocomplete with `spring-boot-configuration-processor`
- `@Value` — simple scalar injection; no validation; breaks if property missing without a default

## 💻 Production Code Example

```java
// application.yml
// app:
//   api:
//     base-url: https://api.stripe.com
//     timeout-ms: 5000
//     max-retries: 3
//   feature-flags:
//     new-checkout: true

// Type-safe binding with Java record (Spring Boot 3.x)
@ConfigurationProperties(prefix = "app.api")
@Validated  // enables Jakarta Bean Validation on properties
public record ApiProperties(
    @NotBlank String baseUrl,
    @Min(100) @Max(30_000) int timeoutMs,
    @Min(0) @Max(10) int maxRetries
) {}

@ConfigurationProperties(prefix = "app.feature-flags")
public record FeatureFlags(boolean newCheckout) {}

// Register both (or use @EnableConfigurationProperties on a @Configuration)
@SpringBootApplication
@EnableConfigurationProperties({ ApiProperties.class, FeatureFlags.class })
public class Application { ... }

// Consume in a service
@Service
public class StripeClient {

    private final ApiProperties config;
    private final RestTemplate http;

    public StripeClient(ApiProperties config, RestTemplateBuilder builder) {
        this.config = config;
        this.http = builder
            .rootUri(config.baseUrl())
            .connectTimeout(Duration.ofMillis(config.timeoutMs()))
            .build();
    }
}

// Profile-specific bean
@Configuration
public class StorageConfig {

    @Bean
    @Profile("prod")
    public StorageService s3Storage(S3Properties props) {
        return new S3StorageService(props);
    }

    @Bean
    @Profile("!prod")
    public StorageService localStorage() {
        return new LocalFileStorageService(Path.of("/tmp/uploads"));
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using `@Value` for groups of related config (5+ properties from the same prefix).
  **Fix:** Bind with `@ConfigurationProperties` — typed, validated, and documented by the configuration processor.

- **Mistake:** Hardcoding secrets in `application.yml` and committing to git.
  **Fix:** Use environment variables, Vault, or a secrets manager. Reference via `${MY_SECRET}` in YAML.

- **Mistake:** Forgetting `@EnableConfigurationProperties` or `@Component` on a `@ConfigurationProperties` class — Spring doesn't register it.
  **Fix:** Add `@EnableConfigurationProperties(MyProps.class)` to a config class or annotate the props class with `@Component`.

- **Mistake:** Hyphen vs camelCase confusion — `app.base-url` in YAML must map to `baseUrl` in Java (relaxed binding handles this automatically).

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@Value` and `@ConfigurationProperties`?
   - **A:** `@Value` injects a single property by SpEL expression — simple but brittle, no validation, no IDE support. `@ConfigurationProperties` binds an entire property namespace to a typed class, supports Jakarta Validation, and works with the configuration processor for autocomplete.

2. **Q:** How does Spring Boot's property override order work?
   - **A:** Command-line args > environment variables > profile-specific YAML > base YAML > `@PropertySource`. Each level overrides the one below it.

3. **Q:** How do you validate configuration properties at startup?
   - **A:** Add `@Validated` to the `@ConfigurationProperties` class and annotate fields with Jakarta constraints (`@NotBlank`, `@Min`, etc.). Spring will fail fast at startup if constraints are violated.

4. **Q:** How do you activate a Spring profile?
   - **A:** `spring.profiles.active=prod` in `application.yml`, the `SPRING_PROFILES_ACTIVE=prod` env var, or `--spring.profiles.active=prod` as a JVM argument.

## 🔗 Related Notes
- [[IoC-Container-And-DI]]
- [[Component-Scanning-And-Stereotypes]]
- [[Docker-Kubernetes-Deployment]]
