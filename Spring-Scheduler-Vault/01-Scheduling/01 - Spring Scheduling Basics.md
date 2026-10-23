---
tags: [scheduling, core, interview]
---

# Spring Scheduling Basics

Related: [[Home]] · [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] · [[05 - Retry Pattern]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Spring's scheduling support lets you run a method **automatically, on a timer** — no cron job, no external scheduler process, no manual trigger needed. It's built into Spring itself.

## 🔑 The Core Players

| Element | Role | Analogy |
|---|---|---|
| `@EnableScheduling` | Turns the feature ON for the whole app | The main power switch |
| `@Scheduled` | Marks *which* method runs on a timer | An alarm clock sticker on a method |
| `TaskScheduler` | The interface that actually fires the timer | The clock mechanism itself |
| `ScheduledTaskRegistrar` | Registers all `@Scheduled` methods with the scheduler | The person writing every alarm into the master calendar |
| `SchedulingConfigurer` | Interface to customize *how* tasks are registered/scheduled | Lets you swap the stock clock for a custom one |

## 🛠️ How it works — code

```java
@SpringBootApplication
@EnableScheduling   // 1. Turn scheduling on
public class ScheduledRun {
    public static void main(String[] args) {
        SpringApplication.run(ScheduledRun.class, args);
    }
}

@Service
public class BirthdayEmailScheduler {

    // Runs every day at 8:00 AM
    @Scheduled(cron = "0 0 8 * * *")
    public void sendBirthdayEmails() { ... }

    // Runs every 60,000ms measured from START of previous run
    @Scheduled(fixedRate = 60000)
    public void pollSomething() { ... }

    // Runs 60,000ms AFTER the previous run FINISHES
    @Scheduled(fixedDelay = 60000)
    public void syncSomething() { ... }
}
```

### `fixedRate` vs `fixedDelay` — the #1 interview trip-up
- **`fixedRate`**: measures interval from the **start** of one execution to the start of the next. If the job takes longer than the rate, executions can queue up / overlap (with a single-threaded scheduler, the next run just waits).
- **`fixedDelay`**: measures interval from the **end** of one execution to the start of the next. Guarantees a gap of rest between runs — safer for jobs whose duration varies.
- **`cron`**: for calendar-based schedules ("every weekday at 8am", "1st of the month").

## ⚙️ Customizing the Scheduler (`SchedulingConfigurer`)

By default, Spring uses a **single-threaded** scheduler — meaning all your `@Scheduled` methods share one thread and can block each other! In production you almost always override this:

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        registrar.setTaskScheduler(taskScheduler());
    }

    @Bean
    public TaskScheduler taskScheduler() {
        SimpleAsyncTaskScheduler scheduler = new SimpleAsyncTaskScheduler();
        scheduler.setVirtualThreads(true); // see [[02 - Virtual Threads and SimpleAsyncTaskScheduler]]
        return scheduler;
    }
}
```

See [[03 - Configuration Classes Overview]] for how this fits with `@Configuration`/`@Bean`.

## ❓ Interview Q&A

**Q: What happens if you forget `@EnableScheduling`?**
A: Your `@Scheduled` methods are silently ignored — no error, they just never run.

**Q: Is the default scheduler multi-threaded?**
A: No — by default Spring uses a single-threaded `TaskScheduler`, so long-running scheduled jobs will delay others. You fix this with a custom `TaskScheduler` bean (e.g., a thread-pool-backed one, or `SimpleAsyncTaskScheduler` with virtual threads).

**Q: Difference between `fixedRate` and `fixedDelay`?**
A: See the callout above — start-to-start vs end-to-start.

**Q: Can `@Scheduled` methods take parameters or return values?**
A: No. They must be `void` with no arguments, since nothing is "calling" them with data.

**Q: How would you make the cron expression configurable per environment?**
A: `@Scheduled(cron = "${scheduler.birthday.cron}")` and define the property in `application.yml`/`application.properties` — see `@Value` / `@ConfigurationProperties` in [[03 - Configuration Classes Overview]].
