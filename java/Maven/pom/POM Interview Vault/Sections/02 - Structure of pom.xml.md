---
tags: [maven, interview, section-2]
---
# Section 2: Structure of pom.xml (Q11–20)
Prev: [[01 - Basics of pom.xml]] · Next: [[03 - Dependencies]] · Home: [[00 - Home]]

## Q11. Main elements?
`modelVersion`, `groupId`, `artifactId`, `version`, `parent`, `properties`, `dependencies`, `dependencyManagement`, `build`, `modules`. Not every project needs all.

## Q12. modelVersion?
```xml
<modelVersion>4.0.0</modelVersion>
```
Version of the POM model, not Maven's version and not your app's version.

## Q13. groupId?
Organization/group identifier, e.g. `com.company.hr`. Need not match Java package exactly.

## Q14. artifactId?
Identifies the project within the group, e.g. `employee-service`. Coordinates: `com.company.hr:employee-service:1.0.0`.

## Q15. `<version>`?
Your artifact's release version (`1.0.0`, `1.1.0`). Not Spring Boot's or a dependency's version.

## Q16. `<packaging>`?
`jar`, `war`, `pom`. Default is `jar`. WAR for external servlet containers; `pom` for parent/aggregator projects.

## Q17. groupId vs artifactId vs version?
Who owns it / which artifact / which release. Together = Maven coordinates `com.company:payment-service:2.1.0`.

## Q18. `<parent>` tag?
Inherits config from another POM. Spring Boot parent gives dependency management, plugin config, build defaults.
```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.5.0</version>
  <relativePath/>
</parent>
```
> [!quote] #interview-answer
> "A parent POM reduces repeated configuration. The Spring Boot parent provides sensible defaults and manages versions for common dependencies."

## Q19. `<relativePath/>`?
Disables the default local lookup (`../pom.xml`); parent is resolved from repositories instead.

## Q20. `<properties>`?
Reusable values: `<java.version>17</java.version>`, referenced as `${java.version}`.
> [!warning]
> A property only matters if a plugin or parent config actually references it.
