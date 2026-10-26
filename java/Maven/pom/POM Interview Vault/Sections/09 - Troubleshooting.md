---
tags: [maven, interview, section-9, troubleshooting]
---
# Section 9: Troubleshooting (Q86–95)
Prev: [[08 - Enterprise and Multi-Module]] · Next: [[10 - Advanced]] · See [[Common Errors]]

## Q86. Maven can't download a dependency?
Causes: network, wrong coordinates, repo down, bad credentials, proxy, nonexistent version. Try `mvn -U clean package`. `-U` won't fix wrong coordinates, auth failure or unavailable repo.

## Q87. `-U` vs `-o`?
`-U` forces update checks of snapshots/missing releases. `-o` offline; works only if everything is already local.

## Q88. Local repo; can we delete it?
`~/.m2/repository`. Delete only specific corrupted artifacts, not the whole repo unless truly needed.

## Q89. "Could not resolve dependencies"?
Read full error → verify groupId/artifactId/version → check network/repo → credentials → corrupted cache → `mvn dependency:tree` → retry. Don't randomly change versions.

## Q90. NoSuchMethodError?
Linkage error: method missing in the loaded class version; often a version conflict. Check `mvn dependency:tree` and the full stack trace.

## Q91. ClassNotFoundException vs NoClassDefFoundError?
- `ClassNotFoundException`: explicit load by name (`Class.forName`) fails.
- `NoClassDefFoundError`: JVM needs a class definition it can't find/load at runtime (e.g. missing from runtime classpath).
Check scopes, tree, packaging.

## Q92. Works on one machine, fails on another?
Different JDK/Maven, env vars, caches, config, secrets, missing declared deps, OS/paths. Fix: consistent Java, Maven Wrapper, explicit dependency management, externalized config, tests, CI.

## Q93. Slow Maven build?
Find the slow stage: downloads, big tests, integration tests, quality checks, plugin runs, rebuilding unchanged modules. Use CI timing and plugin reports (`-Dprofile` has no universal effect). Optimize the real bottleneck.

## Q94. Skipping tests vs ignoring failures?
`-DskipTests`: skips running, still compiles tests. `-Dmaven.test.skip=true`: skips compiling and running. Only for temporary local use, not releases.

## Q95. Builds fine, fails at runtime?
Read stack trace → check runtime Java → DB/external services → runtime deps → conflicts → Spring profile → env vars/secrets → confirm the right artifact. `mvn dependency:tree`, `java -jar target/employee-service.jar`.
