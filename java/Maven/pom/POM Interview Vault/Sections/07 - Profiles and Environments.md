---
tags: [maven, interview, section-7, profiles]
---
# Section 7: Profiles and Environments (Q66–75)
Prev: [[06 - Plugins and Java Config]] · Next: [[08 - Enterprise and Multi-Module]] · See [[Maven Profile vs Spring Profile]]

## Q66. Maven profile?
Customizes build per situation. Defined in `pom.xml` or `settings.xml`; activate with `mvn clean package -Pproduction`. Doesn't configure Spring runtime unless connected explicitly.

## Q67. Maven vs Spring profile?
Maven = build time; Spring = runtime (`--spring.profiles.active=prod`).

## Q68. Manage dev/test/prod?
Spring profiles + externalized config: `application.properties`, `application-dev/test/prod.properties`. Secrets via secure mechanism.

## Q69. DB passwords in pom.xml?
**No.** POMs go to version control. Use env vars:
```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

## Q70. Build-time vs runtime config?
Build: compiler settings, dependency versions, plugins, packaging. Runtime: DB settings, active profile, port, external endpoints. Same artifact, different external config per environment.

## Q71. `settings.xml`?
User/machine Maven config at `~/.m2/settings.xml`: mirrors, credentials, proxies, local repo location. Protect credentials.

## Q72. pom.xml vs settings.xml?
pom = project config, committed. settings = per-user/environment, not committed.

## Q73. Maven mirror?
Redirects repository requests (e.g. to the company repo): central control, caching, governance. Configured in `settings.xml`.

## Q74. Nexus / Artifactory?
Artifact repositories storing approved third-party and internal libraries; share internal libs, governance, caching, access control.

## Q75. Controlled dependency sources in production?
Supply-chain safety: avoids vulnerabilities, unreviewed code, surprise versions, license issues. Internal repo + scanning + version policy + controlled pipeline.
