---
tags: [concept, troubleshooting]
---
# Common Errors
- **Could not resolve dependencies** → check coordinates, network, credentials, corrupt cache; `mvn -U clean package`.
- **NoSuchMethodError** → version conflict; `mvn dependency:tree`.
- **ClassNotFoundException** → class loaded by name (`Class.forName`) not found.
- **NoClassDefFoundError** → class existed at build but missing/failed at runtime (scope, packaging).
- **Works on my machine** → compare JDK, Maven, env vars, config; use Maven Wrapper + CI.

See [[Sections/09 - Troubleshooting]]
