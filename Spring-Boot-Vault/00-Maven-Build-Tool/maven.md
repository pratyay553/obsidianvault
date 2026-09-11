Absolutely. Below is a **comprehensive interview question bank** focused on **Maven + Spring Boot project management**, especially the kind of questions asked around **build tools, `pom.xml`, dependency management, lifecycle, plugins, profiles, internal resolution, troubleshooting, and project clarification**.

## 1. Maven Fundamentals

1. What is Maven?
    
2. Why do we use Maven in Java projects?
    
3. What problem does Maven solve?
    
4. What is a build tool?
    
5. What is the difference between Maven, Gradle, and Ant?
    
6. What is the Maven architecture?
    
7. What is the Maven lifecycle?
    
8. What are the three standard Maven lifecycles?
    
9. What are the phases in the Maven default lifecycle?
    
10. What is the difference between a Maven phase and a Maven goal?
    
11. What is a Maven plugin?
    
12. What is a Maven goal?
    
13. How are plugins and goals related?
    
14. What happens when you run `mvn clean install`?
    
15. What is the difference between `mvn package` and `mvn install`?
    
16. What is the difference between `mvn install` and `mvn deploy`?
    
17. What does `mvn clean` do?
    
18. What does `mvn validate` do?
    
19. What does `mvn compile` do?
    
20. What does `mvn test` do?
    
21. What does `mvn verify` do?
    
22. What does `mvn site` do?
    
23. What is Maven's default lifecycle?
    
24. Can you execute a Maven goal without executing the complete lifecycle?
    
25. What happens if a previous Maven phase fails?
    

---

# 2. `pom.xml` Interview Questions

26. What is `pom.xml`?
    
27. What does POM stand for?
    
28. What are the major sections of a `pom.xml`?
    
29. What is `<groupId>`?
    
30. What is `<artifactId>`?
    
31. What is `<version>`?
    
32. What is `<packaging>`?
    
33. What is the default packaging type?
    
34. What is the difference between `jar`, `war`, and `pom` packaging?
    
35. What is `<name>`?
    
36. What is `<description>`?
    
37. What is `<url>`?
    
38. What is `<properties>`?
    
39. Why do we define versions inside `<properties>`?
    
40. How do you use a property inside a dependency?
    
41. What is `<dependencies>`?
    
42. What is `<dependencyManagement>`?
    
43. What is the difference between `<dependencies>` and `<dependencyManagement>`?
    
44. Can `dependencyManagement` download a dependency by itself?
    
45. Why is `dependencyManagement` useful in multi-module projects?
    
46. What is `<build>`?
    
47. What is `<plugins>`?
    
48. What is `<pluginManagement>`?
    
49. Difference between `<plugins>` and `<pluginManagement>`?
    
50. What is `<repositories>`?
    
51. What is `<pluginRepositories>`?
    
52. What is `<distributionManagement>`?
    
53. What is `<parent>`?
    
54. What is `<modules>`?
    
55. What is `<profiles>`?
    
56. What is `<reporting>`?
    
57. What is `<scm>`?
    
58. What is `<developers>`?
    
59. What is `<licenses>`?
    

---

# 3. Maven Dependency Management

60. What is a Maven dependency?
    
61. How does Maven download dependencies?
    
62. Where does Maven store downloaded dependencies?
    
63. What is the local Maven repository?
    
64. What is `.m2`?
    
65. Where is the Maven local repository located?
    
66. What is Maven Central?
    
67. What is a remote repository?
    
68. What is an internal/private Maven repository?
    
69. Why do organizations use Nexus or Artifactory?
    
70. What is dependency resolution?
    
71. How does Maven identify a dependency?
    
72. Explain:
    

```xml
<groupId>
<artifactId>
<version>
```

73. What is a transitive dependency?
    
74. What is direct dependency?
    
75. What is indirect dependency?
    
76. What is dependency mediation?
    
77. What happens when two dependencies require different versions of the same library?
    
78. What is "nearest definition" in Maven?
    
79. How does Maven decide which dependency version to use?
    
80. What is dependency conflict?
    
81. How do you resolve dependency conflicts?
    
82. What is `<exclusions>`?
    
83. Why would you exclude a transitive dependency?
    
84. Can you give an example of dependency exclusion?
    
85. What is dependency scope?
    
86. Explain `compile` scope.
    
87. Explain `provided` scope.
    
88. Explain `runtime` scope.
    
89. Explain `test` scope.
    
90. Explain `system` scope.
    
91. What is the default dependency scope?
    
92. Difference between `compile` and `runtime`.
    
93. Difference between `provided` and `compile`.
    
94. Why is `spring-boot-starter-test` usually test scoped?
    
95. Why shouldn't we manually add every Spring dependency?
    

---

# 4. Maven Dependency Troubleshooting

96. What would you do if Maven cannot download a dependency?
    
97. What does `Could not resolve dependencies` mean?
    
98. What does `Could not find artifact` mean?
    
99. How do you troubleshoot dependency resolution?
    
100. What if the dependency exists in Nexus but Maven still cannot download it?
    
101. What if Maven works locally but fails in Jenkins?
    
102. What if Maven works on one developer's machine but not another?
    
103. How do you force Maven to download dependencies again?
    
104. What does `mvn clean install -U` do?
    
105. When would you delete the `.m2` repository?
    
106. Is deleting `.m2` a good first troubleshooting step?
    
107. How do you identify which dependency introduced a particular library?
    
108. What does this command do?
    

```bash
mvn dependency:tree
```

109. How do you find dependency conflicts?
    
110. What does `mvn dependency:tree -Dverbose` do?
    
111. How do you check effective dependencies?
    
112. What does `mvn help:effective-pom` do?
    
113. How do you identify the version Maven actually selected?
    
114. How do you find duplicate dependencies?
    

---

# 5. Spring Boot + Maven

115. What is Spring Boot?
    
116. Why is Maven commonly used with Spring Boot?
    
117. What is `spring-boot-starter-parent`?
    
118. Why do we use the Spring Boot parent POM?
    
119. What is dependency version management in Spring Boot?
    
120. Does Spring Boot parent contain all dependencies?
    
121. What is a Spring Boot starter?
    
122. What is `spring-boot-starter-web`?
    
123. What is `spring-boot-starter-data-jpa`?
    
124. What is `spring-boot-starter-security`?
    
125. What is `spring-boot-starter-test`?
    
126. Why are starters useful?
    
127. What dependencies are included by `spring-boot-starter-web`?
    
128. What is Spring Boot's embedded Tomcat?
    
129. Why don't we need to install Tomcat separately for a Spring Boot application?
    
130. What is the Spring Boot Maven plugin?
    
131. Why do we use:
    

```xml
<artifactId>spring-boot-maven-plugin</artifactId>
```

132. What does `mvn spring-boot:run` do?
    
133. Difference between:
    

```bash
mvn package
```

and

```bash
mvn spring-boot:run
```

134. How does Spring Boot create an executable JAR?
    
135. What is a fat JAR?
    
136. What is an executable JAR?
    
137. What does `spring-boot:repackage` do?
    
138. Why can a normal JAR sometimes not be directly executed with `java -jar`?
    
139. How do you run a packaged Spring Boot application?
    
140. How do you pass JVM arguments to a Spring Boot Maven application?
    

---

# 6. Spring Boot Parent vs Dependency Management

141. What is the difference between:
    

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
</parent>
```

and importing:

```xml
spring-boot-dependencies
```

142. Can Spring Boot be used without `spring-boot-starter-parent`?
    
143. Why would a company avoid using Spring Boot's parent POM?
    
144. How do you use Spring Boot dependency management with another corporate parent?
    
145. What is a corporate parent POM?
    
146. Can Maven have multiple parent POMs?
    
147. What is the limitation regarding Maven parent inheritance?
    
148. How can you use multiple dependency-management sources?
    

---

# 7. Maven Build Lifecycle

149. Explain the Maven lifecycle from start to finish.
    
150. What happens during `validate`?
    
151. What happens during `compile`?
    
152. What happens during `test`?
    
153. What happens during `package`?
    
154. What happens during `verify`?
    
155. What happens during `install`?
    
156. What happens during `deploy`?
    
157. Explain this command:
    

```bash
mvn clean install
```

158. Explain:
    

```bash
mvn clean package
```

159. Explain:
    

```bash
mvn clean verify
```

160. Explain:
    

```bash
mvn clean install -DskipTests
```

161. Explain:
    

```bash
mvn clean install -Dmaven.test.skip=true
```

162. Difference between `-DskipTests` and `-Dmaven.test.skip=true`.
    
163. How do you run only one test class?
    
164. How do you run one test method?
    
165. How do you skip tests?
    
166. Should tests be skipped in production CI/CD?
    
167. What is the difference between compile failure and test failure?
    

---

# 8. Maven Plugins

168. What is a Maven plugin?
    
169. Why does Maven need plugins?
    
170. What are build plugins?
    
171. What are reporting plugins?
    
172. What is the Maven Compiler Plugin?
    
173. What is the Maven Surefire Plugin?
    
174. What is the Maven Failsafe Plugin?
    
175. What is the Maven JAR Plugin?
    
176. What is the Maven WAR Plugin?
    
177. What is the Maven Resources Plugin?
    
178. What is the Maven Clean Plugin?
    
179. What is the Spring Boot Maven Plugin?
    
180. What is the Maven Enforcer Plugin?
    
181. What is the Maven Dependency Plugin?
    
182. What is the Maven Assembly Plugin?
    
183. What is the Maven Shade Plugin?
    
184. Difference between Surefire and Failsafe.
    
185. How do you configure Java version in Maven?
    
186. What is the difference between `source`, `target`, and `release`?
    
187. How do you configure compiler arguments?
    

---

# 9. Maven Profiles

188. What is a Maven profile?
    
189. Why are Maven profiles used?
    
190. What is an environment-specific profile?
    
191. How do you define a Maven profile?
    
192. How do you activate a Maven profile?
    
193. What does this do?
    

```bash
mvn clean package -Pdev
```

194. Can Maven profiles be activated automatically?
    
195. What is activation by property?
    
196. What is activation by JDK?
    
197. What is activation by OS?
    
198. What is activation by file existence?
    
199. How do Maven profiles differ from Spring Boot profiles?
    
200. Should database credentials be stored in Maven profiles?
    
201. What are the risks of environment-specific Maven configuration?
    

---

# 10. Maven Multi-Module Projects

202. What is a multi-module Maven project?
    
203. Why do companies use multi-module Maven projects?
    
204. What is a parent POM?
    
205. What is an aggregator POM?
    
206. Difference between parent POM and aggregator POM.
    
207. Can a POM be both parent and aggregator?
    
208. What is `<modules>`?
    
209. How does Maven build multiple modules?
    
210. What is module dependency order?
    
211. What happens if module A depends on module B?
    
212. How does Maven determine build order?
    
213. What is reactor build?
    
214. What is Maven Reactor?
    
215. What does this command do?
    

```bash
mvn clean install -pl service-a
```

216. What does `-pl` mean?
    
217. What does `-am` mean?
    
218. Explain:
    

```bash
mvn clean install -pl service-a -am
```

219. What does `-amd` do?
    
220. How do you build only a specific module?
    
221. How do you skip a module?
    
222. How do you share dependency versions between modules?
    
223. How do you share plugin configuration between modules?
    

---

# 11. Internal / Corporate Maven Repository Questions

This is especially important in enterprise interviews.

224. What is an internal Maven repository?
    
225. Why does a company maintain its own Maven repository?
    
226. What is Nexus?
    
227. What is Artifactory?
    
228. What is the difference between Maven Central and Nexus/Artifactory?
    
229. How does Maven know where to download dependencies from?
    
230. What is `settings.xml`?
    
231. Difference between `pom.xml` and `settings.xml`.
    
232. Where is `settings.xml` located?
    
233. What is a Maven mirror?
    
234. What is `<mirrorOf>`?
    
235. Why would an organization configure:
    

```xml
<mirrorOf>*</mirrorOf>
```

236. What is a repository URL?
    
237. What is a repository ID?
    
238. How is repository authentication configured?
    
239. Where should Maven credentials be stored?
    
240. What is a Maven server configuration?
    
241. How are credentials mapped to a repository?
    
242. What is `settings-security.xml`?
    
243. What is Maven password encryption?
    
244. What happens if your internal repository is unavailable?
    
245. Can Maven use Maven Central as a fallback?
    
246. How does an enterprise proxy repository work?
    
247. Why might an organization block direct access to Maven Central?
    
248. What is an artifact repository manager?
    
249. What is a hosted repository?
    
250. What is a proxy repository?
    
251. What is a virtual/group repository?
    
252. Difference between hosted, proxy, and group repositories.
    
253. How does an internal repository improve build reproducibility?
    
254. How do you troubleshoot HTTP 401 from Nexus?
    
255. How do you troubleshoot HTTP 403?
    
256. How do you troubleshoot HTTP 404?
    
257. What does HTTP 407 indicate during Maven dependency download?
    
258. How would you troubleshoot a corporate proxy issue?
    

---

# 12. Internal Artifact Publishing

259. How do you publish a Maven artifact internally?
    
260. What is `mvn deploy`?
    
261. Where does `mvn deploy` upload the artifact?
    
262. What is `<distributionManagement>`?
    
263. Difference between `<repositories>` and `<distributionManagement>`.
    
264. How do you publish a SNAPSHOT?
    
265. What is a SNAPSHOT version?
    
266. What is a release version?
    
267. Difference between:
    

```text
1.0.0-SNAPSHOT
```

and

```text
1.0.0
```

268. Can a release artifact be overwritten?
    
269. Why should release artifacts be immutable?
    
270. How are internal libraries versioned?
    
271. What is semantic versioning?
    
272. How do you consume an internally published library?
    
273. What happens if the internal artifact version doesn't exist?
    
274. How would you debug an artifact that exists in the repository but cannot be resolved?
    

---

# 13. Maven `settings.xml`

275. What is Maven `settings.xml`?
    
276. User-level vs global `settings.xml`.
    
277. What is `<servers>`?
    
278. What is `<mirrors>`?
    
279. What is `<profiles>` inside `settings.xml`?
    
280. What is `<activeProfiles>`?
    
281. What is `<proxies>`?
    
282. How do you configure a corporate proxy?
    
283. How do you configure repository credentials?
    
284. Should passwords be committed to Git?
    
285. How do CI/CD pipelines provide Maven credentials?
    
286. What happens if both user-level and global settings contain configuration?
    
287. Which settings take precedence?
    

---

# 14. Maven + Git + CI/CD

288. How does Maven fit into a CI/CD pipeline?
    
289. Explain a typical Spring Boot Maven pipeline.
    
290. What Maven command would you use in CI?
    
291. Difference between local and CI Maven builds.
    
292. Why might a build pass locally but fail in Jenkins?
    
293. What is a clean build?
    
294. Why should CI use a clean environment?
    
295. How does CI cache Maven dependencies?
    
296. Why is Maven dependency caching useful?
    
297. What happens when a CI agent has an empty `.m2` directory?
    
298. How do you optimize Maven build time?
    
299. How can parallel Maven builds be enabled?
    
300. What is:
    

```bash
mvn -T 1C
```

301. What problems can occur with parallel Maven builds?
    
302. How do you configure Maven in Jenkins?
    
303. How does Jenkins authenticate against Nexus?
    
304. How do you publish a Spring Boot artifact from Jenkins?
    
305. Where should secrets be stored in CI/CD?
    
306. How would you prevent credentials from appearing in Maven logs?
    

---

# 15. Maven + Testing

307. How does Maven execute unit tests?
    
308. Which plugin executes unit tests?
    
309. What is Surefire?
    
310. What is Failsafe?
    
311. Difference between unit and integration tests from Maven's perspective.
    
312. What naming conventions does Surefire recognize?
    
313. What naming conventions does Failsafe recognize?
    
314. How do you separate unit and integration tests?
    
315. How do you run integration tests only?
    
316. How do you skip integration tests?
    
317. How do you configure test profiles?
    
318. How does Maven generate test reports?
    
319. What happens when one unit test fails?
    
320. Does Maven continue compiling other modules after a test failure?
    
321. How do you debug Maven test failures?
    

---

# 16. Maven Security Questions

322. How do you secure Maven dependencies?
    
323. What is dependency vulnerability scanning?
    
324. How do you identify vulnerable dependencies?
    
325. What is dependency convergence?
    
326. Why should dependency versions be controlled?
    
327. What is Maven Enforcer?
    
328. How can Enforcer enforce Java versions?
    
329. How can Enforcer prevent dependency conflicts?
    
330. How do you prevent SNAPSHOT dependencies in production?
    
331. Why shouldn't credentials be committed into `pom.xml`?
    
332. Why shouldn't passwords be committed into `settings.xml`?
    
333. How do you manage private repository credentials securely?
    
334. What is a Software Bill of Materials (SBOM)?
    
335. How can Maven builds participate in supply-chain security?
    

---

# 17. Spring Boot Configuration + Maven

336. Difference between Maven profiles and Spring profiles.
    
337. How do Maven profiles interact with Spring Boot?
    
338. Should `application-dev.yml` be selected through Maven?
    
339. How do you build the same artifact for multiple environments?
    
340. Why is building separate JARs for each environment often discouraged?
    
341. What is externalized configuration?
    
342. How does Spring Boot load configuration?
    
343. How do you pass configuration when starting the JAR?
    
344. Difference between build-time and runtime configuration.
    
345. What should be decided during Maven build?
    
346. What should be decided at Spring Boot runtime?
    

---

# 18. Important Scenario-Based Questions

These are **very common in experienced-level interviews**.

### Scenario 1

**Your Spring Boot project works on your laptop but fails in Jenkins. What will you check?**

Expected discussion:

- Java version
    
- Maven version
    
- `settings.xml`
    
- repository access
    
- credentials
    
- proxy
    
- environment variables
    
- dependency cache
    
- OS differences
    
- Maven profiles
    
- generated resources
    
- filesystem assumptions
    

---

### Scenario 2

**Maven says "Could not resolve dependency." How will you troubleshoot?**

Walk through:

```bash
mvn dependency:tree
mvn help:effective-pom
mvn clean install -U
```

Then check:

- dependency coordinates
    
- repository URL
    
- internal repository
    
- credentials
    
- proxy
    
- artifact existence
    
- `.m2`
    
- parent POM
    
- transitive dependency conflicts
    

---

### Scenario 3

**Your project has dependency A version 1.0 and dependency B brings A version 2.0. Which one does Maven use?**

Discuss:

- dependency mediation
    
- nearest dependency
    
- dependency tree
    
- explicit version management
    
- exclusions
    
- dependencyManagement
    

---

### Scenario 4

**Developer says: "The JAR exists in Nexus, but Maven cannot download it." What do you investigate?**

Check:

- repository ID
    
- repository URL
    
- `settings.xml`
    
- credentials
    
- mirror
    
- proxy
    
- artifact coordinates
    
- snapshot vs release repository
    
- permissions
    
- HTTP response
    

---

### Scenario 5

**A developer committed a password into `pom.xml`. What would you do?**

Discuss:

- immediately rotate/revoke credential
    
- remove secret from repository
    
- check Git history
    
- move credential to secret management/CI configuration
    
- prevent recurrence with secret scanning
    

---

### Scenario 6

**Why does `mvn package` succeed but `java -jar application.jar` fail?**

Investigate:

- Spring Boot Maven Plugin
    
- executable/fat JAR packaging
    
- `repackage`
    
- `Main-Class`
    
- dependency inclusion
    
- manifest
    

---

### Scenario 7

**You changed a dependency version, but Maven is still using the old version. Why?**

Possible areas:

- dependencyManagement
    
- parent POM
    
- transitive dependency
    
- effective POM
    
- Maven profile
    
- duplicate dependency declarations
    
- local repository/cache
    

---

### Scenario 8

**Your build takes 15 minutes. How would you optimize it?**

Discuss:

- dependency caching
    
- parallel builds
    
- unnecessary plugins
    
- test separation
    
- incremental compilation
    
- multi-module structure
    
- Docker build caching
    
- CI workspace caching
    
- repository performance
    

---

### Scenario 9

**One module depends on another module in the same Maven project. How does Maven build them?**

Discuss:

```text
Parent
 ├── module-a
 ├── module-b
 └── module-c
```

If:

```text
module-b → module-a
```

Maven determines the reactor build order:

```text
module-a
   ↓
module-b
```

---

### Scenario 10

**Your organization has a corporate parent POM and Spring Boot also provides a parent POM. Which one do you use?**

This is an excellent interview question because it tests:

- Maven inheritance
    
- parent POM limitations
    
- dependency management
    
- `spring-boot-dependencies`
    
- corporate standards
    

---

# 19. "Internal Clarification" Questions You Should Be Ready For

If by **internal clarification** you mean questions you may be asked when explaining how your organization's Maven/Spring Boot build works, prepare for these:

1. Where does your company host Maven artifacts?
    
2. Do you use Nexus or Artifactory?
    
3. Do developers access Maven Central directly?
    
4. Is Maven Central proxied through an internal repository?
    
5. What is your internal repository URL?
    
6. How is authentication handled?
    
7. Where are Maven credentials configured?
    
8. How does Jenkins authenticate?
    
9. Who manages repository permissions?
    
10. Who publishes internal libraries?
    
11. What is your organization's release process?
    
12. How are SNAPSHOTs handled?
    
13. How are release versions handled?
    
14. Do you have separate snapshot and release repositories?
    
15. What is your branching strategy?
    
16. How is a release version created?
    
17. How is an artifact promoted from one repository to another?
    
18. How are dependencies approved?
    
19. How are vulnerabilities detected?
    
20. Who owns the parent POM?
    
21. Do all projects inherit from a corporate parent?
    
22. Do you maintain a company-wide BOM?
    
23. How do you control dependency versions?
    
24. How do you handle CVE/security upgrades?
    
25. How do you handle breaking dependency upgrades?
    
26. How do you troubleshoot CI-only Maven failures?
    
27. How do developers configure Maven locally?
    
28. What does the standard `settings.xml` contain?
    
29. What Maven command does your CI pipeline execute?
    
30. Where does the generated JAR get published?
    
31. How does deployment happen after Maven build?
    
32. How are environment configurations separated from the artifact?
    
33. What happens when the internal repository is down?
    
34. Do you allow SNAPSHOT dependencies in production?
    
35. How do you ensure reproducible builds?
    

---

# 20. Advanced Maven Questions

For **2–5+ years experience**, these can differentiate you from candidates who only know basic commands:

1. Explain Maven's reactor.
    
2. Explain dependency mediation.
    
3. Explain Maven's effective POM.
    
4. Explain inheritance vs aggregation.
    
5. Explain dependencyManagement internally.
    
6. Explain pluginManagement internally.
    
7. How does Maven resolve a dependency?
    
8. What happens internally when `mvn clean install` runs?
    
9. How does Maven determine lifecycle bindings?
    
10. How are packaging types associated with lifecycle phases?
    
11. How does Maven resolve transitive dependencies?
    
12. How does Maven choose between conflicting versions?
    
13. What is a BOM?
    
14. How does `import` scope work?
    
15. What is dependency convergence?
    
16. How do you enforce dependency convergence?
    
17. How does Maven handle SNAPSHOT metadata?
    
18. What happens when a SNAPSHOT is updated?
    
19. What is Maven metadata?
    
20. How does Maven resolve plugins?
    
21. What is plugin prefix resolution?
    
22. Difference between dependency repository and plugin repository.
    
23. How does mirror resolution work?
    
24. How does Maven authentication work?
    
25. How does Maven proxy configuration work?
    
26. What is Maven's local repository layout?
    
27. What is `.lastUpdated` in `.m2`?
    
28. Why can `.lastUpdated` files cause dependency-resolution issues?
    
29. What is an effective settings file?
    
30. How do you debug Maven at verbose/debug level?
    

For example:

```bash
mvn -X clean install
```

---

# 21. Spring Boot Project Management Questions

Interviewers may also move from Maven into **how you manage the actual Spring Boot project**:

1. Explain your Spring Boot project structure.
    
2. Why did you choose Maven?
    
3. How do you manage dependencies?
    
4. How do you manage dependency versions?
    
5. How do you manage environment configuration?
    
6. How do you package your application?
    
7. How do you create the deployable artifact?
    
8. How do you run the application locally?
    
9. How do you run it in QA?
    
10. How does the application reach production?
    
11. How does Maven participate in your CI/CD pipeline?
    
12. How do you handle unit tests?
    
13. How do you handle integration tests?
    
14. How do you handle code quality?
    
15. How do you handle security scanning?
    
16. How do you handle dependency vulnerabilities?
    
17. How do you upgrade Spring Boot?
    
18. What happens if a Spring Boot upgrade introduces dependency conflicts?
    
19. How do you roll back a release?
    
20. How do you version your application?
    
21. How do you identify which JAR version is deployed?
    
22. How do you make builds reproducible?
    
23. How do you handle production configuration?
    
24. How do you troubleshoot production startup failures?
    
25. How do you troubleshoot a build that passes locally but fails in production?
    

---

## 22. The 20 Questions You Should Definitely Master

If you don't have time to study all 300+ questions, **master these first**:

1. What is Maven and why do we use it?
    
2. Explain the Maven lifecycle.
    
3. Explain `mvn clean install`.
    
4. Difference between `package`, `install`, and `deploy`.
    
5. Explain `pom.xml`.
    
6. `dependencies` vs `dependencyManagement`.
    
7. Direct vs transitive dependency.
    
8. Explain dependency conflict resolution.
    
9. Explain Maven scopes.
    
10. How do you troubleshoot dependency-resolution failures?
    
11. What is `settings.xml`?
    
12. `repositories` vs `distributionManagement`.
    
13. What is Nexus/Artifactory?
    
14. How does Maven authenticate with an internal repository?
    
15. What is a Maven profile?
    
16. Maven profile vs Spring Boot profile.
    
17. What is `spring-boot-starter-parent`?
    
18. What does `spring-boot-maven-plugin` do?
    
19. How does `mvn package` create a runnable Spring Boot JAR?
    
20. **Walk me through your company's complete Spring Boot → Maven → Nexus/Artifactory → Jenkins → deployment flow.**
    

### The last one is particularly important.

A strong candidate should be able to draw something like:

```text
Developer
   │
   ▼
Git Repository
   │
   ▼
Jenkins / CI
   │
   ├── mvn clean
   ├── mvn compile
   ├── mvn test
   ├── mvn package
   │
   ▼
Spring Boot JAR
   │
   ▼
Internal Artifact Repository
(Nexus / Artifactory)
   │
   ▼
Deployment Pipeline
   │
   ▼
QA / UAT / Production
```

And explain **exactly what Maven does at each stage, where dependencies come from, where the artifact goes, how credentials are handled, and how the same artifact moves between environments**.

If you're preparing for a **Java/Spring Boot interview**, I can also turn this into a **"Maven + Spring Boot Interview — 100 Questions with detailed answers and real project scenarios"** format, starting from beginner and progressing to **3–5 years experienced-level questions**.