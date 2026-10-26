---
tags: [maven, interview, section-6, plugins]
---
# Section 6: Plugins and Java Config (Q56–65)
Prev: [[05 - Lifecycle and Commands]] · Next: [[07 - Profiles and Environments]]

## Q56. Maven plugin?
Provides goals for tasks: compile, test, package JAR, reports, executable Spring Boot apps.

## Q57. Compiler Plugin?
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration><release>17</release></configuration>
</plugin>
```
JDK must be compatible. In Spring Boot, check parent config first to avoid duplication.

## Q58. Configure Java version?
Spring Boot parent: `<java.version>17</java.version>`. Without it: configure Compiler Plugin directly.

## Q59. pom Java version ≠ installed JDK?
Older JDK can't compile newer target → build fails. Java 17 bytecode on Java 11 runtime → startup fails. Check `java -version` and `mvn -version`. Keep versions consistent across dev, build, test, prod.

## Q60. Surefire?
Runs unit tests in the `test` phase; generates reports.

## Q61. Failsafe?
Runs integration tests (`integration-test` + `verify` phases). Use `mvn verify`.
> [!quote] #interview-answer
> "Surefire for fast unit tests, Failsafe for integration tests that need more infrastructure."

## Q62. Spring Boot Maven Plugin?
`repackage` goal makes an executable JAR/WAR. `mvn spring-boot:run` runs the app. Different from the Spring Boot framework itself.

## Q63. plugins vs pluginManagement?
`plugins` = used by this build. `pluginManagement` = default versions/config; doesn't run goals by itself. Useful in multi-module projects.

## Q64. Dependency vs plugin?
| Dependency | Plugin |
|---|---|
| Library for app/tests | Tool for Maven build tasks |
| `<dependencies>` | `<build><plugins>` |
| Spring Web, JUnit, MySQL | Compiler, Surefire, Spring Boot plugin |

## Q65. `<build>` section?
Plugins, plugin config, resources, artifact name. `<finalName>employee-service</finalName>` → `target/employee-service.jar`.
