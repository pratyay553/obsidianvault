---
tags: [concept, maven]
---
# Parent vs BOM vs Aggregator
| | Parent POM | BOM | Aggregator |
|---|---|---|---|
| Tag | `<parent>` | `<dependencyManagement>` + `scope=import` | `<modules>` |
| Purpose | Inherit config | Manage versions only | Build modules together |
| Count | Only one direct parent | Many allowed | – |

- `spring-boot-starter-parent` = parent: versions + plugin defaults.
- `spring-boot-dependencies` = BOM: versions only; use when you already have another parent.
- Root POM of multi-module projects is often both parent and aggregator (`<packaging>pom</packaging>`).

See [[Sections/04 - Parent POM, BOM, Versions]], [[Sections/08 - Enterprise and Multi-Module]]
