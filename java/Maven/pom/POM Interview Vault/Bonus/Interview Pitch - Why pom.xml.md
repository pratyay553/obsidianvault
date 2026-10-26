---
tags: [interview-answer]
---
# "Why is pom.xml important? Can't you add JARs manually?"

> [!quote] #interview-answer
> In a small app we can manage JARs manually. In an enterprise Spring Boot app, dependencies pull in more libraries, so we get version conflicts, missing dependencies and inconsistent builds. With Maven and `pom.xml` we declare dependencies, manage compatible versions, configure plugins and define the project structure. Maven resolves dependencies, compiles, runs tests and packages the app, and keeps the build consistent across developer machines and CI/CD. So `pom.xml` reduces manual effort, improves maintainability and makes the app easier to build, test and release.

Related: [[Sections/10 - Advanced]]
