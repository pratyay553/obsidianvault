---
tags: [maven, interview, section-1]
---
# Section 1: Basics of pom.xml (Q1–10)
Next: [[02 - Structure of pom.xml]] · Home: [[00 - Home]]

## Q1. What is pom.xml in a Java Maven application?
The main configuration file of a Maven project: project info, dependencies, Java version, plugins, packaging. You declare Spring Web in it and Maven manages the library.

> [!quote] #interview-answer
> "`pom.xml` is the central configuration file for dependency management and the build process. It helps manage libraries, Java versions, plugins and packaging without handling every JAR manually."

## Q2. Full form of POM?
**Project Object Model.** It describes structure, dependencies and build configuration in XML.

## Q3. Why do we need pom.xml?
Without Maven: manual JAR downloads, version confusion, missing dependencies, developers on different versions, hard builds on new machines. With it, everything is declared in one place and Maven resolves it.

> [!quote] #interview-answer
> "In enterprise applications with many libraries and developers, `pom.xml` gives a standard, repeatable way to build the application."

## Q4. Is pom.xml mandatory for every Java application?
No. Simple programs can use `javac Main.java` / `java Main`. For medium/large apps Maven makes dependencies, tests and packaging easier. Java = language; Maven = build tool.

## Q5. What is Maven?
A build automation and project management tool: manages dependencies, compiles, tests, packages JAR/WAR, downloads from repositories, standardizes builds (`mvn clean package`).

## Q6. Relationship between Java, Spring Boot, Maven and pom.xml?
See table in [[What is pom.xml]]. Java = language, Spring Boot = framework, Maven = build, pom.xml = Maven config, JVM = runtime.

## Q7. What if we delete pom.xml?
Maven commands fail (no project POM). Source files remain; you lose dependency declarations and build config until restored.

## Q8. pom.xml vs application.properties?
| pom.xml | application.properties |
|---|---|
| Used by Maven | Used by the app |
| Dependencies, plugins, metadata | Port, DB URL, runtime settings |
| Build time | Startup/runtime |

```xml
<dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency>
```
```properties
server.port=8081
```

## Q9. Maven vs Gradle?
| Maven | Gradle |
|---|---|
| XML, `pom.xml` | Groovy/Kotlin DSL, `build.gradle(.kts)` |
| Convention-based, declarative | Flexible, programmable |
`mvn clean package` vs `./gradlew clean build`. Choice depends on team and project.

## Q10. Can Spring Boot run without Maven?
Yes. Maven only builds. With an executable JAR: `java -jar employee-service.jar` (Java runtime still needed).
