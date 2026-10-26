---
tags: [maven, cheatsheet]
---
# Maven Commands Cheat Sheet
| Command | Purpose |
|---|---|
| `mvn clean` | Remove previous build output |
| `mvn compile` | Compile main code |
| `mvn test` | Run test phase |
| `mvn package` | Build and package |
| `mvn clean package` | Fresh package |
| `mvn verify` | Lifecycle through verification |
| `mvn install` | Install to local repo |
| `mvn deploy` | Publish to remote repo |
| `mvn dependency:tree` | Dependency hierarchy |
| `mvn help:effective-pom` | Effective POM |

Extras: `-U` force update · `-o` offline · `-Pprod` profile · `-DskipTests` · `./mvnw` wrapper · `mvn spring-boot:run`

Best validation command: `mvn clean verify`. See [[Concepts/Maven Lifecycle]].
