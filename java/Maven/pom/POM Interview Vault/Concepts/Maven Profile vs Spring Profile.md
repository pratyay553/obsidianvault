---
tags: [concept, maven, spring-boot]
---
# Maven Profile vs Spring Profile
| Maven profile | Spring profile |
|---|---|
| Build time | Runtime |
| `mvn clean package -Pproduction` | `java -jar app.jar --spring.profiles.active=prod` |
| Defined in `pom.xml` / `settings.xml` | `application-prod.properties` |

Keep secrets out of `pom.xml`; use env vars: `spring.datasource.password=${DB_PASSWORD}`.

See [[Sections/07 - Profiles and Environments]]
