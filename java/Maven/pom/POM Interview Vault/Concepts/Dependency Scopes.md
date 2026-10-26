---
tags: [concept, maven, dependency]
---
# Dependency Scopes
| Scope | Meaning |
|---|---|
| `compile` | Default. Compile + test + runtime |
| `provided` | Compile + test; runtime supplied by container |
| `runtime` | Not for compiling; needed to run (e.g. DB driver) |
| `test` | Tests only |
| `system` | Local path; discouraged |
| `import` | Imports a BOM into `dependencyManagement` |

See [[Sections/03 - Dependencies]] (Q26–28)
