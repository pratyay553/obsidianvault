---
tags: [concept, maven, lifecycle]
---
# Maven Lifecycle
Three lifecycles: **default**, **clean**, **site**.

`validate → compile → test → package → verify → install → deploy`

Running a phase runs all earlier phases. A **goal** is a plugin task (`compiler:compile`); a **phase** is a stage.
- `install` → local repo (`~/.m2`)
- `deploy` → remote repo (Nexus/Artifactory)

See [[Sections/05 - Lifecycle and Commands]], [[Bonus/Maven Commands Cheat Sheet]]
