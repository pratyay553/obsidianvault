---
tags: [microservices, springboot, java, interview, data]
category: Data and Transactions
questions: 34-41
status: to-revise
---

# Data and Transactions (Q34 to Q41)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q34. What is the Database per Service pattern? 🟢
> Each microservice owns its **private database**. No other service reads or writes it directly. They ask through APIs or events.
> **Why:** teams can change their schema freely, services stay independent, and each can use the best database type.
> **Cost:** no SQL JOINs across services and no single big transaction (see Saga pattern).
>
> 📖 **Simple words:**
> - **Schema** = the structure of a database (tables, columns).

> [!question]- Q35. What is Spring Data JPA? 🟢
> **JPA** is the Java standard to map objects to database tables. **Hibernate** is the common implementation. **Spring Data JPA** goes further: you write only an interface and Spring creates the code.
> Spring Boot also gives you **HikariCP**, a fast connection pool, by default.
>
> 📖 **Simple words:**
> - **ORM (Object-Relational Mapping)** = a tool that converts between Java objects and database rows.
> - **Connection pool** = a ready set of reusable DB connections, so you do not open a new one every time.
>
> ```java
> @Entity
> public class Product {
>     @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
>     private Long id;
>     private String name;
> }
>
> public interface ProductRepository extends JpaRepository<Product, Long> {
>     List<Product> findByNameContainingIgnoreCase(String name);  // query from method name
> }
> ```

> [!question]- Q36. What is the N+1 query problem and how do you fix it? 🟡
> You load 1 list of orders (1 query), then for each order Hibernate loads its items (N more queries). 100 orders = 101 queries. This slows the service badly.
> **Fix:** fetch the related data in one query using `JOIN FETCH` or `@EntityGraph`. Turn on SQL logging in development to spot it.
>
> ```java
> @Query("select o from Order o join fetch o.items")
> List<Order> findAllWithItems();
>
> // or
> @EntityGraph(attributePaths = "items")
> List<Order> findByStatus(String status);
> ```

> [!question]- Q37. How does @Transactional work and what are its common traps? 🟡
> A **transaction** means "all steps succeed, or none are saved". `@Transactional` starts one before the method and commits at the end (rolls back on unchecked exceptions).
>
> Traps:
> - It works through a **proxy**, so calling a `@Transactional` method from the **same class** (`this.method()`) does not start a transaction.
> - Only `public` methods through the Spring proxy count.
> - Do not call slow remote services inside a transaction; the DB connection stays locked.
> - Use `readOnly = true` for queries.
>
> 📖 **Simple words:**
> - **Proxy** = a wrapper object Spring puts around your bean to add behaviour like transactions.
> - **Rollback** = undo all changes made in the transaction.
>
> ```java
> @Transactional
> public void placeOrder(Order order) {
>     orderRepository.save(order);
>     outboxRepository.save(OutboxEvent.of(order));  // both saved, or both rolled back
> }
> ```

> [!question]- Q38. What is the difference between optimistic and pessimistic locking? 🔴
> Both stop two users from overwriting each other.
> - **Optimistic:** assume conflicts are rare. Add a `@Version` column; if the version changed when you save, you get an error and retry. Fast, no DB locks.
> - **Pessimistic:** lock the row while you work (`SELECT ... FOR UPDATE`). Safe for high-conflict cases, but slower.
>
> 📖 **Simple words:**
> - **Lock** = a mark that stops others from changing the same data at the same time.
>
> ```java
> @Entity
> class Account {
>     @Id Long id;
>     @Version Long version;      // optimistic locking
>     BigDecimal balance;
> }
>
> @Lock(LockModeType.PESSIMISTIC_WRITE)   // pessimistic locking
> @Query("select a from Account a where a.id = :id")
> Optional<Account> lockById(Long id);
> ```

> [!question]- Q39. Why use Flyway or Liquibase for database changes? 🟢
> They keep database changes in **versioned scripts** inside your project, so every environment (dev, test, prod) gets the same schema in the same order. Never rely on `spring.jpa.hibernate.ddl-auto=update` in production.
>
> 📖 **Simple words:**
> - **Migration** = a script that changes the database structure from one version to the next.
>
> ```sql
> -- src/main/resources/db/migration/V1__create_orders.sql
> CREATE TABLE orders (
>   id BIGSERIAL PRIMARY KEY,
>   status VARCHAR(20) NOT NULL,
>   created_at TIMESTAMP NOT NULL DEFAULT now()
> );
> ```

> [!question]- Q40. How do you choose between SQL and NoSQL? 🟡
> | Need | Good choice |
> |---|---|
> | Strong consistency, relations, transactions (orders, payments) | **SQL** (PostgreSQL, MySQL) |
> | Flexible documents, fast reads by key (catalog, profile) | **MongoDB** |
> | Caching, sessions, counters | **Redis** |
> | Full-text search | **Elasticsearch** |
> | Huge write volume, time series | **Cassandra** / time-series DB |
>
> Microservices allow **polyglot persistence**: each service picks the right store.
>
> 📖 **Simple words:**
> - **Polyglot persistence** = using different database types for different jobs.

> [!question]- Q41. How do you get data that lives in other services (no JOINs)? 🔴
> Two common ways:
> 1. **API composition:** one service (or the gateway) calls several services and merges the results. Simple, but slower and depends on all of them being up.
> 2. **CQRS read model:** listen to events from other services and keep a **local copy** of the data you need, built for fast reading. Faster and more independent, but data is eventually consistent.
>
> 📖 **Simple words:**
> - **CQRS** = Command Query Responsibility Segregation: separate models for writing and reading.

---
Previous: [[04 - REST API Design]] | Next: [[06 - Service Communication and Messaging]]
