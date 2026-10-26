---
tags: [maven, interview, section-5, lifecycle]
---
# Section 5: Lifecycle and Commands (Q46–55)
Prev: [[04 - Parent POM, BOM, Versions]] · Next: [[06 - Plugins and Java Config]] · See [[Maven Lifecycle]], [[Bonus/Maven Commands Cheat Sheet]]

## Q46. Build lifecycle?
Standard sequence of phases. Three lifecycles: `default`, `clean`, `site`.
`validate → compile → test → package → verify → install → deploy` (not exhaustive). A later phase runs earlier ones.

## Q47. compile vs test vs package?
`mvn compile` compiles main code; `mvn test` runs tests; `mvn package` runs earlier phases and builds JAR/WAR in `target/`.

## Q48. `mvn clean`?
Removes `target/`. Does not touch source or `~/.m2`.

## Q49. `clean package` vs `package`?
`clean package` first deletes old output; ideal for CI/CD to avoid stale files.

## Q50. install vs deploy?
`install` → local Maven repo. `deploy` → configured remote repo (needs credentials).

## Q51. `target` directory?
Build output: `classes/`, `test-classes/`, `surefire-reports/`, the JAR. Don't commit to Git.

## Q52. `mvn package` vs `java -jar`?
Maven builds; `java -jar target/employee-service.jar` runs. Production can run the CI-built JAR without Maven.

## Q53. `mvn verify`?
Runs lifecycle through verify (compile, test, package, plus any integration/quality checks configured). Doesn't guarantee every check exists.

## Q54. Goals vs phases?
Phase = stage (`package`). Goal = plugin task (`mvn compiler:compile`). Phases execute goals bound to them.

## Q55. Maven Wrapper?
`mvnw`, `mvnw.cmd`, `.mvn/wrapper/`. Uses the project's Maven version: `./mvnw clean package`. Consistent across devs and CI.
