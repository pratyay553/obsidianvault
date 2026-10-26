---
tags: [maven, interview, section-10, advanced]
---
# Section 10: Advanced (Q96–100)
Prev: [[09 - Troubleshooting]] · Home: [[00 - Home]]

## Q96. Effective POM?
Final POM after merging inheritance, active profiles and defaults.
```bash
mvn help:effective-pom
```
Great for finding where inherited config comes from.

## Q97. Snapshot vs release?
`1.0.0-SNAPSHOT` = changing development version. `1.0.0` = fixed release. Production should use release artifacts, not changing snapshots.

## Q98. Reproducible build?
Same source + same inputs = same artifact. Pin dependency and plugin versions, consistent JDK, Maven Wrapper, no snapshots in releases, reproducible archive settings, controlled CI/CD.

## Q99. Dependency security?
Supported versions → vulnerability reports → find which direct/transitive dep introduces it → compatible patched version → careful upgrade/override → tests → verify packaged versions → scanning in CI/CD.

## Q100. Complete role of pom.xml in enterprise Spring Boot?
> [!quote] #interview-answer
> "`pom.xml` is the central Maven configuration. It defines project identity (`groupId`, `artifactId`, `version`), manages dependencies (Spring Web, Data JPA, DB driver), keeps versions compatible via the Spring Boot parent or BOM, configures plugins for compile/test/package, and supports automated builds with `mvn clean verify` and `mvn clean package`. In larger systems it enables multi-module projects, reusable internal libraries and artifact repositories, giving developers and CI/CD a consistent build process."

Revision: [[Bonus/Final Revision - 15 Points]]
