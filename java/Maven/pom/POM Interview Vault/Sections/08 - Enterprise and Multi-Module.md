---
tags: [maven, interview, section-8, enterprise]
---
# Section 8: Enterprise and Multi-Module (Q76–85)
Prev: [[07 - Profiles and Environments]] · Next: [[09 - Troubleshooting]] · See [[Parent vs BOM vs Aggregator]]

## Q76. How does pom.xml help enterprise apps?
Many classes, libs, teams, tests, environments, modules, compliance. pom.xml standardizes dependencies, compiler config, plugins and builds for all developers and CI/CD.

## Q77. Multi-module project?
Related modules under a common parent/aggregator:
```
employee-management/
  pom.xml
  employee-api/  employee-service/  employee-data/
```
Not the same as microservices.

## Q78. Parent POM vs aggregator POM?
Parent = shared config via inheritance. Aggregator = lists modules to build together. One root POM often does both.
```xml
<packaging>pom</packaging>
<modules><module>employee-api</module><module>employee-service</module></modules>
```

## Q79. `<modules>`?
Lists subprojects relative to the aggregator POM; Maven builds them in dependency order.

## Q80. Share dependency versions?
Parent `dependencyManagement` defines versions; children declare without `<version>`.

## Q81. Module depends on another module?
```xml
<dependency>
  <groupId>com.company</groupId><artifactId>employee-common</artifactId>
  <version>${project.version}</version>
</dependency>
```
Typical shared content: DTOs, validators, domain classes.

## Q82. Multi-module vs microservices?
Multi-module = code/build organization. Microservices = independently deployable services (own repo/pipeline/lifecycle). Related, not the same.

## Q83. Corporate parent POM?
Internal parent with Java settings, approved plugin versions, dependency management, test/quality config, repo conventions, build policies.

## Q84. Reusable enterprise libraries?
Build as Maven artifact, publish to the company repo, declare as dependency. Avoid unnecessary coupling between services.

## Q85. Maven in CI/CD?
`mvn clean verify` in a pipeline:
`Push → CI → Checkout → Set up JDK → Resolve deps → Compile/Test → Package → Quality/Security checks → Publish → Deploy`
