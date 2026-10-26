---
tags: [maven, interview, section-3, dependency]
---
# Section 3: Dependencies (Q21–35)
Prev: [[02 - Structure of pom.xml]] · Next: [[04 - Parent POM, BOM, Versions]] · Scopes: [[Dependency Scopes]]

## Q21. What is a dependency?
An external library/project your app needs, declared in `<dependency>`; Maven resolves it. Avoids rewriting web, DB, security, logging, testing code.

## Q22. What happens when we add one?
Read declaration → check local repo → check remote repos → resolve transitive deps → add to classpaths. IDEs may auto-import. Download happens when Maven/IDE needs it.

## Q23. Direct vs transitive?
Direct = you declare it. Transitive = needed by your dependency (Spring Web → Spring Framework libs). Learn the tree to debug conflicts and security issues.

## Q24. Maven repository?
Stores JARs/POMs. **Local** `~/.m2/repository`; **Remote** (company repo); **Central** (Maven Central). Downloads are cached locally.

## Q25. dependencies vs dependencyManagement?
- `dependencies`: what the project **uses**.
- `dependencyManagement`: controls **versions/config**; does not add the dependency by itself.
Child declares the dependency without `<version>` and inherits it.
> [!quote] #interview-answer
> "`dependencies` says which libraries we need; `dependencyManagement` keeps their versions consistent across modules."

## Q26. Dependency scope?
Where a dependency is available. See [[Dependency Scopes]].

## Q27. compile vs runtime vs test?
- compile: web starter (default).
- runtime: `mysql-connector-j` (code doesn't reference driver classes directly).
- test: `spring-boot-starter-test`.
```xml
<dependency><groupId>com.mysql</groupId><artifactId>mysql-connector-j</artifactId><scope>runtime</scope></dependency>
```

## Q28. No scope specified?
Defaults to `compile`. Choose explicit scope for test/runtime-only libraries.

## Q29. spring-boot-starter-web?
Brings Spring MVC + embedded servlet container for REST/web apps.
```java
@RestController @RequestMapping("/employees")
public class EmployeeController {
    @GetMapping public String getEmployees() { return "Employee list"; }
}
```

## Q30. What is a starter?
A dependency descriptor grouping compatible libs: `-web`, `-data-jpa`, `-security`, `-test`, `-validation`.

## Q31. spring-boot-starter-data-jpa?
JPA repositories, entities, persistence, Hibernate.
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {}
```
You still need a DB driver and DB config.

## Q32. Database driver dependency?
Lets the app talk to a DB via JDBC (MySQL → Connector/J, PostgreSQL → its driver). Missing driver = connection failure.

## Q33. spring-boot-starter-test?
JUnit Jupiter, Mockito, Spring Test, AssertJ. Use `test` scope.

## Q34. Which dependency brings another?
```bash
mvn dependency:tree
mvn dependency:tree -Dincludes=org.slf4j
```

## Q35. Optional dependency?
`<optional>true</optional>`: not passed on to consumers of your library. Your own project still uses it.
