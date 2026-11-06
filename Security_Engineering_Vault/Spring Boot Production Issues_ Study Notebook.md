# 🛠️ Spring Boot Production Issues: Understand, Debug, Fix[¶](#%F0%9F%9B%A0%EF%B8%8F-Spring-Boot-Production-Issues:-Understand,-Debug,-Fix)

A hands-on study notebook for the **top 15 priority issues** (plus a quick-reference for the other 35).

**How each issue is organised**

| Block | What it gives you |
| --- | --- |
| 🔥 Symptom | What you see in production |
| 🧠 Why it happens | The mental model, in plain words |
| 🔍 How to investigate | Exact logs / commands / metrics to check |
| 🐞 Buggy code → ✅ Fixed code | Java/Spring snippets |
| 🧪 Try it | A small **runnable Python simulation** so you *see* the bug happen |
| 🎤 Interview answer | A 6-step answer you can say out loud |

> Run the 🧪 cells (Shift+Enter). The Java snippets are for reading; the simulations reproduce the *behaviour* of each bug so the concept sticks.

---

## The 6-step answer framework (use for every incident)[¶](#The-6-step-answer-framework-%28use-for-every-incident%29)

1. **Problem**: what broke, who was impacted (users, revenue, SLA)?
2. **Investigation**: which logs, metrics, dashboards, alerts did you look at first?
3. **Root cause**: how did you *prove* it (not guess it)?
4. **Fix**: exact code / SQL / config change.
5. **Verification**: how you confirmed it's resolved (metrics back to normal, load test).
6. **Prevention**: alerts, tests, limits, code-review rules you added.

## Symptom → first move (triage cheat-sheet)[¶](#Symptom-%E2%86%92-first-move-%28triage-cheat-sheet%29)

| Symptom | First thing to check |
| --- | --- |
| App won't start | Last `Caused by:` in the stack trace |
| Requests hang / slow | Thread dump + Hikari active/pending metrics |
| 5xx spike | Logs by traceId, then the downstream dependency |
| CPU 100% | `top -H` → thread id → hex → thread dump |
| Memory climbing, then restart | Heap dump (`jmap`), GC logs |
| Kafka consumers behind | Consumer-group lag per partition |
| Works locally, fails in prod | Diff of config / env vars / data volume |

# 1️⃣ Database connection pool exhausted (HikariCP)[¶](#1%EF%B8%8F%E2%83%A3-Database-connection-pool-exhausted-%28HikariCP%29)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

```
HikariPool-1 - Connection is not available, request timed out after 30000ms.
```

APIs hang for 30 s then return 500. DB itself looks fine.

### 🧠 Why it happens[¶](#%F0%9F%A7%A0-Why-it-happens)

A pool is a **fixed number of seats** (default `maximumPoolSize=10`). A request borrows a connection and must give it back. It runs out when:

1. **Leak**: connection borrowed, never returned (no `close()`, missing try-with-resources).
2. **Slow queries / long transactions** hold connections too long.
3. **Remote calls inside `@Transactional`** (HTTP call while holding a DB connection).
4. Pool too small for traffic, or too big for the DB.

### 🔍 How to investigate[¶](#%F0%9F%94%8D-How-to-investigate)

- Actuator/Micrometer: `hikaricp.connections.active`, `.pending`, `.timeout`
- Turn on leak detection: `spring.datasource.hikari.leak-detection-threshold=20000` → logs a stack trace of the code that borrowed the connection.
- DB side: `SELECT * FROM pg_stat_activity WHERE state <> 'idle';` (Postgres) or `SHOW PROCESSLIST;` (MySQL) → long-running / "idle in transaction".
- Thread dump: many threads in `HikariPool.getConnection`.

### 🐞 Buggy code[¶](#%F0%9F%90%9E-Buggy-code)

```
@Transactional
public Order placeOrder(OrderRequest req) {
    Order o = orderRepo.save(new Order(req));
    paymentClient.charge(req);        // ❌ 5-second HTTP call holding a DB connection
    return o;
}

// leak
Connection c = dataSource.getConnection();
ResultSet rs = c.createStatement().executeQuery("select ...");  // ❌ never closed
```

### ✅ Fix[¶](#%E2%9C%85-Fix)

```
// 1. Always close
try (Connection c = dataSource.getConnection();
     PreparedStatement ps = c.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) { ... }

// 2. Keep transactions short: do remote calls OUTSIDE the transaction
public Order placeOrder(OrderRequest req) {
    PaymentResult p = paymentClient.charge(req);      // no DB connection held
    return orderService.saveOrder(req, p);            // @Transactional only here
}
```

```
spring.datasource.hikari:
  maximum-pool-size: 20
  connection-timeout: 5000        # fail fast instead of 30s
  leak-detection-threshold: 20000
  max-lifetime: 1740000           # < DB/firewall idle timeout
```

**Sizing rule of thumb**: `pool ≈ (core_count × 2) + effective_spindle_count`. Bigger is *not* better.

### 🧪 Try it: a leak drains the pool[¶](#%F0%9F%A7%AA-Try-it:-a-leak-drains-the-pool)

In \[1\]:

```
import queue

class Pool:
    def __init__(self, size):
        self.q = queue.Queue()
        for i in range(size): self.q.put(f"conn-{i}")
    def get(self, timeout=0.05):
        try: return self.q.get(timeout=timeout)
        except queue.Empty: raise TimeoutError("Connection is not available, request timed out")
    def release(self, c): self.q.put(c)
    def available(self): return self.q.qsize()

def handle_request(pool, leak):
    c = pool.get()
    try:
        pass  # run query
    finally:
        if not leak:          # leaky code forgets to release
            pool.release(c)

for leak in (True, False):
    pool, failures = Pool(5), 0
    for i in range(20):
        try: handle_request(pool, leak)
        except TimeoutError: failures += 1
    print(f"leak={leak!s:5} | failed requests: {failures:2}/20 | free connections: {pool.available()}")
```

```
leak=True  | failed requests: 15/20 | free connections: 0
leak=False | failed requests:  0/20 | free connections: 5
```

**Take-away:** with the leak, requests fail after the 5th, even though traffic is tiny. Without it, the pool is never exhausted.

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Orders started timing out with *Connection is not available*. Hikari's `pending` metric was high and `active` was pinned at max. Leak detection pointed at a method calling a payment API inside `@Transactional`, so connections were held \~5 s each. I moved the remote call outside the transaction, set a 5 s connection timeout, and right-sized the pool. Active connections dropped to \~20% under load test; I added an alert on `pending > 0` for 1 minute."

# 2️⃣ N+1 query problem[¶](#2%EF%B8%8F%E2%83%A3-N+1-query-problem)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

One endpoint is slow; SQL log shows **hundreds of near-identical `select`s**.

### 🧠 Why it happens[¶](#%F0%9F%A7%A0-Why-it-happens)

You load N parents with **1** query, then touching a lazy child collection on each parent fires **N** more queries → **N+1**.

### 🔍 How to investigate[¶](#%F0%9F%94%8D-How-to-investigate)

- `spring.jpa.show-sql=true` or better `logging.level.org.hibernate.SQL=DEBUG`
- Hibernate statistics: `spring.jpa.properties.hibernate.generate_statistics=true`
- Test with `datasource-proxy` / `QuickPerf` to assert query count.

### 🐞 Buggy code[¶](#%F0%9F%90%9E-Buggy-code)

```
List<Author> authors = authorRepo.findAll();            // 1 query
authors.forEach(a -> a.getBooks().size());               // ❌ N queries
```

### ✅ Fixes[¶](#%E2%9C%85-Fixes)

```
// Fetch join
@Query("select distinct a from Author a join fetch a.books")
List<Author> findAllWithBooks();

// Entity graph
@EntityGraph(attributePaths = "books")
List<Author> findAll();

// Batch fetching (good for many collections)
spring.jpa.properties.hibernate.default_batch_fetch_size=50

// Or project to a DTO and avoid entities altogether
@Query("select new com.x.AuthorDto(a.name, count(b)) from Author a left join a.books b group by a.name")
```

⚠️ Fetch-joining **two** collections → `MultipleBagFetchException` / cartesian product. Use batch size or separate queries. `@ManyToOne` defaults to EAGER, so set `fetch = LAZY` explicitly.

### 🧪 Try it: count the queries[¶](#%F0%9F%A7%AA-Try-it:-count-the-queries)

In \[2\]:

```
import sqlite3

class CountingDB:
    def __init__(self):
        self.conn = sqlite3.connect(":memory:"); self.count = 0
    def q(self, sql, *a):
        self.count += 1
        return self.conn.execute(sql, a).fetchall()

db = CountingDB()
db.conn.executescript('''
create table author(id integer primary key, name text);
create table book(id integer primary key, author_id int, title text);
''')
for i in range(1, 101):
    db.conn.execute("insert into author values (?,?)", (i, f"Author {i}"))
    for j in range(3): db.conn.execute("insert into book(author_id,title) values (?,?)", (i, f"Book {i}-{j}"))

# N+1
db.count = 0
authors = db.q("select id, name from author")
for aid, name in authors:
    db.q("select title from book where author_id=?", aid)
print("N+1 queries      :", db.count)

# Fetch join equivalent
db.count = 0
rows = db.q("select a.id, a.name, b.title from author a left join book b on b.author_id=a.id")
print("JOIN FETCH query :", db.count)

# Batch fetching equivalent (IN clause)
db.count = 0
authors = db.q("select id from author")
ids = [a[0] for a in authors]
db.q(f"select author_id, title from book where author_id in ({','.join('?'*len(ids))})", *ids)
print("batch (IN) query :", db.count)
```

```
N+1 queries      : 101
JOIN FETCH query : 1
batch (IN) query : 2
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "The orders list API took 6 s. SQL logging showed 1 query for orders plus \~400 for order items. Root cause was lazy loading inside a loop. I used an `@EntityGraph` for the list endpoint and a DTO projection for the summary endpoint, and set `default_batch_fetch_size`. Queries dropped from 401 to 2, latency from 6 s to 300 ms. I added a test asserting max query count so it can't regress."

# 3️⃣ Slow API response[¶](#3%EF%B8%8F%E2%83%A3-Slow-API-response)

### 🧠 Where time can go[¶](#%F0%9F%A7%A0-Where-time-can-go)

```
Client → LB → [Tomcat thread wait] → Controller → Service → [DB query] → [External API] → Serialization → Client
```

Never guess; **measure each segment**.

### 🔍 How to investigate[¶](#%F0%9F%94%8D-How-to-investigate)

1. APM / tracing (Micrometer Tracing, Zipkin, New Relic, Datadog): which span is fat?
2. `http.server.requests` metric (p95/p99, not average)
3. DB: slow query log + `EXPLAIN ANALYZE`
4. Thread dump: are threads waiting on DB, socket, or lock?
5. Payload size, since large JSON serialisation can cost real time.

### Common culprits & fixes[¶](#Common-culprits-&-fixes)

| Cause | Fix |
| --- | --- |
| Missing index / full table scan | Add index, rewrite query |
| N+1 | See #2 |
| Sequential external calls | `CompletableFuture` / parallel calls with timeouts |
| Huge payload | Pagination, DTOs, compression |
| Repeated expensive reads | Cache with TTL (`@Cacheable`) |

### 🧪 Try it: an index changes the query plan[¶](#%F0%9F%A7%AA-Try-it:-an-index-changes-the-query-plan)

In \[3\]:

```
import sqlite3, time, random
conn = sqlite3.connect(":memory:")
conn.execute("create table orders(id integer primary key, customer_id int, status text, amount real)")
conn.executemany("insert into orders(customer_id,status,amount) values (?,?,?)",
    [(random.randint(1, 20000), random.choice(["NEW","PAID","SHIPPED"]), random.random()*100) for _ in range(300_000)])
conn.commit()

def timed(label):
    t = time.perf_counter()
    conn.execute("select * from orders where customer_id=12345").fetchall()
    print(f"{label:12} {(time.perf_counter()-t)*1000:8.2f} ms |",
          conn.execute("explain query plan select * from orders where customer_id=12345").fetchall()[0][-1])

timed("no index")
conn.execute("create index idx_orders_customer on orders(customer_id)")
timed("with index")
```

```
no index        11.91 ms | SCAN orders
with index       0.06 ms | SEARCH orders USING INDEX idx_orders_customer (customer_id=?)
```

Read the plan: **SCAN** = reads every row (bad on big tables). **SEARCH ... USING INDEX** = jumps straight to matches.

> In Postgres/MySQL use `EXPLAIN ANALYZE`, and look for `Seq Scan`, high `rows`, and sorts spilling to disk.

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "p99 on `/orders/search` hit 8 s. Tracing showed 92% of time in one SQL. `EXPLAIN ANALYZE` showed a sequential scan over 20M rows because the filter column wasn't indexed. I added a composite index created concurrently, rewrote the `OFFSET` pagination to keyset, and p99 fell to 180 ms. I added a slow-query alert at 1 s."

# 4️⃣ OutOfMemoryError[¶](#4%EF%B8%8F%E2%83%A3-OutOfMemoryError)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

`java.lang.OutOfMemoryError: Java heap space` (or `GC overhead limit exceeded`, `Metaspace`, `unable to create native thread`). Pod restarts (OOMKilled in Kubernetes).

### 🧠 Why it happens[¶](#%F0%9F%A7%A0-Why-it-happens)

Objects are still **reachable** so the GC can't free them. Typical sources:

- **Unbounded caches / static maps / lists**
- Loading huge result sets (`findAll()` on millions of rows)
- Large file upload/export built entirely in memory
- ThreadLocals never cleared; listeners never removed
- Too small `-Xmx` for the real workload

### 🔍 How to investigate[¶](#%F0%9F%94%8D-How-to-investigate)

```
# Auto-capture on crash (put in JVM options)
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/dumps

# Manual
jcmd <pid> GC.heap_dump /tmp/heap.hprof
jmap -histo:live <pid> | head -20       # which classes dominate?

# GC behaviour
-Xlog:gc*:file=gc.log
```

Open `.hprof` in **Eclipse MAT** → *Leak Suspects* → "Dominator Tree" → follow the **path to GC roots**.

Kubernetes: set `-XX:MaxRAMPercentage=75` so heap + off-heap fits in the container limit.

### 🐞 Buggy code[¶](#%F0%9F%90%9E-Buggy-code)

```
private static final Map<String, Report> CACHE = new HashMap<>();   // ❌ grows forever
public Report get(String key) { return CACHE.computeIfAbsent(key, this::build); }
```

### ✅ Fix[¶](#%E2%9C%85-Fix)

```
Cache<String, Report> cache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(10))
        .build();
// Stream large data instead of loading it: Pageable, @QueryHints(fetchSize), Stream<T>
```

### 🧪 Try it: unbounded vs bounded cache[¶](#%F0%9F%A7%AA-Try-it:-unbounded-vs-bounded-cache)

In \[4\]:

```
from collections import OrderedDict

class LRU:
    def __init__(self, cap): self.cap, self.d = cap, OrderedDict()
    def put(self, k, v):
        self.d[k] = v; self.d.move_to_end(k)
        if len(self.d) > self.cap: self.d.popitem(last=False)

unbounded, bounded = {}, LRU(1000)
for i in range(200_000):             # 200k *distinct* keys (e.g. per-user / per-request)
    unbounded[f"user-{i}"] = "x" * 100
    bounded.put(f"user-{i}", "x" * 100)

print("unbounded entries:", len(unbounded))
print("bounded entries  :", len(bounded.d))
```

```
unbounded entries: 200000
bounded entries  : 1000
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "A service restarted every \~6 hours with OOM. I enabled heap dump on OOM and opened it in MAT: one `HashMap` held 70% of the heap, a cache keyed by request id with no eviction. I replaced it with Caffeine (max size + TTL), verified heap stayed flat over a soak test, and added alerts on heap-after-GC and GC pause time."

# 5️⃣ High CPU utilization[¶](#5%EF%B8%8F%E2%83%A3-High-CPU-utilization)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

CPU at 90-100%, latency up, maybe the autoscaler goes crazy.

### 🧠 Common causes[¶](#%F0%9F%A7%A0-Common-causes)

Infinite/very hot loops · bad regex (catastrophic backtracking) · excessive GC (really a memory problem) · serialising huge objects · busy-wait polling · too many threads context-switching.

### 🔍 The classic 4-step Linux recipe[¶](#%F0%9F%94%8D-The-classic-4-step-Linux-recipe)

```
top -H -p <pid>                  # 1. find the hottest THREAD id (decimal)
printf "%x\n" <tid>              # 2. convert to hex  (e.g. 12345 -> 3039)
jstack <pid> > dump.txt          # 3. take a thread dump (take 3, 5s apart)
grep -A 30 "nid=0x3039" dump.txt # 4. see what that thread is doing
```

- Same method appears in all 3 dumps → that's your hot spot.
- If the hot threads are **"GC Task Thread"** → it's a memory problem (#4).
- Use async-profiler / JFR for flame graphs: `jcmd <pid> JFR.start duration=60s filename=cpu.jfr`

### 🧪 Try it: a "regex bomb"[¶](#%F0%9F%A7%AA-Try-it:-a-%22regex-bomb%22)

In \[5\]:

```
import re, time
pattern = re.compile(r"^(a+)+$")         # nested quantifiers = catastrophic backtracking
for n in (16, 20, 24):
    s = "a" * n + "b"
    t = time.perf_counter(); pattern.match(s)
    print(f"input length {n+1:3}: {time.perf_counter()-t:7.3f} s")
```

```
input length  17:   0.003 s
input length  21:   0.040 s
```

```
input length  25:   0.713 s
```

Time roughly **doubles for each extra character**, so one malicious input pins a CPU core. The thread dump would show the thread sitting in `java.util.regex.Pattern$...`.

**Fix:** simpler regex / atomic groups / length limits / validate with libraries.

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "CPU hit 100% on two pods after a release. `top -H` gave one hot thread; converting its id to hex and finding it in three thread dumps showed it inside a regex in an input validator. A crafted input caused backtracking. I rewrote the pattern, capped input length, and added a unit test with a pathological string."

# 6️⃣ Thread pool exhaustion[¶](#6%EF%B8%8F%E2%83%A3-Thread-pool-exhaustion)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

App "freezes": no CPU, no errors, but requests queue and time out. Health check may even fail.

### 🧠 Why it happens[¶](#%F0%9F%A7%A0-Why-it-happens)

Tomcat has \~200 worker threads (`server.tomcat.threads.max`). If every thread is **blocked** (slow DB, slow downstream with no timeout, lock), no thread is free for new requests. Same for your own `ExecutorService` / `@Async` pool, whose queue can fill and reject (`RejectedExecutionException`).

### 🔍 How to investigate[¶](#%F0%9F%94%8D-How-to-investigate)

- Thread dump: hundreds of threads `http-nio-8080-exec-*` in `WAITING`/`TIMED_WAITING` or `RUNNABLE` in `SocketInputStream.read` → same call stack.
- Metrics: `tomcat.threads.busy`, executor `active/queue.size`.

### 🐞 Buggy code[¶](#%F0%9F%90%9E-Buggy-code)

```
// no timeout -> thread waits forever if downstream hangs
RestTemplate rt = new RestTemplate();

@Bean Executor ex() { return Executors.newFixedThreadPool(10); }   // ❌ unbounded queue hides overload
```

### ✅ Fix[¶](#%E2%9C%85-Fix)

```
// timeouts everywhere
SimpleClientHttpRequestFactory f = new SimpleClientHttpRequestFactory();
f.setConnectTimeout(2000); f.setReadTimeout(3000);

// bounded, observable executor with back-pressure
ThreadPoolTaskExecutor ex = new ThreadPoolTaskExecutor();
ex.setCorePoolSize(10); ex.setMaxPoolSize(20); ex.setQueueCapacity(100);
ex.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
```

Also: bulkhead (separate pools per dependency), circuit breaker (#7), or non-blocking WebClient.

### 🧪 Try it: slow downstream starves the pool[¶](#%F0%9F%A7%AA-Try-it:-slow-downstream-starves-the-pool)

In \[6\]:

```
import time
from concurrent.futures import ThreadPoolExecutor

def request(downstream_seconds):
    time.sleep(downstream_seconds)      # thread is BLOCKED waiting on downstream
    return "ok"

def run(label, downstream_s, workers=4, requests=16):
    t = time.perf_counter()
    with ThreadPoolExecutor(max_workers=workers) as pool:
        list(pool.map(lambda _: request(downstream_s), range(requests)))
    print(f"{label:28} total time for {requests} requests: {time.perf_counter()-t:5.2f}s")

run("healthy downstream (0.05s)", 0.05)
run("slow downstream (0.5s)", 0.5)      # same code, 10x slower dependency -> throughput collapses
```

```
healthy downstream (0.05s)   total time for 16 requests:  0.20s
```

```
slow downstream (0.5s)       total time for 16 requests:  2.00s
```

Throughput = `threads / latency`. Slow dependency → latency up → throughput down → queue grows. **Timeouts + circuit breakers cap the damage.**

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "The service stopped responding though CPU was idle. Thread dumps showed all 200 Tomcat threads blocked reading from a payment API that had no read timeout. I added 2 s/3 s timeouts, a Resilience4j circuit breaker with fallback, and a bulkhead for that client; I also alerted on `tomcat.threads.busy > 80%`."

# 7️⃣ Downstream microservice unavailable → Circuit breaker[¶](#7%EF%B8%8F%E2%83%A3-Downstream-microservice-unavailable-%E2%86%92-Circuit-breaker)

### 🧠 The idea[¶](#%F0%9F%A7%A0-The-idea)

Like an electrical breaker: after too many failures, **stop calling** the broken service for a while (fail fast / fallback), then **probe** to see if it recovered.

```
CLOSED --(failure rate > threshold)--> OPEN --(wait duration)--> HALF_OPEN --(probes succeed)--> CLOSED
                                                                   |--(probes fail)--> OPEN
```

### ✅ Resilience4j in Spring Boot[¶](#%E2%9C%85-Resilience4j-in-Spring-Boot)

```
@CircuitBreaker(name = "inventory", fallbackMethod = "fallback")
@Retry(name = "inventory")
@TimeLimiter(name = "inventory")
public Stock getStock(String sku) { return inventoryClient.get(sku); }

private Stock fallback(String sku, Throwable t) { return Stock.unknown(sku); }
```

```
resilience4j.circuitbreaker.instances.inventory:
  sliding-window-size: 20
  failure-rate-threshold: 50
  wait-duration-in-open-state: 10s
  permitted-number-of-calls-in-half-open-state: 3
resilience4j.retry.instances.inventory:
  max-attempts: 3
  wait-duration: 200ms
  enable-exponential-backoff: true
```

⚠️ Retry only **idempotent** calls, and use backoff + jitter, otherwise retries amplify the outage (retry storm).

### 🔍 Investigate[¶](#%F0%9F%94%8D-Investigate)

`/actuator/circuitbreakers`, `resilience4j_circuitbreaker_state` metric, logs of `CallNotPermittedException`.

### 🧪 Try it: a tiny circuit breaker[¶](#%F0%9F%A7%AA-Try-it:-a-tiny-circuit-breaker)

In \[7\]:

```
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=3, open_seconds=0.3):
        self.state, self.fails, self.opened_at = "CLOSED", 0, 0
        self.th, self.open_s = failure_threshold, open_seconds
    def call(self, fn, fallback):
        if self.state == "OPEN":
            if time.time() - self.opened_at >= self.open_s: self.state = "HALF_OPEN"
            else: return fallback("circuit open (fast fail)")
        try:
            r = fn(); self.state, self.fails = "CLOSED", 0; return r
        except Exception as e:
            self.fails += 1
            if self.state == "HALF_OPEN" or self.fails >= self.th:
                self.state, self.opened_at = "OPEN", time.time()
            return fallback(f"error: {e}")

service_up = False
def downstream():
    if not service_up: raise ConnectionError("503")
    return "real data"

cb = CircuitBreaker()
for i in range(8):
    print(i, cb.state.ljust(9), "->", cb.call(downstream, lambda why: f"FALLBACK ({why})"))
service_up = True; time.sleep(0.35)      # downstream recovers, wait period elapses
for i in range(8, 11):
    print(i, cb.state.ljust(9), "->", cb.call(downstream, lambda why: f"FALLBACK ({why})"))
```

```
0 CLOSED    -> FALLBACK (error: 503)
1 CLOSED    -> FALLBACK (error: 503)
2 CLOSED    -> FALLBACK (error: 503)
3 OPEN      -> FALLBACK (circuit open (fast fail))
4 OPEN      -> FALLBACK (circuit open (fast fail))
5 OPEN      -> FALLBACK (circuit open (fast fail))
6 OPEN      -> FALLBACK (circuit open (fast fail))
7 OPEN      -> FALLBACK (circuit open (fast fail))
```

```
8 OPEN      -> real data
9 CLOSED    -> real data
10 CLOSED    -> real data
```

Notice: after 3 failures, calls are **rejected instantly** (the broken service is left alone to recover), then a half-open probe succeeds and it closes.

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "When the inventory service went down, our checkout threads piled up waiting and checkout failed too, a cascading failure. I added a time limiter, circuit breaker and a cached-stock fallback via Resilience4j. In the next outage checkout degraded gracefully instead of failing."

# 8️⃣ API timeout issues[¶](#8%EF%B8%8F%E2%83%A3-API-timeout-issues)

### 🧠 Three different timeouts: don't mix them up[¶](#%F0%9F%A7%A0-Three-different-timeouts:-don%27t-mix-them-up)

| Timeout | Meaning | Error you see |
| --- | --- | --- |
| **Connect** | Time to establish TCP connection | `ConnectTimeoutException` |
| **Read / Socket** | Time waiting for response bytes after connecting | `SocketTimeoutException: Read timed out` |
| **Server / Gateway** | Proxy or LB gives up (nginx 504, API gateway) | `504 Gateway Timeout` |

### Golden rule: **timeouts must shrink as you go deeper**[¶](#Golden-rule:-timeouts-must-shrink-as-you-go-deeper)

```
Client 10s > Gateway 8s > Service A 6s > Service B 4s > DB 2s
```

If the downstream timeout is *longer* than the caller's, the caller gives up while work continues wastefully (and may double-execute on retry).

### ✅ Configuration examples[¶](#%E2%9C%85-Configuration-examples)

```
// WebClient
HttpClient hc = HttpClient.create()
    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 2000)
    .responseTimeout(Duration.ofSeconds(3));

// RestClient / RestTemplate (Spring Boot 3.x)
@Bean RestClient rc(RestClient.Builder b) {
    var f = new JdkClientHttpRequestFactory(HttpClient.newBuilder().connectTimeout(Duration.ofSeconds(2)).build());
    f.setReadTimeout(Duration.ofSeconds(3));
    return b.requestFactory(f).build();
}

// Feign
spring.cloud.openfeign.client.config.default.connect-timeout=2000
spring.cloud.openfeign.client.config.default.read-timeout=3000

// DB query timeout
spring.jpa.properties.jakarta.persistence.query.timeout=3000
```

### 🔍 Investigate[¶](#%F0%9F%94%8D-Investigate)

Which timeout fired (message text) → which hop → trace the request across services → was the downstream still working?

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Users saw random 504s. Gateway timeout was 30 s but Service A's read timeout to B was 60 s, so the gateway gave up while A kept waiting. I aligned timeouts decreasing downward, set explicit connect/read timeouts on every client, and added retries only for idempotent GETs."

# 9️⃣ Transaction rolled back (or *didn't*) unexpectedly[¶](#9%EF%B8%8F%E2%83%A3-Transaction-rolled-back-%28or-didn%27t%29-unexpectedly)

### 🧠 The 5 classic `@Transactional` traps[¶](#%F0%9F%A7%A0-The-5-classic-@Transactional-traps)

| # | Trap | Why |
| --- | --- | --- |
| 1 | **Checked exception doesn't roll back** | Default rollback only for `RuntimeException`/`Error` |
| 2 | **Self-invocation** | `this.method()` bypasses the Spring proxy → no transaction |
| 3 | **Non-public / `final` method** | Proxy can't intercept |
| 4 | **Exception swallowed** with try/catch | Spring never sees it → commits |
| 5 | **Wrong propagation** | `REQUIRED` inner failure marks outer tx rollback-only → `UnexpectedRollbackException` |

### 🐞 Buggy code[¶](#%F0%9F%90%9E-Buggy-code)

```
@Service
public class OrderService {
    @Transactional
    public void create(Order o) throws IOException {
        repo.save(o);
        throw new IOException("boom");       // ❌ checked → COMMITS anyway
    }

    public void process(Order o) {
        this.create(o);                       // ❌ self-invocation: no proxy, no transaction
    }

    @Transactional
    public void outer() {
        try { inner.doWork(); }               // inner is @Transactional (REQUIRED) and throws
        catch (Exception e) { log.warn("ignored"); }
        // ❌ outer tx is already marked rollback-only → UnexpectedRollbackException at commit
    }
}
```

### ✅ Fixes[¶](#%E2%9C%85-Fixes)

```
@Transactional(rollbackFor = Exception.class)
public void create(Order o) throws IOException { ... }

// call through another bean (or inject self proxy) so the proxy is used
orderTxService.create(o);

// independent unit of work
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void audit(...) { ... }
```

### 🔍 Investigate[¶](#%F0%9F%94%8D-Investigate)

`logging.level.org.springframework.transaction=TRACE` and `org.springframework.orm.jpa=DEBUG` shows *"Creating new transaction"*, *"Participating in existing"*, *"Initiating rollback"*.

### 🧪 Try it: proxy-style simulation of self-invocation[¶](#%F0%9F%A7%AA-Try-it:-proxy-style-simulation-of-self-invocation)

In \[8\]:

```
class Proxy:
    '''Mimics Spring AOP: transaction logic wraps calls made THROUGH the proxy only.'''
    def __init__(self, target): self._t = target; target._self_proxy = self
    def __getattr__(self, name):
        fn = getattr(self._t, name)
        if getattr(fn, "_tx", False):
            def wrapped(*a, **k):
                print(f"  [TX BEGIN] {name}")
                try: r = fn(*a, **k); print(f"  [TX COMMIT] {name}"); return r
                except Exception as e: print(f"  [TX ROLLBACK] {name}: {e}"); raise
            return wrapped
        return fn

def transactional(fn): fn._tx = True; return fn

class OrderService:
    @transactional
    def create(self): print("  saving order"); 
    def process(self): self.create()        # self-invocation: bypasses proxy

svc = Proxy(OrderService())
print("Calling create() through proxy:");        svc.create()
print("Calling process() -> this.create():");    svc.process()
```

```
Calling create() through proxy:
  [TX BEGIN] create
  saving order
  [TX COMMIT] create
Calling process() -> this.create():
  saving order
```

In the second call there's **no `TX BEGIN`**: the exact production bug when `@Transactional` "doesn't work".

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Order rows were saved but the payment record was missing after a failure. The method threw a checked exception, so Spring committed. I set `rollbackFor = Exception.class`, moved a self-invoked method into its own bean, and added an integration test asserting rollback."

# 🔟 Duplicate API requests → **Idempotency**[¶](#%F0%9F%94%9F-Duplicate-API-requests-%E2%86%92-Idempotency)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

Customer charged twice; two identical orders; duplicate rows.

### 🧠 Why[¶](#%F0%9F%A7%A0-Why)

Client retries on timeout, user double-clicks, gateway/consumer retries. **The first request may have succeeded**, the client just didn't hear back.

### ✅ Solution: idempotency key[¶](#%E2%9C%85-Solution:-idempotency-key)

1. Client sends `Idempotency-Key: <uuid>` header.
2. Server stores `(key → response)` with a **unique constraint**.
3. Same key again → return the stored response, don't re-execute.

```
@PostMapping("/payments")
public ResponseEntity<Payment> pay(@RequestHeader("Idempotency-Key") String key, @RequestBody PayReq req) {
    return idempotencyService.execute(key, () -> paymentService.charge(req));
}
```

```
CREATE TABLE idempotency_keys (
  key         VARCHAR(64) PRIMARY KEY,   -- unique constraint is what makes it safe under concurrency
  request_hash CHAR(64),
  response    JSONB,
  created_at  TIMESTAMP DEFAULT now()
);
```

Insert-first approach: `INSERT ... ON CONFLICT DO NOTHING`; if 0 rows inserted, someone else owns it → wait/return their result. **Check-then-insert without a unique constraint is racy.**

### 🧪 Try it: retries without vs with idempotency[¶](#%F0%9F%A7%AA-Try-it:-retries-without-vs-with-idempotency)

In \[9\]:

```
import threading

charges_naive, charges_safe = [], []
store, lock = {}, threading.Lock()

def charge_naive(req_id):
    charges_naive.append(req_id)                 # executes every time
    return "charged"

def charge_idempotent(key):
    with lock:                                   # stands in for DB unique constraint
        if key in store: return store[key]
        charges_safe.append(key); store[key] = "charged"
        return store[key]

# Client times out and retries the same payment 3 times
for _ in range(3): charge_naive("order-42")
for _ in range(3): charge_idempotent("order-42")
print("naive      -> customer charged", len(charges_naive), "times")
print("idempotent -> customer charged", len(charges_safe), "time")
```

```
naive      -> customer charged 3 times
idempotent -> customer charged 1 time
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Customers were double-charged when the mobile app retried after timeouts. I introduced an `Idempotency-Key` header stored with a unique constraint; repeated keys return the original response. I also added a unique business constraint on `(order_id, payment_type)` as a last line of defence."

# 1️⃣1️⃣ Kafka consumer lag increasing[¶](#1%EF%B8%8F%E2%83%A31%EF%B8%8F%E2%83%A3-Kafka-consumer-lag-increasing)

### 🧠 Concepts[¶](#%F0%9F%A7%A0-Concepts)

- **Lag** = latest offset (produced) − committed offset (consumed), per partition.
- Max parallelism of a consumer group = **number of partitions**. Extra consumers sit idle.

### 🔍 Investigate[¶](#%F0%9F%94%8D-Investigate)

```
kafka-consumer-groups.sh --bootstrap-server b:9092 --describe --group orders-cg
# check LAG per partition: is it ALL partitions (slow processing) or ONE (hot key / stuck message)?
```

Metrics: `records-lag-max`, processing time per record, rebalances (`rebalance` log lines), GC pauses.

### Causes → fixes[¶](#Causes-%E2%86%92-fixes)

| Cause | Fix |
| --- | --- |
| Slow processing per record (DB/HTTP call) | Optimise, batch DB writes, async with care |
| Too few consumers/partitions | Add partitions + consumers (partitions ≥ consumers) |
| Poison message blocks one partition | Retry with backoff → DLT (dead-letter topic) |
| `max.poll.interval.ms` exceeded → constant rebalancing | Lower `max.poll.records` or raise interval |
| Hot partition (skewed key) | Better partition key |

```
spring.kafka.listener.concurrency: 6            # threads per instance
spring.kafka.consumer.max-poll-records: 100
spring.kafka.consumer.properties.max.poll.interval.ms: 300000
```

### 🧪 Try it: how much lag, and how many consumers do I need?[¶](#%F0%9F%A7%AA-Try-it:-how-much-lag,-and-how-many-consumers-do-I-need?)

In \[10\]:

```
import math

produce_rate   = 1000   # msgs/sec arriving
per_consumer   = 150    # msgs/sec one consumer can process
partitions     = 6

def lag_after(seconds, consumers):
    active = min(consumers, partitions)             # idle consumers beyond #partitions
    drain = active * per_consumer
    return max(0, (produce_rate - drain) * seconds)

for c in (2, 4, 6, 8):
    print(f"{c} consumers -> capacity {min(c,partitions)*per_consumer:4}/s | lag after 10 min: {lag_after(600,c):>8,}")

needed = math.ceil(produce_rate / per_consumer)
print(f"\nNeed {needed} consumers to keep up, but only {partitions} partitions -> add partitions OR speed up processing.")
```

```
2 consumers -> capacity  300/s | lag after 10 min:  420,000
4 consumers -> capacity  600/s | lag after 10 min:  240,000
6 consumers -> capacity  900/s | lag after 10 min:   60,000
8 consumers -> capacity  900/s | lag after 10 min:   60,000

Need 7 consumers to keep up, but only 6 partitions -> add partitions OR speed up processing.
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Order-event lag grew to millions. Lag was on every partition, so it was throughput, not a stuck message. Per-record DB calls were the bottleneck. I batched inserts, raised partitions from 6 to 12 and concurrency to match, and alerted on lag growth rate. Lag cleared in 20 minutes."

# 1️⃣2️⃣ Kafka duplicate messages[¶](#1%EF%B8%8F%E2%83%A32%EF%B8%8F%E2%83%A3-Kafka-duplicate-messages)

### 🧠 Why duplicates are *normal*[¶](#%F0%9F%A7%A0-Why-duplicates-are-normal)

Kafka default is **at-least-once**: if a consumer processes a message but crashes/rebalances **before committing the offset**, the message is redelivered. Producers can also duplicate on retry (fixable with `enable.idempotence=true`).

### ✅ Fix: make the **consumer idempotent**[¶](#%E2%9C%85-Fix:-make-the-consumer-idempotent)

```
@KafkaListener(topics = "payments")
@Transactional
public void on(PaymentEvent e) {
    if (processedRepo.existsByEventId(e.getId())) return;     // dedupe
    paymentService.apply(e);
    processedRepo.save(new ProcessedEvent(e.getId()));         // same transaction as business write
}
```

- Unique constraint on `processed_events.event_id` guards against concurrent duplicates.
- Producer: `enable.idempotence=true`, `acks=all`.
- Prefer **natural idempotency**: "set status = PAID" instead of "add 100 to balance".
- For DB→Kafka consistency use the **transactional outbox** pattern.

### 🧪 Try it[¶](#%F0%9F%A7%AA-Try-it)

In \[11\]:

```
balance, processed = {"naive": 0, "idem": 0}, set()
events = [("evt-1", 100), ("evt-2", 50), ("evt-1", 100), ("evt-3", 25), ("evt-2", 50)]   # redeliveries

for eid, amt in events:
    balance["naive"] += amt                    # blindly applies duplicates
    if eid in processed: continue
    processed.add(eid); balance["idem"] += amt # applies once

print("expected balance: 175")
print("naive consumer  :", balance["naive"])
print("idempotent      :", balance["idem"])
```

```
expected balance: 175
naive consumer  : 325
idempotent      : 175
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Wallet balances were occasionally too high after consumer restarts. Kafka redelivered uncommitted messages. I made the consumer idempotent with a `processed_events` table written in the same transaction as the balance update, plus a unique constraint, and enabled idempotent producers."

# 1️⃣3️⃣ Database deadlocks[¶](#1%EF%B8%8F%E2%83%A33%EF%B8%8F%E2%83%A3-Database-deadlocks)

### 🔥 Symptom[¶](#%F0%9F%94%A5-Symptom)

```
Deadlock found when trying to get lock; try restarting transaction     (MySQL)
ERROR: deadlock detected                                                (Postgres)
```

### 🧠 Why[¶](#%F0%9F%A7%A0-Why)

Two transactions each hold a lock the other needs:

```
Tx A: locks row 1 → wants row 2
Tx B: locks row 2 → wants row 1     💥
```

The DB kills one (victim) and throws `DeadlockLoserDataAccessException` / `CannotAcquireLockException`.

### ✅ Fixes[¶](#%E2%9C%85-Fixes)

1. **Consistent lock ordering**: always update rows in the same order (e.g., sort by id).
2. **Short transactions**: lock late, commit early.
3. Proper indexes (fewer rows locked).
4. **Retry** the victim transaction (it's safe if idempotent).
5. Lower isolation level / use optimistic locking where acceptable.

```
// consistent ordering
List<Long> ids = Stream.of(fromId, toId).sorted().toList();
List<Account> accounts = repo.findAllByIdInOrderById(ids);   // with PESSIMISTIC_WRITE

@Retryable(retryFor = CannotAcquireLockException.class, maxAttempts = 3, backoff = @Backoff(delay = 100, multiplier = 2))
@Transactional
public void transfer(...) { ... }
```

### 🔍 Investigate[¶](#%F0%9F%94%8D-Investigate)

- Postgres: `deadlock_timeout`, server log shows both queries involved.
- MySQL: `SHOW ENGINE INNODB STATUS\G` → *LATEST DETECTED DEADLOCK*.

### 🧪 Try it: opposite lock order deadlocks, same order doesn't[¶](#%F0%9F%A7%AA-Try-it:-opposite-lock-order-deadlocks,-same-order-doesn%27t)

In \[12\]:

```
import threading, time

def transfer(first, second, name, results, timeout=0.5):
    if not first.acquire(timeout=timeout): results[name] = "gave up"; return
    time.sleep(0.1)                                   # do some work while holding first lock
    got = second.acquire(timeout=timeout)
    results[name] = "OK" if got else "DEADLOCK (timed out waiting)"
    if got: second.release()
    first.release()

def scenario(label, order_a, order_b):
    r, L = {}, {1: threading.Lock(), 2: threading.Lock()}
    ta = threading.Thread(target=transfer, args=(L[order_a[0]], L[order_a[1]], "Tx A", r))
    tb = threading.Thread(target=transfer, args=(L[order_b[0]], L[order_b[1]], "Tx B", r))
    ta.start(); tb.start(); ta.join(); tb.join()
    print(f"{label:26}", r)

scenario("opposite order (1->2, 2->1)", (1, 2), (2, 1))
scenario("same order     (1->2, 1->2)", (1, 2), (1, 2))
```

```
opposite order (1->2, 2->1) {'Tx B': 'DEADLOCK (timed out waiting)', 'Tx A': 'OK'}
```

```
same order     (1->2, 1->2) {'Tx A': 'OK', 'Tx B': 'OK'}
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "Fund transfers deadlocked at peak. `SHOW ENGINE INNODB STATUS` showed A→B and B→A transfers locking accounts in opposite order. I sorted account ids before locking, shortened the transaction, and added a bounded retry on `CannotAcquireLockException`. Deadlocks went to zero."

# 1️⃣4️⃣ JWT authentication failures (401 / 403)[¶](#1%EF%B8%8F%E2%83%A34%EF%B8%8F%E2%83%A3-JWT-authentication-failures-%28401-/-403%29)

### 401 vs 403[¶](#401-vs-403)

| Code | Meaning | Typical cause |
| --- | --- | --- |
| **401 Unauthorized** | "I don't know who you are" | Missing/expired/invalid token, bad signature, wrong issuer/audience |
| **403 Forbidden** | "I know you, but you can't do this" | Missing role/authority, wrong `ROLE_` prefix, method security |

### 🔍 Checklist when tokens "suddenly fail"[¶](#%F0%9F%94%8D-Checklist-when-tokens-%22suddenly-fail%22)

1. **Expired** (`exp`) and **clock skew** between servers → allow \~30-60 s leeway.
2. **Signature**: wrong secret/public key, key rotation, `kid` not found (JWKS cache).
3. `iss` / `aud` mismatch between environments (dev token in prod).
4. Filter order: custom JWT filter not before `UsernamePasswordAuthenticationFilter`.
5. Header format: must be `Authorization: Bearer <token>` (no extra quotes/spaces).
6. **Authorities mapping**: roles claim isn't converted → every call is 403.
7. CORS preflight (OPTIONS) blocked and shows up like auth failure.
8. Proxy/gateway stripping the `Authorization` header.

```
// Spring Security 6 resource server: map the claim to authorities
@Bean JwtAuthenticationConverter jwtAuthConverter() {
    var granted = new JwtGrantedAuthoritiesConverter();
    granted.setAuthoritiesClaimName("roles");
    granted.setAuthorityPrefix("ROLE_");
    var conv = new JwtAuthenticationConverter();
    conv.setJwtGrantedAuthoritiesConverter(granted);
    return conv;
}
```

```
logging.level.org.springframework.security: DEBUG    # shows which filter rejected the request
```

### 🧪 Try it: decode a JWT (never trust, just inspect) and check expiry[¶](#%F0%9F%A7%AA-Try-it:-decode-a-JWT-%28never-trust,-just-inspect%29-and-check-expiry)

In \[13\]:

```
import base64, json, time

def b64url(d): return base64.urlsafe_b64encode(json.dumps(d).encode()).rstrip(b"=").decode()
def b64url_decode(s): return json.loads(base64.urlsafe_b64decode(s + "=" * (-len(s) % 4)))

# Build a fake (unsigned) token that expired 10 minutes ago
token = ".".join([b64url({"alg": "HS256", "typ": "JWT"}),
                  b64url({"sub": "alice", "roles": ["USER"], "iss": "auth.dev", "exp": int(time.time()) - 600}),
                  "fake-signature"])

header, payload, _ = token.split(".")
claims = b64url_decode(payload)
print("claims:", claims)

now = time.time(); leeway = 60
print("expired?        ", claims["exp"] + leeway < now, f"({int(now - claims['exp'])}s ago)")
print("has ADMIN role? ", "ADMIN" in claims["roles"], "-> would be 403 on admin endpoints")
```

```
claims: {'sub': 'alice', 'roles': ['USER'], 'iss': 'auth.dev', 'exp': 1793634616}
expired?         True (600s ago)
has ADMIN role?  False -> would be 403 on admin endpoints
```

Tip: paste a token into jwt.io (use test tokens only) to read `exp`, `iss`, `roles` quickly during an incident.

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "After a release all mobile users got 401. Logs at DEBUG showed `Jwt expired`, but tokens were fresh. The auth server clock had drifted \~2 minutes, so `iat` was in the future. I fixed NTP, set a 60 s clock-skew tolerance and added an alert on 401 rate."

# 1️⃣5️⃣ Application startup failure[¶](#1%EF%B8%8F%E2%83%A35%EF%B8%8F%E2%83%A3-Application-startup-failure)

### 🧠 Golden rule[¶](#%F0%9F%A7%A0-Golden-rule)

**Scroll to the bottom of the stack trace and read the last `Caused by:`.** The top is Spring noise; the real reason is the deepest cause.

### Decoder ring[¶](#Decoder-ring)

| Message | Meaning | Fix |
| --- | --- | --- |
| `NoSuchBeanDefinitionException` | Bean not found | Missing `@Component/@Service`, outside component-scan package, wrong `@Profile`, missing `@Bean` |
| `NoUniqueBeanDefinitionException` | 2+ candidates | `@Primary` or `@Qualifier` |
| `BeanCurrentlyInCreationException` | Circular dependency | Refactor; constructor → extract shared logic; (last resort) `@Lazy` |
| `UnsatisfiedDependencyException` | Wrapper: look deeper | Read the nested `Caused by` |
| `Could not resolve placeholder 'x'` | Property missing | Check profile file / env var / Config Server |
| `Web server failed to start. Port 8080 was already in use` | Port conflict | `server.port`, or `lsof -i :8080` and kill |
| `Failed to configure a DataSource: 'url' attribute is not specified` | No DB config / driver | Set `spring.datasource.*`, check profile active |
| `FlywayException: Validate failed` | Migration checksum/order mismatch | Never edit applied migrations; `repair` carefully |
| `ClassNotFoundException / NoSuchMethodError` | Dependency version conflict | `mvn dependency:tree`, align BOM versions |
| `UnsupportedClassVersionError` | Built with newer Java than runtime | Align JDK |

### Config precedence (high → low) — why "works locally, fails in prod"[¶](#Config-precedence-%28high-%E2%86%92-low%29-%E2%80%94-why-%22works-locally,-fails-in-prod%22)

```
1. Command-line args            (--server.port=9000)
2. SPRING_APPLICATION_JSON / JVM -D properties
3. OS environment variables     (SPRING_DATASOURCE_URL)
4. application-{profile}.yml    (outside jar, then inside)
5. application.yml
6. @PropertySource / defaults
```

### 🔍 Debug toolkit[¶](#%F0%9F%94%8D-Debug-toolkit)

```
java -jar app.jar --debug                      # condition evaluation report (why auto-config on/off)
java -jar app.jar --spring.profiles.active=prod
curl localhost:8080/actuator/env               # effective properties (secure it!)
curl localhost:8080/actuator/beans
```

### 🐞 Circular dependency & fix[¶](#%F0%9F%90%9E-Circular-dependency-&-fix)

```
@Service class A { A(B b) {} }      // A needs B
@Service class B { B(A a) {} }      // B needs A   ❌ BeanCurrentlyInCreationException

// Fix: extract the shared logic
@Service class Shared {}
@Service class A { A(Shared s) {} }
@Service class B { B(Shared s) {} }
```

### 🧪 Try it: detect a dependency cycle (what Spring does at startup)[¶](#%F0%9F%A7%AA-Try-it:-detect-a-dependency-cycle-%28what-Spring-does-at-startup%29)

In \[14\]:

```
def find_cycle(graph):
    visiting, done = [], set()
    def dfs(n):
        if n in visiting: return visiting[visiting.index(n):] + [n]
        if n in done: return None
        visiting.append(n)
        for dep in graph.get(n, []):
            c = dfs(dep)
            if c: return c
        visiting.pop(); done.add(n)
    for n in graph:
        c = dfs(n)
        if c: return c

beans = {"orderService": ["paymentService"], "paymentService": ["notificationService"],
         "notificationService": ["orderService"], "userService": []}
cycle = find_cycle(beans)
print("Cycle found:", " -> ".join(cycle) if cycle else "none")

beans["notificationService"] = ["eventPublisher"]         # break it with an event-based design
print("After fix  :", find_cycle(beans) or "no cycle")
```

```
Cycle found: orderService -> paymentService -> notificationService -> orderService
After fix  : no cycle
```

### 🎤 Interview answer[¶](#%F0%9F%8E%A4-Interview-answer)

> "A release failed to start in prod only. The deepest `Caused by` was *Could not resolve placeholder 'payment.url'*. The `application-prod.yml` was missing it and a Kubernetes env var was misnamed. I fixed the config, added `@ConfigurationProperties` with `@Validated` so missing values fail early with a clear message, and added a startup smoke test in the pipeline."

# 📚 Quick reference: the other 35 issues[¶](#%F0%9F%93%9A-Quick-reference:-the-other-35-issues)

Use this table as a **first-move + typical-fix** cheat sheet. When you have time, expand each into the 6-step story.

## Startup & configuration[¶](#Startup-&-configuration)

| # | Issue | First move | Typical fix |
| --- | --- | --- | --- |
| 3 | NoSuchBeanDefinition | Check package of `@SpringBootApplication`, active profiles | Move class under scan root / add `@ComponentScan` / fix `@Profile` |
| 4 | Works locally, fails in prod | Diff env vars, config, JDK/dependency versions, DB/data | Same container image everywhere; externalise config |
| 5 | Wrong profile | `--debug` log: "The following profiles are active" | Set `SPRING_PROFILES_ACTIVE` explicitly |
| 6 | Config not loading | `/actuator/env` to see the winning source | Fix precedence; `@ConfigurationProperties` over scattered `@Value` |
| 7 | Port in use | `lsof -i :PORT` / `netstat` | Stop other process or change `server.port` |
| 8 | Starts but APIs unavailable | `server.servlet.context-path`, security rules, `/actuator/health`, ingress route | Fix path/route/readiness probe |

## REST / HTTP[¶](#REST-/-HTTP)

| # | Issue | First move | Typical fix |
| --- | --- | --- | --- |
| 9 | 500 error | Find stack trace by traceId | Handle in `@RestControllerAdvice`; fix root cause |
| 10 | 400 | Log validation errors / `HttpMessageNotReadableException` | `@Valid`, correct DTO types, date formats |
| 11 | 404 | Check mapping, HTTP verb, context path, trailing slash | Correct route; gateway rewrite rules |
| 12/13 | 401 / 403 | See #14 | Token / authorities mapping |
| 15 | Timeouts | See #8 | Aligned timeouts |
| 16 | Duplicate requests | See #10 | Idempotency keys |
| 17 | JSON issues | Check Jackson errors | `@JsonIgnore`/DTOs for cycles, `JavaTimeModule`, ISO dates |
| 18 | Huge payloads | Check response size | Pagination, field filtering, `server.compression.enabled=true` |

## Database[¶](#Database)

| # | Issue | First move | Typical fix |
| --- | --- | --- | --- |
| 20 | Too many DB connections | `SELECT count(*) FROM pg_stat_activity` × instances | pool size × pods ≤ DB `max_connections`; use PgBouncer |
| 21 | Slow SQL | `EXPLAIN ANALYZE`, slow query log | Index, rewrite, keyset pagination |
| 23 | `LazyInitializationException` | Where is the lazy field touched, outside the session? | Fetch join / DTO mapping inside the service; avoid `open-in-view` as a crutch |
| 25 | `OptimisticLockException` | Two updates on same row | `@Version`, catch + retry or return 409 |
| 27 | Data inconsistency | Check tx boundaries across multiple writes | Single transaction / saga / outbox |
| 28 | Duplicate rows | Race between check and insert | Unique constraint + handle `DataIntegrityViolationException` |
| 29 | Migration failed | Flyway history table, error line | Forward-only migrations; expand/contract for backward compatibility |
| 30 | Query fails only in prod | Compare schema, collation, SQL mode, data volume | Mirror prod data shape in staging |

## Microservices / messaging[¶](#Microservices-/-messaging)

| # | Issue | First move | Typical fix |
| --- | --- | --- | --- |
| 32 | External API slow | Tracing span + client timeouts | Timeouts, cache, async, circuit breaker |
| 33 | Cascading failure | Which service failed first? | Circuit breaker, bulkhead, rate limiting |
| 34 | Inter-service call fails | DNS, service name, port, TLS, auth headers | Fix discovery/config; Feign/WebClient error decoders |
| 37 | Message processing fails | Look at error + offset | Retry w/ backoff → DLT; fix poison message |
| 38 | Distributed inconsistency | Which step succeeded/failed? | Saga + compensation, transactional outbox |
| 39 | Service discovery failure | Eureka registry / Consul health / DNS TTL | Heartbeat config, health checks, client-side cache refresh |
| 40 | Gateway 502/503/504 | Gateway logs vs downstream logs | Downstream health, routing, gateway timeouts, retries |

## Performance / concurrency[¶](#Performance-/-concurrency)

| # | Issue | First move | Typical fix |
| --- | --- | --- | --- |
| 43 | Frequent GC | GC logs: pause time, allocation rate | Reduce allocation, right-size heap, G1/ZGC tuning |
| 45 | App unresponsive | 3 thread dumps | Find deadlock/blocked threads, add timeouts |
| 46 | Latency under load | Load test + profiler | Remove bottleneck (DB, locks, pool sizes) |
| 47 | Race condition | Intermittent wrong data | `synchronized`/`Atomic*`/DB locking/idempotent design |
| 48 | Cache memory leak | Heap dump → biggest map | Max size + TTL (Caffeine) |
| 49 | Crash under traffic | Pod restarts, limits | Autoscaling, rate limiting, resource limits, load shedding |
| 50 | Logs fill disk | `du -sh /var/log` | Rotation (`logback` `maxHistory`, `totalSizeCap`), centralised logging, log level hygiene |

### Bonus: 🧪 race condition demo (#47) and optimistic locking (#25)[¶](#Bonus:-%F0%9F%A7%AA-race-condition-demo-%28%2347%29-and-optimistic-locking-%28%2325%29)

In \[15\]:

```
import threading, time

# --- Race condition: read-modify-write is NOT atomic
counter = {"v": 0}
def unsafe():
    for _ in range(20_000):
        v = counter["v"]; time.sleep(0); counter["v"] = v + 1   # thread switch can happen between read and write

lock, safe_counter = threading.Lock(), {"v": 0}
def safe():
    for _ in range(20_000):
        with lock: safe_counter["v"] += 1

for fn, c, label in ((unsafe, counter, "unsafe"), (safe, safe_counter, "with lock")):
    ts = [threading.Thread(target=fn) for _ in range(4)]
    [t.start() for t in ts]; [t.join() for t in ts]
    print(f"{label:10} expected 80000, got {c['v']}")

# --- Optimistic locking: update only if version unchanged
row = {"balance": 100, "version": 1}
def update(read_version, new_balance):
    if row["version"] != read_version:
        return "CONFLICT (OptimisticLockException) -> retry or HTTP 409"
    row.update(balance=new_balance, version=read_version + 1); return "updated"

print("\nUser A reads v1, User B reads v1")
print("A saves:", update(1, 120))
print("B saves:", update(1, 150))
```

```
unsafe     expected 80000, got 20000
with lock  expected 80000, got 80000

User A reads v1, User B reads v1
A saves: updated
B saves: CONFLICT (OptimisticLockException) -> retry or HTTP 409
```

*(In Java, `count++` on a shared field loses updates the same way. Fixes: `AtomicInteger`, `synchronized`, or DB-level locking.)*

# 🧰 Production debugging toolbox[¶](#%F0%9F%A7%B0-Production-debugging-toolbox)

| Need | Tool / Command |
| --- | --- |
| Health & metrics | Spring Boot Actuator, Micrometer → Prometheus → Grafana |
| Logs | ELK / Splunk / Loki; correlate with `traceId` (MDC) |
| Tracing | Micrometer Tracing + Zipkin / Jaeger / OpenTelemetry |
| Thread dump | `jstack <pid>`, `jcmd <pid> Thread.print`, `kill -3 <pid>` |
| Heap dump | `jcmd <pid> GC.heap_dump file.hprof` → Eclipse MAT |
| Profiling | JFR, async-profiler, VisualVM |
| DB | `EXPLAIN ANALYZE`, slow query log, `pg_stat_activity`, `SHOW PROCESSLIST` |
| Kafka | `kafka-consumer-groups.sh --describe`, Kafka UI, lag exporter |
| Kubernetes | `kubectl logs/describe pod`, `kubectl top`, OOMKilled events |

### Actuator essentials[¶](#Actuator-essentials)

```
management:
  endpoints.web.exposure.include: health,metrics,prometheus,threaddump,heapdump,env,loggers
  endpoint.health.probes.enabled: true
```

⚠️ Secure these endpoints, since `env` and `heapdump` leak sensitive data.

### Handy log-level tweaks at runtime (no restart)[¶](#Handy-log-level-tweaks-at-runtime-%28no-restart%29)

```
POST /actuator/loggers/org.hibernate.SQL   {"configuredLevel":"DEBUG"}
```

# ✍️ Your practice template[¶](#%E2%9C%8D%EF%B8%8F-Your-practice-template)

Copy this cell for each issue. Fill it in **out loud** until you can tell it in \~2 minutes.

```
ISSUE:
1. Problem (impact):
2. Investigation (logs/metrics/commands):
3. Root cause (how I proved it):
4. Fix (code/SQL/config):
5. Verification:
6. Prevention (alerts/tests/limits):
```

## 🎯 Self-check questions[¶](#%F0%9F%8E%AF-Self-check-questions)

1. Why can a *healthy* database still show "connection not available"?
2. Why does `@Transactional` not roll back on `IOException`?
3. How do you find which thread burns CPU from the OS level?
4. Why can't you scale a Kafka consumer group beyond the partition count?
5. Why must timeouts shrink as you go deeper into the call chain?
6. What makes an idempotency key safe under concurrent requests?
7. Difference between 401 and 403, and 3 reasons for a sudden wave of 401s?
8. Why does fetch-joining two collections break, and what's the alternative?

## 🗺️ Suggested study order[¶](#%F0%9F%97%BA%EF%B8%8F-Suggested-study-order)

1. # 15 Startup → #9 Transactions (Spring fundamentals)[¶](#15-Startup-%E2%86%92-%239-Transactions-%28Spring-fundamentals%29)
2. # 1 Pool, #2 N+1, #3 Slow API (database layer)[¶](#1-Pool,-%232-N+1,-%233-Slow-API-%28database-layer%29)
3. # 4 OOM, #5 CPU, #6 Thread pool (JVM)[¶](#4-OOM,-%235-CPU,-%236-Thread-pool-%28JVM%29)
4. # 7 Circuit breaker, #8 Timeouts, #10 Idempotency (resilience)[¶](#7-Circuit-breaker,-%238-Timeouts,-%2310-Idempotency-%28resilience%29)
5. # 11/#12 Kafka, #13 Deadlocks, #14 JWT[¶](#11/%2312-Kafka,-%2313-Deadlocks,-%2314-JWT)
6. Skim the quick-reference table, then do mock interviews with the 6-step framework.