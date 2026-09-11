---
tags: [building-block]
---

# Repository

## 🧠 Problem
`BookingService` needs to save and fetch `Booking`s, but I don't want it writing raw SQL or knowing about tables/columns.

## 🤔 Why does the problem exist
If persistence code is scattered inline through business logic, switching databases or adding caching means hunting through business logic for every SQL statement — and testing business logic requires a real DB.

## 💡 Idea
Provide a collection-like interface (`save`, `findById`, `findByCustomer`) that hides the actual storage mechanism behind it. Business logic talks to the repository interface, never the DB directly.

## 🏗️ Structure
```
BookingService → BookingRepository (interface)
                        ↑ implements
                 JpaBookingRepository / InMemoryBookingRepository
```

## ☕ Java Implementation
```java
interface BookingRepository {
    void save(Booking b);
    Optional<Booking> findById(UUID id);
    List<Booking> findByCustomer(UUID customerId);
}

class JpaBookingRepository implements BookingRepository {
    // actual JPA/SQL logic lives only here
}
```

## 🏢 SAloon Example
`BookingRepository` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. Tests use an `InMemoryBookingRepository` (a `HashMap` under the hood) so `BookingService` tests run with zero DB setup.

## 🎯 When should I use it?
Any class doing persistence — this is basically mandatory in any LLD design with a database, and directly implements [[../02 - SOLID/04 - DIP|DIP]].

## ⚠️ When NOT to use it?
For a throwaway script/prototype with no tests and no plan to swap storage, a repository layer is unnecessary ceremony.

## 🔥 Interview Question
*"Isn't Repository just the [[../05 - Design Patterns/Structural/Adapter|Adapter]] pattern for databases?"*
→ Similar shape (interface + concrete implementation), but Repository's intent is specifically modeling a *collection-like* interface for a domain object, not translating an external API's shape.

## 🔗 Related
[[../02 - SOLID/04 - DIP]]
[[02 - Service]]

#building-block
