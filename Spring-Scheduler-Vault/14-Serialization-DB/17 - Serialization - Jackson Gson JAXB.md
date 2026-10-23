---
tags: [serialization, json, xml, interview]
---

# Serialization — Jackson, Gson, JAXB

Related: [[07 - REST and SOAP Clients]] · [[02 - Configuration Classes Overview|03 - Configuration Classes Overview]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Every external call means converting Java objects ↔ text (JSON or XML). Different partners speak different formats, so this module keeps multiple serializers on hand.

## 🔑 The Three Libraries

| Library | Format | Notes |
|---|---|---|
| **Jackson** (`jackson-databind`, `jackson-core`) | JSON | Spring Boot's **default** JSON library — auto-configured, used for all `@RestController`/`RestTemplate`/`WebClient` JSON (de)serialization out of the box |
| **Gson** | JSON | Google's alternative — sometimes needed because a specific partner SDK depends on it, or for its more lenient/flexible parsing style |
| **JAXB** (`jakarta.xml.bind`) | XML | Used specifically for **SOAP** clients (see [[07 - REST and SOAP Clients]]) — marshals Java objects to/from XML |

## 🛠️ Jackson Basics

```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());       // support LocalDateTime, etc.
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

String json = mapper.writeValueAsString(trip);      // object -> JSON
Trip trip = mapper.readValue(json, Trip.class);      // JSON -> object
```

```java
@Configuration
public class SerializerConfig {
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }
}
```
`FAIL_ON_UNKNOWN_PROPERTIES = false` is a very common setting for integration code — a partner API might add a new JSON field tomorrow, and you don't want deserialization to break just because your DTO doesn't know about it yet.

## 🛠️ JAXB for SOAP/XML

```java
@XmlRootElement(name = "TripRequest")
public class TripRequest {
    @XmlElement private String tripId;
    @XmlElement private String status;
}

JAXBContext context = JAXBContext.newInstance(TripRequest.class);
Marshaller marshaller = context.createMarshaller();
marshaller.marshal(tripRequest, System.out); // object -> XML
```
Spring's `Jaxb2Marshaller` bean plugs this into `WebServiceTemplate` automatically (see [[07 - REST and SOAP Clients]]).

## ❓ Interview Q&A

**Q: Why would a Spring Boot app use Gson at all, given Jackson is the default?**
A: Usually because a specific third-party SDK/dependency (one of the partner clients like `OmnicommClient` or `NovireClient`) is built against Gson internally, or a developer prefers Gson's more forgiving handling of malformed/partial JSON for a specific integration — not a general recommendation, just a pragmatic per-integration choice.

**Q: What's the risk of leaving `FAIL_ON_UNKNOWN_PROPERTIES` at its Jackson default (`true`)?**
A: Any time an external partner adds a new field to their JSON response, your deserialization throws an exception and the integration breaks — even though you don't even care about that new field. Disabling it makes your client more tolerant of upstream API evolution.

**Q: Marshalling vs unmarshalling — which direction is which?**
A: **Marshalling** = Java object → XML/JSON (going out). **Unmarshalling** = XML/JSON → Java object (coming in). Same "marshal/unmarshal" terminology JAXB uses; Jackson calls the JSON equivalents "serialize/deserialize."

**Q: How do you handle a `LocalDateTime` field with Jackson without it breaking?**
A: Register the `JavaTimeModule` (`jackson-datatype-jsr310`) on your `ObjectMapper` — without it, Jackson doesn't know how to (de)serialize Java 8+ date/time types and either throws or produces an ugly array-based format.
