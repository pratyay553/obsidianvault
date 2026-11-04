---
tags: [microservices, springboot, java, interview, architecture]
category: Microservices Basics
questions: 1-8
status: to-revise
---

# Microservices Basics (Q1 to Q8)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q1. What is a microservice? 🟢
> A microservice is a small application that does **one business job**, for example *Payments* or *Orders*. It has its own code, its own database, and is deployed on its own. Other services talk to it through an **API**.
> Think of a food court: every shop cooks one type of food. If one shop closes, the others still work.
>
> 📖 **Simple words:**
> - **API** = a set of rules (usually URLs) that lets one program talk to another program.

> [!question]- Q2. What is the difference between a monolith and microservices? 🟢
> - **Monolith:** the whole app is one deployable unit (one JAR). Simple to build and test at the start.
> - **Microservices:** the app is split into many small units. Each one can be built, deployed and scaled alone.
>
> Rule of thumb: start simple. Move to microservices when the team and the app grow and one big deployment becomes painful.
>
> 📖 **Simple words:**
> - **Deploy** = put the application on a server so users can use it.
> - **Scale** = run more copies (or bigger machines) to handle more users.

> [!question]- Q3. What are the advantages and disadvantages of microservices? 🟢
> **Advantages:** deploy each service independently, scale only the busy part, one team owns one service, one failure does not always kill the whole app, freedom to choose technology.
>
> **Disadvantages:** network calls can fail or be slow, data is spread over many databases, testing and debugging are harder, you need DevOps skills (Docker, CI/CD, monitoring).
>
> Interview tip: always mention both sides. It shows real-world thinking.

> [!question]- Q4. When should you NOT use microservices? 🟡
> Avoid them when the team is small (1 to 5 developers), the product idea is still changing, you do not know the business boundaries yet, or you have no monitoring and DevOps skills.
> A **modular monolith** is often a better first step.
>
> 📖 **Simple words:**
> - **Modular monolith** = one deployable app, but the code is split into clean, separate modules that could become services later.

> [!question]- Q5. What is a bounded context and how do you decide the size of a service? 🟡
> A **bounded context** is a boundary inside which a word has one clear meaning. "Customer" in Billing (card, invoices) is not the same as "Customer" in Shipping (address, delivery slot). Each bounded context is a good candidate for a service.
>
> Size check:
> - One team can own it.
> - It changes for one reason.
> - It can be deployed without touching other services.
> - If two services always deploy together, they are probably one service.
>
> 📖 **Simple words:**
> - **DDD (Domain-Driven Design)** = designing software around real business language and rules.

> [!question]- Q6. What is the Twelve-Factor App? 🟡
> It is a checklist of good habits for cloud-ready apps. The most useful ones for Spring Boot:
> - **Config** lives in environment variables, not in code.
> - **Stateless** processes (no user data kept in memory).
> - **Backing services** (DB, message broker) are plugged in with a URL, so they can be replaced.
> - **Logs** go to the console; the platform collects them.
> - **Dev/prod parity**: the same build runs in every environment.
> - **Disposability**: fast start and graceful shutdown.

> [!question]- Q7. What is the CAP theorem and eventual consistency? 🟡
> When the network between servers breaks (a **partition**), you can keep only one of these:
> - **Consistency**: everyone sees the latest data.
> - **Availability**: every request gets an answer.
>
> You cannot avoid network problems, so you choose. A bank usually chooses consistency. A social feed usually chooses availability.
> Microservices often use **eventual consistency**: data may differ for a short time, but all copies become equal soon.
>
> 📖 **Simple words:**
> - **Partition** = a network failure that splits servers so they cannot talk.
> - **Eventual consistency** = copies of data will match... after a short delay, not instantly.

> [!question]- Q8. Which microservice patterns should every developer know? 🟡
> | Pattern | Problem it solves |
> |---|---|
> | API Gateway | One entry point for clients |
> | Service Discovery | Find services whose address changes |
> | Circuit Breaker | Stop calling a failing service |
> | Database per Service | Each service owns its data |
> | Saga | A business transaction across many services |
> | Outbox | Save data and publish an event safely |
> | CQRS | Separate models for reading and writing |
> | Strangler Fig | Move from monolith step by step |
>
> Each pattern is explained in later questions.

---
Previous: - | Next: [[02 - Java Essentials for Microservices]]
