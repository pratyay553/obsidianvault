---
tags: [concept, maven]
---
# What is pom.xml?
**POM = Project Object Model.** The main Maven config file: project identity, dependencies, Java version, plugins, packaging.

**Flow:** write code → declare libraries in `pom.xml` → Maven reads it → downloads dependencies (local → remote repo) → compiles → runs tests → packages JAR/WAR.

```bash
mvn clean package
```

| Tool | Role |
|---|---|
| Java | Language |
| Spring Boot | Application framework |
| Maven | Build + dependency tool |
| `pom.xml` | Maven project config |
| JVM | Runs bytecode |

Minimal Spring Boot POM: see [[Bonus/Sample Enterprise pom.xml]].
Related: [[Sections/01 - Basics of pom.xml]], [[Sections/02 - Structure of pom.xml]]
