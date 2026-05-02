# Deutsche Bank — Interview Questions & Answers

**Role:** Java Full Stack Developer  
**Interview Type:** Technical + Logical + SQL

---

## Java & Multithreading

### Q1: What is API latency?

**A:** API latency is the time from when a client sends a request to when it receives a response. Includes network latency, server processing time, and data transfer time.

**Optimization techniques:** Caching, load balancing, CDNs, async processing, code/query optimization, compression.

---

### Q2: What is Java Reflection API?

**A:** Reflection allows inspecting and modifying the behavior of classes, methods, and fields at runtime.

```java
Class<?> clazz = Class.forName("com.example.MyClass");
Method method = clazz.getMethod("myMethod");
method.invoke(clazz.newInstance());
```

**Use cases:** Frameworks (Spring DI), testing, serialization libraries.  
**Caution:** Performance overhead, breaks encapsulation.

---

### Q3: Why do we use Executor Framework?

**A:** Managing threads manually is error-prone. Executor Framework provides:
- Thread pool management
- Task submission and lifecycle management
- Configurable pool sizes, rejection policies

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> processRequest());
executor.shutdown();
```

---

### Q4: How to limit max concurrent API requests in Spring Boot?

**A:** Use a `Semaphore` to limit concurrency:

```java
private final Semaphore semaphore = new Semaphore(10);

@GetMapping("/process")
public ResponseEntity<?> process() {
    semaphore.acquire();
    try {
        return ResponseEntity.ok(doWork());
    } finally {
        semaphore.release();
    }
}
```

**Alternative:** Use Resilience4j `Bulkhead` pattern in microservices.

---

### Q5: ThreadPoolExecutor vs ScheduledThreadPoolExecutor?

**A:**

| Feature | ThreadPoolExecutor | ScheduledThreadPoolExecutor |
|---------|-------------------|---------------------------|
| Purpose | Execute tasks in a pool | Execute tasks with delay or periodically |
| Methods | `execute()`, `submit()` | `schedule()`, `scheduleAtFixedRate()` |
| Use case | REST API processing, batch jobs | Cron-like tasks, polling, retry timers |

---

### Q6: What is Semaphore?

**A:** A concurrency primitive that controls access to a shared resource via permits.

```java
Semaphore sem = new Semaphore(3); // max 3 concurrent threads
sem.acquire();  // blocks if no permits available
try { /* critical section */ }
finally { sem.release(); }
```

**Use cases:** Connection pool limiting, rate limiting, resource throttling.

---

### Q7: What is the volatile keyword?

**A:** `volatile` ensures visibility across threads — every read sees the latest write. Does NOT guarantee atomicity.

```java
private volatile boolean running = true;
```

**When to use:** Flags, status variables read by multiple threads.  
**When NOT sufficient:** `count++` (need `AtomicInteger` or `synchronized`).

---

### Q8: Thread-safe collection classes?

**A:** `ConcurrentHashMap`, `CopyOnWriteArrayList`, `CopyOnWriteArraySet`, `ConcurrentLinkedQueue`, `ConcurrentLinkedDeque`, `BlockingQueue` implementations (`ArrayBlockingQueue`, `LinkedBlockingQueue`).

---

### Q9: How to make a collection synchronized?

**A:**
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
```

**Better option:** Use `ConcurrentHashMap` over `synchronizedMap` for concurrent access.

---

## Java 8 & Streams

### Q10: Stream API — sum of squares of even numbers > 10

```java
int sum = list.stream()
    .filter(n -> n % 2 == 0 && n > 10)
    .mapToInt(n -> n * n)
    .sum();
```

---

### Q11: Functional interfaces in Java?

**A:** `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`, `UnaryOperator<T>`, `BinaryOperator<T>`, `Comparator<T>`, `Runnable`, `Callable<V>`.

---

### Q12: Default methods in interfaces?

**A:** Added in Java 8 to allow adding new methods to interfaces without breaking existing implementations.

```java
interface Sortable {
    default void sort() {
        System.out.println("Default sort");
    }
}
```

**Why:** Backward compatibility, interface evolution (e.g., `List.sort()`, `Collection.stream()`).

---

## SQL & Database

### Q13: Find employee with highest code commits per department

**SQL:**
```sql
SELECT e.name, d.dept_name, c.commit_count
FROM employee e
JOIN department d ON e.dept_id = d.id
JOIN (
    SELECT emp_id, COUNT(*) as commit_count
    FROM commits GROUP BY emp_id
) c ON e.id = c.emp_id
WHERE c.commit_count = (
    SELECT MAX(c2.commit_count)
    FROM commits c2
    JOIN employee e2 ON c2.emp_id = e2.id
    WHERE e2.dept_id = e.dept_id
);
```

---

### Q14: Sort HashMap by values (descending)

```java
Map<String, Integer> sorted = map.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .collect(Collectors.toMap(
        Map.Entry::getKey, Map.Entry::getValue,
        (e1, e2) -> e1, LinkedHashMap::new));
```

---

### Q15: Many-to-Many relationship (Student & Teacher)

```java
@Entity
public class Student {
    @ManyToMany
    @JoinTable(name = "student_teacher",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "teacher_id"))
    private Set<Teacher> teachers;
}

@Entity
public class Teacher {
    @ManyToMany(mappedBy = "teachers")
    private Set<Student> students;
}
```

---

## System Design & Caching

### Q16: What are caching strategies?

| Strategy | Description |
|----------|-------------|
| **Cache-Aside** | App checks cache first, loads from DB on miss, writes to cache |
| **Write-Through** | Writes go to cache AND DB simultaneously |
| **Write-Behind** | Writes go to cache, async flush to DB |
| **Read-Through** | Cache loads from DB on miss automatically |
| **Refresh-Ahead** | Proactively refresh before expiry |

---

### Q17: How does Kafka know when to discard messages?

**A:** Via **retention policies:**
- `retention.ms` — Time-based (default 7 days)
- `retention.bytes` — Size-based per partition
- `cleanup.policy=compact` — Keep only latest per key
- Consumer offset — Messages before committed offset are eligible for cleanup

---

## Logical & Problem Solving

### Q18: 10x10x10 painted cube — how many cubes have no painted face?

**A:** Inner cubes = (10-2)³ = 8³ = **512 cubes**.

Remove outer layer on each side (2 faces per dimension), leaving 8x8x8 inner cubes.

---

### Q19: Sort binary array (0s and 1s)

```java
// Two-pointer approach - O(n)
int left = 0, right = arr.length - 1;
while (left < right) {
    if (arr[left] == 1 && arr[right] == 0) {
        arr[left] = 0; arr[right] = 1;
    }
    if (arr[left] == 0) left++;
    if (arr[right] == 1) right--;
}
```

---

### Q20: Trees vs Graphs — 5 differences

| Aspect | Tree | Graph |
|--------|------|-------|
| Structure | Hierarchical | Network |
| Cycles | No cycles | Can have cycles |
| Root | Has root node | No root concept |
| Edges | N-1 edges for N nodes | Any number of edges |
| Traversal | DFS/BFS from root | DFS/BFS from any node |

---

## Version Control & General

### Q21: How do you use GitHub as a developer?

**A:** GitHub is a cloud-hosted Git platform for version control and collaboration → it allows developers to create repositories, track commits/branches, raise pull requests, perform code reviews, manage issues, and integrate CI/CD pipelines → it enables teams to work on code simultaneously without overwriting each other, and serves as a portfolio to showcase projects; key use cases include open source contribution, automated testing via Actions, and documentation hosting.

---

### Q22: How do you initialize a local Git repository?

**A:** A local Git repo is a version-controlled folder on your machine → you initialize it using `git init`, then stage files with `git add .`, commit with `git commit -m "message"`, and optionally push to a remote with `git remote add origin <url>` followed by `git push -u origin main` → this establishes a full commit history locally and optionally syncs with GitHub/GitLab for collaboration and backup.

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

---

### Q23: What is Frontend and Backend Communication?

**A:** Frontend (browser/UI) and backend (server/database) are separate layers in a web application → they communicate over HTTP/HTTPS using REST APIs or GraphQL; the frontend sends requests (GET, POST, etc.) and the backend processes them, queries the database, and returns a JSON/XML response → this separation enables independent scaling, maintainability, and allows different teams to own each layer; common patterns include AJAX calls, Fetch API, and Axios on the frontend side.

---

### Q24: What are APIs? Did they exist before web programming?

**A:** An API (Application Programming Interface) is a contract that defines how software components communicate — it exposes methods/endpoints that other code can call without knowing the internal implementation → APIs work by accepting structured requests and returning structured responses; they predate the web and existed as OS-level APIs (Win32 API for Windows, POSIX for Unix), library APIs (C standard library), and database APIs long before HTTP-based REST APIs → today web APIs (REST, GraphQL, gRPC) are dominant, but the core concept of exposing a defined interface for inter-component communication is universal and decades old.

---

### Q25: What are Pointers? (C/C++ context)

**A:** A pointer is a variable that stores the memory address of another variable rather than its value directly → in C/C++, you declare a pointer with `*`, assign it the address of a variable using `&`, and dereference it with `*` to access or modify the value at that address → pointers enable dynamic memory allocation, pass-by-reference semantics, and efficient data structures like linked lists and trees, but they carry risks: dangling pointers (pointing to freed memory), null pointer dereferencing, and memory leaks if `free()`/`delete` is not called; Java removes explicit pointers, replacing them with references and garbage collection.

---

## SQL — Extended

### Q26: Find employee with highest commits using 3 tables (employee, department, commit)

**A:** Given separate `employee`, `department`, and `commit` tables, a subquery approach finds the max commits per department and then joins to retrieve employee details.

```sql
SELECT d.department_id, e.employee_id, c.num_commits
FROM department d
JOIN employee e ON d.department_id = e.department_id
JOIN commit c ON e.employee_id = c.employee_id
WHERE (d.department_id, c.num_commits) IN (
    SELECT e2.department_id, MAX(c2.num_commits)
    FROM commit c2
    JOIN employee e2 ON c2.employee_id = e2.employee_id
    GROUP BY e2.department_id
);
```

**Java Streams equivalent:**
```java
// Group commits by employeeId, keep max per dept
Map<Integer, Commit> topByDept = commits.stream()
    .collect(Collectors.toMap(
        c -> employeeMap.get(c.employeeId).departmentId,
        c -> c,
        (c1, c2) -> c1.numCommits >= c2.numCommits ? c1 : c2
    ));
```

---

## Threading — Extended

### Q27: How is Executor Framework used in batch job processing?

**A:** Batch processing involves running a large set of tasks (e.g., data transforms, report generation) efficiently → the Executor Framework provides a `ThreadPoolExecutor` that manages a pool of worker threads, accepting task submissions via `execute()` or `submit()` so threads are reused rather than created per task → benefits include controlled concurrency (avoiding resource exhaustion), task queuing, graceful shutdown, and `Future`-based result retrieval; a typical pattern creates a fixed-size pool equal to CPU cores × 2 for I/O-bound jobs.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
for (BatchTask task : tasks) {
    executor.submit(task);
}
executor.shutdown();
executor.awaitTermination(1, TimeUnit.HOURS);
```

---

### Q28: How do you decide the minimum number of threads required?

**A:** Thread count depends on task type and hardware → for CPU-bound tasks, the formula is `threads = CPU cores + 1` to keep CPUs busy with minimal context switching; for I/O-bound tasks (DB calls, HTTP), use `threads = CPU cores × (1 + wait_time / compute_time)` since threads spend most time waiting → profile with different pool sizes using load tests and monitor CPU utilization, context switch rate, and throughput; too few threads wastes CPU, too many causes excessive context switching and memory overhead; in practice, start with `Runtime.getRuntime().availableProcessors() * 2` and tune from there.

---

## JavaScript

### Q29: What are Closures in JavaScript?

**A:** A closure is a function that retains access to variables from its outer (enclosing) scope even after the outer function has finished executing → JavaScript implements closures because inner functions capture a reference to the outer function's scope chain, not just the values; this allows the captured variables to persist in memory as long as the inner function exists → closures are used for data encapsulation (private variables), callback patterns, module patterns, and memoization; a caution: closures can cause unintended memory leaks if they hold references to large objects longer than needed.

```javascript
function counter() {
    let count = 0;            // captured by closure
    return () => ++count;     // inner function = closure
}
const inc = counter();
inc(); // 1
inc(); // 2 — count persists across calls
```

---

## Finance Domain

### Q30: What is Financial Risk?

**A:** Financial risk is the probability of losing money or failing to meet financial obligations due to uncertainty in markets, operations, or counterparties → it manifests through multiple types: **market risk** (price/rate changes), **credit risk** (borrower default), **liquidity risk** (inability to convert assets to cash), **operational risk** (system failures, fraud), and **currency risk** (FX rate movements) → firms manage financial risk through diversification, hedging (derivatives), insurance, stress testing, and regulatory compliance (Basel III, Dodd-Frank); in a banking context like Deutsche Bank, risk-weighted assets and VaR (Value at Risk) models are core tools.
