---
tags: [configuration, spring-core, interview]
---

# Configuration Classes Overview

Related: [[01 - Spring Scheduling Basics]] · [[08 - Spring Cloud and Microservices]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
`@Configuration` classes are where you tell Spring **how to build the objects (beans)** your app needs — schedulers, HTTP clients, mail senders, cloud storage clients — instead of `new`-ing them up manually everywhere.

## 🔑 Core Annotations

| Annotation | Purpose |
|---|---|
| `@Configuration` | Marks a class as a source of bean definitions |
| `@Bean` | Marks a method whose return value Spring should manage as a bean |
| `@Qualifier` | Disambiguates *which* bean to inject when multiple beans of the same type exist |
| `@Value` | Injects a single property value (e.g., `@Value("${app.timeout}")`) |
| `@ConfigurationProperties` | Binds a whole group of properties to a strongly-typed class |
| `@Profile` | Activates a bean/config only for a given environment (dev/test/prod) |

## 🛠️ Example: Why `@Qualifier` matters

```java
@Configuration
public class HttpClientConfig {

    @Bean
    @Qualifier("hindalcoRestTemplate")
    public RestTemplate hindalcoRestTemplate() {
        return new RestTemplateBuilder()
                .setConnectTimeout(Duration.ofSeconds(5))
                .build();
    }

    @Bean
    @Qualifier("odtsRestTemplate")
    public RestTemplate odtsRestTemplate() {
        return new RestTemplateBuilder()
                .setConnectTimeout(Duration.ofSeconds(30))
                .build();
    }
}
```
Without `@Qualifier`, Spring can't tell which `RestTemplate` to inject into `HindalcoClient` vs `ODTSClient` — you'd get a `NoUniqueBeanDefinitionException`.

## 🗂️ The Config Classes in This Module — What Each One Actually Does

| Class | Responsibility |
|---|---|
| `SchedulerConfig` | Wires the custom `TaskScheduler` (virtual threads) — see [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] |
| `HttpClientConfig` | Builds `RestTemplate`/`WebClient`/`HttpClient5` beans for outbound API calls |
| `MailConfig` | Sets up `JavaMailSender` or Microsoft 365 (`MSAL4J`/EWS) beans — see [[11 - Email Services]] |
| `WSConfig` | Configures `WebServiceTemplate` for SOAP calls — see [[07 - REST and SOAP Clients]] |
| `SerializerConfig` | Customizes Jackson `ObjectMapper` / Gson / JAXB — see [[17 - Serialization - Jackson Gson JAXB]] |
| `HowenVssProperties` | A `@ConfigurationProperties`-style class binding a custom `application.yml` section |
| `TSLGoogleCloudConfig` | Builds the GCS `Storage` client bean — see [[12 - File Processing]] |

## 🧵 `@ConfigurationProperties` example (type-safe config)

```java
@ConfigurationProperties(prefix = "howen.vss")
public record HowenVssProperties(String endpoint, String apiKey, int timeoutMs) {}
```
```yaml
howen:
  vss:
    endpoint: https://vss.example.com
    api-key: ${VSS_API_KEY}
    timeout-ms: 5000
```
This is preferred over scattering `@Value("${...}")` everywhere because it's **type-safe, testable, and IDE-autocompletable**.

## ❓ Interview Q&A

**Q: `@Component` vs `@Bean` — when do you use which?**
A: `@Component` (and its specializations `@Service`, `@Repository`, `@Controller`) is a class-level annotation for classes *you own* — Spring auto-detects them via component scanning. `@Bean` is a method-level annotation inside a `@Configuration` class, typically used for objects you **don't own the source of** (third-party classes like `RestTemplate`, `ObjectMapper`) or that need custom construction logic.

**Q: `@Value` vs `@ConfigurationProperties`?**
A: `@Value` injects one property at a time and is fine for a couple of values; `@ConfigurationProperties` binds a whole nested block of YAML to a POJO/record — better for maintainability and validation with many related settings.

**Q: What does `@Qualifier` solve?**
A: Ambiguity when multiple beans share the same type — it tells Spring exactly which bean to autowire by name.

**Q: How do you make a bean environment-specific (dev vs prod)?**
A: `@Profile("prod")` on the `@Bean` method or `@Configuration` class, combined with `spring.profiles.active=prod`.
