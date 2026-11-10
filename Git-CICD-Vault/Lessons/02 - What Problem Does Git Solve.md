---
tags: [git, lesson]
lesson: 2
---
# Lesson 2 – What Problem Does Git Solve?
⬅️ [[01 - The Big Picture]] | ➡️ [[03 - Git Architecture]]

A Spring Boot project starts simple:
```text
scheduler/
    src/
    pom.xml
    application.yml
```
You add employee management ✅, appointment scheduling ✅, Kafka integration ❌ (everything breaks). You want the previous working version.

> [!failure] Without version control
> ```text
> scheduler-final/
> scheduler-final-new/
> scheduler-final-updated/
> scheduler-final-latest/
> scheduler-final-latest-working/
> ```

> [!success] With Git
> ```mermaid
> flowchart TD
>     V1[V1: Initial Spring Boot app] --> V2[V2: Employee module]
>     V2 --> V3[V3: Appointment module]
>     V3 --> V4[V4: Kafka integration]
>     V4 --> V5[V5: Bug fix]
> ```

Git remembers project history so you can inspect and restore previous versions. This is **Version Control**.
