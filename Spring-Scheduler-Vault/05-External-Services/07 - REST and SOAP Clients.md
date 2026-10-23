---
tags: [integration, rest, soap, interview]
---

# REST and SOAP Clients

Related: [[02 - Configuration Classes Overview|03 - Configuration Classes Overview]] · [[05 - Retry Pattern]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
This module talks to many external systems (`HindalcoClient`, `ODTSClient`, `OmnicommClient`, `FarEyeClient`, `AmnexClient`, `LindeClient`, `NovireClient`, `FuelPriceClient`) — each one wraps calls to a different partner's REST or SOAP API.

## 🔀 REST Client Options

| Client | Style | Notes |
|---|---|---|
| `RestTemplate` | Synchronous, blocking | The classic choice; **in maintenance mode** since Spring 5 (still widely used in existing code) |
| `WebClient` | Reactive, non-blocking | Part of Spring WebFlux; can be used synchronously too via `.block()` |
| `@RestClient` (Spring Boot 3.2+) | Declarative, synchronous | New interface-based client — write an interface, Spring generates the implementation |
| `HttpClient5` (Apache) | Low-level HTTP client | Used underneath `RestTemplate`/`WebClient` for fine-grained control (connection pooling, timeouts) |

```java
// RestTemplate style
public class HindalcoClient {
    private final RestTemplate restTemplate;

    public TripData fetchTrip(String tripId) {
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<TripData> response = restTemplate.exchange(
            "/api/trips/{id}", HttpMethod.GET, entity, TripData.class, tripId);
        return response.getBody();
    }
}
```

```java
// Declarative @RestClient (Spring Boot 3.2+) style
@RestClient(baseUrl = "https://api.fareye.com")
public interface FarEyeClient {
    @PostExchange("/trips")
    TripResponse uploadTrip(@RequestBody TripRequest request);
}
```

## 🧾 SOAP Client

```java
@Configuration
public class WSConfig {
    @Bean
    public WebServiceTemplate webServiceTemplate(Jaxb2Marshaller marshaller) {
        WebServiceTemplate template = new WebServiceTemplate(marshaller);
        template.setDefaultUri("https://partner.example.com/soap-endpoint");
        return template;
    }
}
```
`WebServiceTemplate` sends/receives XML payloads, usually marshalled/unmarshalled via **JAXB** (see [[17 - Serialization - Jackson Gson JAXB]]). `@WebServiceClient` configures the client-side binding info.

## ⚠️ Reliability Concerns (ties into other notes)
- External calls can fail transiently → wrap with the [[05 - Retry Pattern]].
- Every outbound call is a good candidate for `@Around` logging via [[04 - AOP - Aspect Oriented Programming]] (log latency, success/failure).
- Timeouts **must** be set explicitly (`RestTemplateBuilder.setConnectTimeout/setReadTimeout`) — an unbounded HTTP call inside a scheduled job can hang a whole run.

## ❓ Interview Q&A

**Q: Why is `RestTemplate` "in maintenance mode," and what's the alternative?**
A: Spring hasn't added new features to `RestTemplate` since Spring 5, recommending `WebClient` (reactive) or, more recently, the declarative `@RestClient` (Spring Boot 3.2+) for new code. `RestTemplate` remains fully supported for existing applications, just not actively enhanced.

**Q: Can you use `WebClient` in a non-reactive app?**
A: Yes — call `.block()` on the resulting `Mono`/`Flux` to get a synchronous result, though that partially defeats its non-blocking purpose. It's still popular purely for its cleaner, fluent API.

**Q: REST vs SOAP — why would a partner integration use SOAP in 2024+?**
A: Legacy enterprise systems (often in logistics, government, or specific industries in this module's domain like fleet/geofence data) still expose SOAP/WSDL-based APIs; you can't change the partner's API, so you must speak their protocol.

**Q: What's the risk of not setting a connect/read timeout on `RestTemplate`?**
A: The default is **no timeout** — a hung/slow partner API can block the calling thread indefinitely, potentially stalling an entire scheduled job (or, without proper thread isolation, other jobs sharing the scheduler).
