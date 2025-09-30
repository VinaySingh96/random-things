# 🚀 Parallelization in Java

## 🌍 Why Parallelization?

In any application, work can be split into two categories:

1. **CPU-bound tasks**

   * Heavy calculations, image/video processing, ML models, cryptography.
   * The bottleneck is the CPU itself.
   * To speed up, we use **multi-core parallelism** (splitting the work across multiple CPU cores/threads).

2. **IO-bound tasks**

   * Waiting for network APIs, database queries, disk reads, message queues.
   * The bottleneck is **waiting on external systems**, not CPU.
   * To speed up, we **do not block threads** — instead we overlap multiple IO calls concurrently.

👉 Parallelization helps in both cases:

* For **CPU-bound** → divide work across cores.
* For **IO-bound** → issue multiple requests at once and process responses as they arrive.

---

## ⚙️ Ways to Achieve Parallelization in Java

### 1. **Multi-threading (classic way)**

* Java provides **threads**: each thread runs a separate task.
* Problems: managing threads manually is complex (synchronization, deadlocks).

### 2. **Thread pools / Executors**

* Instead of creating threads manually, Java provides `ExecutorService`.
* Example:

  ```java
  ExecutorService pool = Executors.newFixedThreadPool(10);
  pool.submit(() -> doWork());
  ```
* This makes parallelization **scalable** (fixed # of threads reused).

### 3. **`CompletableFuture` + `@Async` (Spring)**

* A higher-level abstraction for async work.
* You return a `CompletableFuture<T>` that will complete later.
* Spring’s `@Async` annotation makes it very easy to run methods in parallel with thread pools.

### 4. **Reactive Programming (Mono/Flux, Reactor, RxJava)**

* Instead of thread-per-task, it uses an **event loop** with non-blocking IO.
* A few threads can handle **thousands of concurrent IO-bound requests**.
* Values are modeled as **streams** (`Mono = single`, `Flux = many`).
* Built-in backpressure: consumer controls how much it can handle.

### 5. **Fork/Join Framework**

* Java’s built-in **divide and conquer** framework for CPU-bound tasks.
* Example: parallel array sorting.
* `ForkJoinPool` can recursively split work and join results.

---

## 🔀 Parallelization Approaches: High-Level View

| Approach                       | Best for                                        | Example                                   |
| ------------------------------ | ----------------------------------------------- | ----------------------------------------- |
| **Threads**                    | Low-level control                               | `new Thread(runnable).start()`            |
| **ExecutorService**            | General async pool management                   | `Executors.newFixedThreadPool(10)`        |
| **CompletableFuture / @Async** | Easy async APIs                                 | `@Async public CompletableFuture<T>`      |
| **Reactive (Mono/Flux)**       | Non-blocking IO, massive concurrency, streaming | `WebClient.get().retrieve().bodyToMono()` |
| **Fork/Join**                  | CPU-bound parallel computations                 | `pool.invoke(new RecursiveTask())`        |

---

## 🚦 Sequential vs Parallel

### Sequential

```java
String a = callServiceA(); // wait
String b = callServiceB(); // wait
return a + b;
```

⏱ Total time = `time(A) + time(B)`

### Parallel

```java
CompletableFuture<String> a = callServiceAAsync();
CompletableFuture<String> b = callServiceBAsync();
return a.thenCombine(b, (ra, rb) -> ra + rb);
```

⏱ Total time = `max(time(A), time(B))`

---

## 🟢 Benefits of Parallelization

* **Reduced latency**: do multiple tasks at once (e.g., fetch data from multiple APIs).
* **Better throughput**: handle more requests in the same time.
* **Resource utilization**: use all CPU cores effectively.
* **Scalability**: serve thousands of IO-bound requests with limited threads (reactive).

---

## 🔴 Challenges of Parallelization

* **Thread safety**: shared state must be synchronized.
* **Deadlocks**: careless locks can freeze your app.
* **Starvation**: one task hogs resources, blocking others.
* **Context switching overhead**: too many threads slow things down.
* **Debugging complexity**: async stack traces can be harder to follow.

---

## 🧭 Summary

* Parallelization = running **multiple tasks simultaneously** (CPU-bound or IO-bound).
* In Java, you can achieve it at different levels:

  * **Threads/Executors** → manual/imperative.
  * **CompletableFuture / @Async** → easy async in Spring.
  * **Reactor Mono/Flux** → scalable, non-blocking, streaming.
* Choice depends on:

  * Do you use **blocking libraries**? → `@Async`.
  * Do you use **non-blocking stack** (WebFlux, WebClient, R2DBC)? → `Mono/Flux`.
  * Is the work **CPU-heavy**? → Fork/Join or parallel streams.
---

## 1. Using `@Async` + `CompletableFuture`

### Setup

Enable async in Spring Boot:

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

### Service

```java
@Service
public class AsyncService {

    @Async
    public CompletableFuture<String> task1() throws InterruptedException {
        Thread.sleep(1000); // simulate blocking call
        return CompletableFuture.completedFuture("Result from Task1");
    }

    @Async
    public CompletableFuture<String> task2() throws InterruptedException {
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("Result from Task2");
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/async")
public class AsyncController {

    private final AsyncService asyncService;

    public AsyncController(AsyncService asyncService) {
        this.asyncService = asyncService;
    }

    @GetMapping("/parallel")
    public CompletableFuture<List<String>> runParallel() {
        CompletableFuture<String> f1 = asyncService.task1();
        CompletableFuture<String> f2 = asyncService.task2();

        return CompletableFuture.allOf(f1, f2)
                .thenApply(v -> List.of(f1.join(), f2.join()));
    }
}
```

🟢 Calls `task1()` and `task2()` **in parallel** on a thread pool, returns results once both complete.

---

## 2. Using `Mono` / `Flux` (Project Reactor)

### Service

```java
@Service
public class ReactiveService {

    public Mono<String> task1() {
        return Mono.fromCallable(() -> {
            Thread.sleep(1000); // simulate blocking call
            return "Result from Task1";
        }).subscribeOn(Schedulers.boundedElastic()); // offload blocking work
    }

    public Mono<String> task2() {
        return Mono.fromCallable(() -> {
            Thread.sleep(1000);
            return "Result from Task2";
        }).subscribeOn(Schedulers.boundedElastic());
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/reactive")
public class ReactiveController {

    private final ReactiveService reactiveService;

    public ReactiveController(ReactiveService reactiveService) {
        this.reactiveService = reactiveService;
    }

    @GetMapping("/parallel")
    public Mono<List<String>> runParallel() {
        return Mono.zip(reactiveService.task1(), reactiveService.task2())
                   .map(tuple -> List.of(tuple.getT1(), tuple.getT2()));
    }
}
```

🟢 Uses Reactor’s `Mono.zip()` (like `Promise.all` in Node.js) to run in parallel and combine results.

---

# ⚖️ Pros and Cons

| Feature                  | `@Async` + `CompletableFuture`                             | `Mono` / `Flux`                                                                      |
| ------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Conceptual model**     | Similar to Java `Future`, imperative style                 | Similar to Node.js `Promise` / RxJS, functional style                                |
| **Best for**             | Blocking calls (JDBC, REST via RestTemplate, file IO)      | Non-blocking calls (WebClient, R2DBC, streaming data)                                |
| **Ease of use**          | Easy to understand, minimal learning curve                 | Steeper learning curve (operators, schedulers)                                       |
| **Threading model**      | One thread per blocking task (configurable pool)           | Few threads handle many tasks via event-loop, offload blocking to `boundedElastic()` |
| **Parallel composition** | `CompletableFuture.allOf()`, `.thenCombine()`              | `Mono.zip()`, `Flux.merge()`, `flatMap()`                                            |
| **Backpressure**         | Not supported                                              | Built-in (Reactive Streams spec)                                                     |
| **Error handling**       | `.exceptionally()`, `.handle()`                            | `.onErrorResume()`, `.retry()`, `.doOnError()`                                       |
| **Integration**          | Works naturally with Spring MVC                            | Works naturally with Spring WebFlux                                                  |
| **Scalability**          | Limited by thread pool size, good for moderate concurrency | Scales to massive concurrency, great for IO-heavy apps                               |
| **Debugging**            | Familiar stack traces                                      | Can be tricky (use Reactor debug tools)                                              |

---

# 🧭 When to Use Which

✅ Use **`@Async` + `CompletableFuture`** if:

* You already use **Spring MVC** (blocking stack).
* Your dependencies are blocking (JDBC, JPA, `RestTemplate`).
* You just need **parallel API calls** or offloading some tasks to background threads.

✅ Use **`Mono` / `Flux`** if:

* You are building with **Spring WebFlux** (reactive stack).
* You use **non-blocking drivers** (`WebClient`, R2DBC, reactive Mongo, Kafka, etc.).
* You need **high concurrency** (thousands of requests) with low resource usage.
* You want **streaming APIs** or need **backpressure**.

---

# 📝 Key Takeaways

* `CompletableFuture` = **Java Promise**.
* `@Async` = easy thread-based parallelism, but scales less for huge concurrency.
* `Mono` = async **single value**, `Flux` = async **stream of values**.
* Reactor gives you advanced composition, streaming, and scalability, but requires **non-blocking stack** for full benefit.
* Both are valid, and you can even **bridge between them**:

  ```java
  Mono.fromFuture(completableFuture);  // CompletableFuture -> Mono
  mono.toFuture();                     // Mono -> CompletableFuture
  ```

---

# Determining the concurrency limit
---

## 1️⃣ Identify the type of workload

### a) CPU-bound tasks

* Tasks that mostly **consume CPU cycles**, like calculations, compression, image processing.
* **Threads ≈ # of CPU cores** (or cores × 1–2)
* Why: each thread is actively using CPU; too many threads → context switching overhead.

**Formula (approx):**

```
Optimal Threads = Number of cores * (1 + Wait time / Compute time)
```

* For CPU-bound, wait time ≈ 0 → Threads ≈ CPU cores

**Example:**

* 8-core CPU → ~8 threads for CPU-heavy work

---

### b) IO-bound tasks

* Tasks that mostly **wait on external systems**: HTTP calls, DB queries, file I/O.
* Threads can be much higher than CPU cores because most of the time threads are idle (waiting).

**Formula (approx):**

```
Optimal Threads = Number of cores * (1 + Average wait time / Average compute time)
```

* If wait time >> compute time → more threads possible

**Example:**

* CPU cores = 8
* Avg compute time = 50ms, avg wait time = 450ms → Wait/Compute = 9
* Threads = 8 × (1 + 9) = 80 threads

---

## 2️⃣ Consider the Thread Pool Executor

Spring `@Async` or `CompletableFuture` uses **thread pools**. You configure:

```java
ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
executor.setCorePoolSize(10);
executor.setMaxPoolSize(50);
executor.setQueueCapacity(200);
executor.initialize();
```

* **CorePoolSize** → always kept alive
* **MaxPoolSize** → max concurrent threads allowed
* **QueueCapacity** → tasks waiting when threads are busy

✅ Setting concurrency limit = `MaxPoolSize` for async tasks.

---

## 3️⃣ Reactive / Mono/Flux

* Non-blocking work (WebClient, R2DBC) doesn’t create one thread per task.
* Concurrency limit depends on:

  1. **Number of event-loop threads** (Netty defaults: #CPU cores × 2)
  2. **Scheduler limits** for blocking operations (`Schedulers.boundedElastic()` has default max threads ~10×CPU cores)
  3. **External system capacity** — if you send 1000 HTTP requests in parallel but API throttles at 50 rps, you need **rate limiting**

Example of limiting concurrency with Reactor:

```java
Flux.fromIterable(tasks)
    .flatMap(task -> callExternalApi(task), 10) // max 10 concurrent calls
    .collectList()
    .block();
```

* The last parameter `10` = concurrency limit.

---

## 4️⃣ External constraints

* **API rate limits** → don’t overwhelm third-party APIs
* **Database connection pool size** → too many parallel DB calls can exhaust connections
* **Memory limits** → each thread consumes stack memory (~512KB–1MB per thread)

✅ Rule of thumb: pick a concurrency limit based on **slowest/bottleneck resource** (CPU, DB, API, or memory).

---

## 5️⃣ Measuring & tuning

1. Start with **safe defaults**:

   * CPU-bound: Threads = #cores
   * IO-bound: Threads = #cores × 2–10
2. Monitor:

   * Thread pool usage
   * Latency, throughput
   * CPU & memory utilization
3. Adjust iteratively.
4. Use metrics:

   * Spring Boot Micrometer: track `TaskExecutor` queue size, active threads
   * Reactor: use `parallelism` and `Schedulers` metrics

---

## 6️⃣ Quick Cheat Sheet

| Workload              | Formula / Rule                                   | Example                                                |
| --------------------- | ------------------------------------------------ | ------------------------------------------------------ |
| CPU-bound             | Threads ≈ #CPU cores                             | 8 cores → 8 threads                                    |
| IO-bound              | Threads ≈ cores × (1 + wait/compute)             | 8 cores, wait/compute=9 → 80 threads                   |
| Reactive non-blocking | Event-loop threads + boundedElastic for blocking | Netty default = cores × 2, boundedElastic ~ 10 × cores |
| External APIs / DB    | Limit by API rate or DB pool                     | HTTP API allows 50 rps → max concurrency = 50          |

---

### ✅ Key takeaways

1. Concurrency limit = **max threads/tasks that can run in parallel without overwhelming CPU, memory, or external systems**.
2. CPU-bound → keep threads ~ cores.
3. IO-bound → threads can be many times cores, but watch memory and downstream limits.
4. Reactive → concurrency limit often controlled at operator level (`flatMap(..., concurrency)`) rather than threads.
5. Always **measure & tune** — theoretical formulas give starting point, not exact number.

---


