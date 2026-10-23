---
tags: [testing, junit, interview]
---

# Testing in Spring Boot

Related: [[01 - Spring Scheduling Basics]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Testing a scheduler module is tricky because the "trigger" is time, not a user click — so tests typically call the scheduled method **directly** rather than waiting for the clock.

## 🔑 Core Tools

| Tool | Role |
|---|---|
| `spring-boot-starter-test` | Bundles JUnit 5, Mockito, AssertJ, Spring Test |
| `@SpringBootTest` | Loads the **full Spring context** — an integration test |
| `@MockBean` | Replaces a real bean with a Mockito mock **inside the Spring context** |
| JUnit | The actual test-running framework (`@Test`, assertions, lifecycle annotations) |

## 🛠️ Testing a Scheduled Method (unit-style, no waiting for cron)

```java
@ExtendWith(MockitoExtension.class)
class BirthdayEmailSchedulerTest {

    @Mock BirthdayDBOperation birthdayDBOperation;
    @Mock MailService mailService;
    @InjectMocks BirthdayEmailScheduler scheduler;

    @Test
    void sendsEmailToEachEmployeeWithBirthdayToday() {
        when(birthdayDBOperation.findEmployeesWithBirthdayToday())
            .thenReturn(List.of(new Employee(1L, "Ana", "ana@x.com")));

        scheduler.sendBirthdayEmails(); // call directly — don't wait for the cron trigger!

        verify(mailService).send(eq("ana@x.com"), anyString(), anyString());
    }

    @Test
    void retriesOnTransientMailFailure() {
        when(birthdayDBOperation.findEmployeesWithBirthdayToday())
            .thenReturn(List.of(new Employee(1L, "Ana", "ana@x.com")));
        doThrow(new MailSendException("timeout"))
            .doNothing()
            .when(mailService).send(any(), any(), any());

        scheduler.sendBirthdayEmails();

        verify(mailService, times(2)).send(any(), any(), any()); // 1 failure + 1 retry
    }
}
```

## 🛠️ Integration Test with `@SpringBootTest` + `@MockBean`

```java
@SpringBootTest
class SchedulerIntegrationTest {

    @MockBean
    private FarEyeClient farEyeClient; // real HTTP call replaced with a mock

    @Autowired
    private FleetedgeScheduler fleetedgeScheduler;

    @Test
    void handlesFarEyeOutageGracefully() {
        when(farEyeClient.uploadTrip(any())).thenThrow(new RestClientException("down"));
        assertDoesNotThrow(() -> fleetedgeScheduler.syncTrips());
    }
}
```

## ❓ Interview Q&A

**Q: Why not just wait for `@Scheduled(cron=...)` to fire in a test?**
A: Slow, flaky, and hard to control. Instead, call the annotated method **directly** like any other public method — the `@Scheduled` annotation only affects *when Spring invokes it in production*, it doesn't change the method itself.

**Q: `@MockBean` vs plain Mockito `@Mock`?**
A: `@Mock` (with `MockitoExtension`) creates a mock outside of any Spring context — fast, true unit test, no Spring startup. `@MockBean` replaces a bean **inside the Spring `ApplicationContext`** — needed for `@SpringBootTest`-style integration tests where the real bean would otherwise be wired in (e.g., you don't want a test to make a real outbound call to `FarEyeClient`).

**Q: How would you test that a `@Retryable`/manual retry loop actually retries the correct number of times?**
A: Use Mockito's `doThrow(...).doThrow(...).doNothing()` chaining (or `thenThrow`/`thenReturn` for return-value methods) to simulate N failures followed by success, then verify the method was invoked exactly `MAX_RETRIES` (or fewer, if it succeeded early) times.

**Q: Why keep `@SpringBootTest` usage minimal in a large codebase?**
A: It boots the *entire* application context, which is slow and couples the test to a lot of infrastructure. Prefer slice tests (`@JdbcTest`, `@WebMvcTest`) or plain unit tests with Mockito wherever the full context isn't actually needed.
