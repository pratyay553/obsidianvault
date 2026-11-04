---
tags: [microservices, springboot, moc]
---

# 15 Design Principles for Spring Boot Microservices

A map of content. Work through the notes in order, or jump by theme. Mark each note's `status` as `to-review`, `learning` or `done`.

## Principles
1. [[01 - Single Responsibility and Bounded Context]]
2. [[02 - Database per Service]]
3. [[03 - APIFirst Design]]
4. [[04 - Loose Coupling and High Cohesion]]
5. [[05 - Statelessness]]
6. [[06 - Externalized Configuration]]
7. [[07 - Resilience and Design for Failure]]
8. [[08 - Service Discovery and API Gateway]]
9. [[09 - Asynchronous and EventDriven Communication]]
10. [[10 - Distributed Data Consistency Saga and Outbox]]
11. [[11 - Observability]]
12. [[12 - Security by Design]]
13. [[13 - Automation CICD and Infrastructure as Code]]
14. [[14 - Containerization and Elastic Scalability]]
15. [[15 - Versioning and Backward Compatibility]]

## By theme
- **Design and domain:** [[01 - Single Responsibility and Bounded Context]], [[04 - Loose Coupling and High Cohesion]]
- **APIs and contracts:** [[03 - APIFirst Design]], [[15 - Versioning and Backward Compatibility]]
- **Data:** [[02 - Database per Service]], [[10 - Distributed Data Consistency Saga and Outbox]]
- **Communication:** [[09 - Asynchronous and EventDriven Communication]], [[08 - Service Discovery and API Gateway]]
- **Reliability and scale:** [[05 - Statelessness]], [[07 - Resilience and Design for Failure]], [[14 - Containerization and Elastic Scalability]]
- **Operations and delivery:** [[06 - Externalized Configuration]], [[11 - Observability]], [[13 - Automation CICD and Infrastructure as Code]]
- **Security:** [[12 - Security by Design]]

## Tools
- [[Service Design Review Checklist]]
- Template: `Templates/Principle Template`

## Dataview: progress (optional plugin)
```dataview
TABLE status, principle
FROM "Principles"
SORT principle ASC
```
