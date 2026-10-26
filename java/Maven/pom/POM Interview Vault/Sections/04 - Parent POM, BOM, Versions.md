---
tags: [maven, interview, section-4, spring-boot]
---
# Section 4: Parent POM, BOM, Versions (Q36–45)
Prev: [[03 - Dependencies]] · Next: [[05 - Lifecycle and Commands]] · See [[Parent vs BOM vs Aggregator]]

## Q36. spring-boot-starter-parent?
Spring Boot's parent POM: dependency versions, plugin config, build conventions, encoding defaults.
> [!quote] #interview-answer
> "I use the Spring Boot parent because it gives a tested set of versions and sensible defaults, reducing conflicts and repeated config."

## Q37. Why omit versions?
The parent manages them: consistent versions, less config, easier upgrades, fewer incompatibilities. Unmanaged libs still need a version.

## Q38. BOM?
**Bill of Materials**: a POM that centrally manages versions.
```xml
<dependencyManagement><dependencies><dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>3.5.0</version>
  <type>pom</type><scope>import</scope>
</dependency></dependencies></dependencyManagement>
```

## Q39. starter-parent vs spring-boot-dependencies?
| starter-parent | spring-boot-dependencies |
|---|---|
| Used via `<parent>` | Imported as BOM |
| Versions + plugin/build defaults | Versions only |
Need a different parent? Import the BOM.

## Q40. Multiple parents?
No, one direct parent. A parent can have its own parent (chain). You can import multiple BOMs.

## Q41. Two dependencies need different versions?
Dependency mediation: **nearest definition wins**; at the same depth, first declaration wins. May cause runtime errors. Investigate with `mvn dependency:tree`; align via dependencyManagement or exclusions.

## Q42. Override a Spring Boot managed version?
First ask whether it's needed. Then use a supported BOM property or your own dependencyManagement entry; test and check the tree.
```xml
<properties><some.library.version>2.0.0</some.library.version></properties>
```
(works only if the BOM uses that property; coordinates illustrative)

## Q43. Version conflict?
Different libs need different versions of one library. Symptoms: `NoSuchMethodError`, `ClassNotFoundException`, startup failure. Fix via compatible versions, management, or exclusions.

## Q44. Exclusion?
```xml
<exclusions><exclusion>
  <groupId>org.example</groupId><artifactId>unwanted-library</artifactId>
</exclusion></exclusions>
```
Only affects that dependency path; verify with the tree.

## Q45. Why avoid arbitrary versions?
Compatibility breakage (e.g. Hibernate vs Boot), security risk, inconsistent builds. Use supported versions, central management, advisories, tests, tree reviews, documented exceptions.
