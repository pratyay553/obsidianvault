---
tags: [interview, scenarios]
---
# Real-World Scenarios

## 1. Works locally, fails in production
Check Java version/runtime, active Spring profile, DB details, env vars, external deps, deployed JAR, required runtime deps, logs/stack trace.

## 2. Two developers, different build results
Compare JDK, Maven, env vars, dependency config. Use Maven Wrapper, managed versions, and a clean CI build.

## 3. Outdated dependency with vulnerability
Find the source via dependency tree + scan report → compatible patched version → update through version management → run tests → verify resolved versions → release.

## 4. 20 Maven modules
Common parent POM or BOM for shared versions; central plugin config; each module declares only what it needs.

## 5. Why `mvn clean verify` before merging?
Cleans output, compiles, runs configured tests, packages, runs verification. Security scans/quality analysis need separate configuration.

See [[Concepts/Common Errors]], [[Sections/09 - Troubleshooting]]
