---
tags: [microservices, springboot, java, interview, springboot]
category: Spring Boot Core
questions: 17-25
status: to-revise
---

# Spring Boot Core (Q17 to Q25)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q17. What is Spring Boot and why do we use it? 🟢
> Spring Boot is a tool built on the Spring Framework that removes boring setup. It gives you:
> - **Auto-configuration** (sensible defaults)
> - **Starter dependencies** (one line adds many libraries)
> - **Embedded server** (Tomcat inside your JAR)
> - **Production features** (Actuator: health, metrics)
>
> You write business code and run `java -jar app.jar`.
>
> 📖 **Simple words:**
> - **Embedded server** = the web server is packaged inside your app, so you do not install it separately.

> [!question]- Q18. What does @SpringBootApplication do? 🟢
> It combines three annotations:
> - `@Configuration`: this class can define beans.
> - `@EnableAutoConfiguration`: turn on automatic setup based on your dependencies.
> - `@ComponentScan`: find your `@Component`, `@Service` etc. in this package and below.
>
> ```java
> @SpringBootApplication
> public class OrderServiceApplication {
>     public static void main(String[] args) {
>         SpringApplication.run(OrderServiceApplication.class, args);
>     }
> }
> ```

> [!question]- Q19. What are auto-configuration and starters? 🟡
> **Auto-configuration** checks your classpath and properties and creates beans for you. If `spring-boot-starter-data-jpa` and a database driver are present, Spring Boot creates the `DataSource` and `EntityManager` automatically.
> A **starter** is one dependency that brings a group of compatible libraries (for example `spring-boot-starter-web`).
> You can override any default by defining your own bean. Run with `--debug` to see what was configured.
>
> 📖 **Simple words:**
> - **Classpath** = the list of libraries (JARs) your app can use.
> - **Bean** = an object created and managed by Spring.

> [!question]- Q20. What are IoC and Dependency Injection? Why prefer constructor injection? 🟢
> **IoC (Inversion of Control):** you do not create objects with `new`; Spring creates and connects them.
> **DI (Dependency Injection):** Spring gives a class the objects it needs.
>
> Prefer constructor injection because fields can be `final`, the class is easy to test (just pass mocks), and missing dependencies fail at startup, not at runtime.
>
> ```java
> @Service
> public class OrderService {
>     private final OrderRepository repository;   // final = safe
>
>     public OrderService(OrderRepository repository) {   // Spring injects it
>         this.repository = repository;
>     }
> }
> ```

> [!question]- Q21. What is the difference between @Component, @Service, @Repository and @Controller? 🟢
> All of them create a Spring bean. The difference is the **role** (and a few extras).
>
> | Annotation | Use for |
> |---|---|
> | `@Component` | Any general bean |
> | `@Service` | Business logic |
> | `@Repository` | Data access (also translates DB exceptions) |
> | `@Controller` / `@RestController` | Web layer (REST endpoints) |
> | `@Configuration` + `@Bean` | Manual bean creation |

> [!question]- Q22. What are bean scopes? 🟡
> The scope tells Spring how many instances to create.
> - **singleton** (default): one shared instance for the whole app.
> - **prototype**: a new instance every time it is requested.
> - **request / session**: one per HTTP request or session (web apps).
>
> Because singleton beans are shared by many threads, keep them **stateless** (do not store user data in fields).
>
> 📖 **Simple words:**
> - **Stateless** = does not remember anything between calls.

> [!question]- Q23. How do you handle configuration and profiles (dev, test, prod)? 🟢
> Put settings in `application.yml`. Create `application-dev.yml`, `application-prod.yml` for environment-specific values. Choose the profile at startup.
>
> Priority (high to low): command-line arguments, environment variables, profile file, default file. So production can override values using environment variables without changing the JAR.
>
> 📖 **Simple words:**
> - **Profile** = a named set of settings for one environment.
>
> ```yaml
> # application.yml
> spring.application.name: order-service
> ---
> spring:
>   config.activate.on-profile: prod
> server.port: 8080
> ```
> ```bash
> SPRING_PROFILES_ACTIVE=prod java -jar app.jar
> ```

> [!question]- Q24. What is the difference between @Value and @ConfigurationProperties? 🟡
> - `@Value("${key}")` reads **one** value. Quick, but scattered and not type-safe for groups.
> - `@ConfigurationProperties` binds a **group** of settings to a class. It supports validation and is easier to test and maintain. Prefer it for microservices.
>
> ```java
> @ConfigurationProperties(prefix = "payment")
> @Validated
> public record PaymentProperties(@NotBlank String baseUrl, @DefaultValue("2s") Duration timeout) {}
> ```

> [!question]- Q25. What is Spring Boot Actuator? 🟢
> Actuator adds ready-made endpoints to watch your app in production: `/actuator/health`, `/actuator/metrics`, `/actuator/prometheus`, `/actuator/info`.
> Security tip: expose only what you need and protect sensitive endpoints. Never expose all of them to the public internet.
>
> 📖 **Simple words:**
> - **Endpoint** = a URL where you can ask the app for information.
>
> ```yaml
> management:
>   endpoints:
>     web:
>       exposure:
>         include: health,info,prometheus
> ```

---
Previous: [[02 - Java Essentials for Microservices]] | Next: [[04 - REST API Design]]
