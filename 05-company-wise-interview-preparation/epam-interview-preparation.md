# EPAM Systems — Interview Prep Guide

**Role:** Senior Software Engineer (Java + React)  
**Interview Type:** 5 rounds — Coding, Backend, Frontend, Architecture, Behavioral

---

## Interview Structure

| Round | Focus | Depth |
|-------|-------|-------|
| 1 | Coding + Core Java | Medium–High |
| 2 | Backend (Spring Boot + System Design) | High |
| 3 | Frontend (React + JS) | Medium–High |
| 4 | Architecture / Managerial | High |
| 5 | Behavioral (STAR-based) | Medium |

---

## Round 1: Core Java Focus Areas

### Q1: How does ConcurrentHashMap work internally (Java 8+)?

**A:**
- **No segment locking** (Java 8 removed Segment class)
- Uses **CAS (Compare-And-Swap)** for most write operations
- **Bucket-level `synchronized`** only on hash collision (linked list/tree node)
- Reads are **lock-free** (volatile reads)
- `computeIfAbsent`, `merge` are **atomic per key**

---

### Q2: equals() vs hashCode() contract

**A:**
- If `a.equals(b)` is true → `a.hashCode() == b.hashCode()` must be true
- If hashCodes are equal → `equals` may or may not be true (collision)
- **Must override both together** — breaking this corrupts HashMap/HashSet behavior

---

### Q3: CompletableFuture — when and how?

```java
CompletableFuture.supplyAsync(() -> fetchFromDB())
    .thenApply(data -> transform(data))
    .thenAccept(result -> save(result))
    .exceptionally(ex -> { log.error(ex); return null; });
```

**Key methods:** `supplyAsync`, `thenApply`, `thenCompose` (flatMap), `allOf`, `anyOf`, `exceptionally`.

**Use case:** Non-blocking downstream API calls in microservices.

---

## Round 2: Spring Boot Focus Areas

### Q4: Bean Lifecycle

```
Instantiate → Populate Properties → BeanNameAware → BeanFactoryAware →
ApplicationContextAware → @PostConstruct → InitializingBean.afterPropertiesSet →
custom init-method → Bean Ready → @PreDestroy → DisposableBean.destroy →
custom destroy-method
```

---

### Q5: @Transactional — propagation & isolation

**Must-know flow:**
```
Request → Controller → @Transactional Service → Repository → DB
```

- Default propagation: `REQUIRED`
- Default isolation: DB default (usually `READ_COMMITTED`)
- **Proxy-based:** Self-invocation (`this.method()`) bypasses the proxy → `@Transactional` won't work
- **Fix:** Inject self, use `AopContext`, or restructure to call from another bean

---

### Q6: REST API design — idempotency

| Method | Idempotent | Safe |
|--------|-----------|------|
| GET | Yes | Yes |
| PUT | Yes | No |
| DELETE | Yes | No |
| POST | No | No |
| PATCH | No | No |

**Making POST idempotent:** Use idempotency key header (`Idempotency-Key: uuid`). Server checks if key was already processed.

---

## Round 3: React Focus Areas

### Q7: Virtual DOM and reconciliation

- React maintains a **virtual DOM** (JS object tree)
- On state change → new virtual DOM created → **diffed** against previous
- Only **changed nodes** are updated in real DOM (reconciliation)
- **Keys** help React identify which items changed in lists

---

### Q8: React state management at scale

```
Local state (useState) → Lifted state → Context API → Redux/Zustand
```

- **Context:** Good for low-frequency global data (auth, theme)
- **Redux:** Complex cross-component state with middleware (thunk, saga)
- **React Query/SWR:** Server state management — don't duplicate API data in Redux

---

## Round 4: Architecture Focus Areas

### Q9: Microservices resilience patterns

| Pattern | Purpose |
|---------|---------|
| **Circuit Breaker** | Stop calling failing service after threshold |
| **Retry** | Retry transient failures with exponential backoff |
| **Bulkhead** | Isolate failures to prevent cascading |
| **Timeout** | Don't wait forever for downstream |
| **Fallback** | Provide default response when service fails |

**Implementation:** Resilience4j (Spring Boot), Istio (service mesh).

---

### Q10: Event-Driven Architecture

```
Producer → Message Broker (Kafka/SQS) → Consumer
```

**When to use:** Decoupling services, async processing, audit trails, CQRS.  
**Key concepts:** Event sourcing, eventual consistency, idempotent consumers, DLQ for failures.

---

## Round 5: Behavioral (STAR Format)

### Q11: Tell me about a time you made an impactful technical decision

Use STAR:
- **S:** Team planned a costly DB migration
- **T:** Validate necessity before committing resources
- **A:** Analyzed performance data, built cost-benefit comparison, recommended deferral
- **R:** Saved significant cost and engineering time

---

## Program: find pallendroms from list of strings :

```
import java.util.*;
import java.util.stream.*;

class Main {
    public static void main(String[] args) {
        System.out.println("Start ...");
        List<String> list = List.of("aba", "abcd", "bananab", "AMMA", "121","12","1","122");

        List<String> pallindromList= list.stream().filter(str ->isPallendrome(str)).collect(Collectors.toList());
     
        System.out.println(pallindromList);
    }
    
    static boolean isPallendrome(String str){
        int left = 0;
        int right = str.length()-1;
        while(left<right){
            if(str.charAt(left) != str.charAt(right)){
                return false;
            }
            left++;
            right--;
        }
       return true;
    }
 
}
```

---
Program : max sum of sub array for 'k' th values in java
```
// Online Java Compiler
// Use this editor to write, compile and run your Java code online
import java.lang.*;
import java.util.*;

class Main {
    public static void main(String[] args) {
        int[] arr = {1,3,6,4,8,2};
        int k = 4;
        int result = subArrayMaxSum(arr, k);
        System.out.println("Start small. Ship something.=> "+result);
    }
    // O(nk) O(1)
    public static int subArrayMaxSum(int[] arr, int k) {
        System.out.println("Start small. Ship something.");
        int n = arr.length;
        int maxSum = Integer.MIN_VALUE;
        
        for(int i=0; i< n; i++){
            int tempSum = 0;
            for (int j=i; j < Math.min(i+k, n); j++){
                tempSum += arr[j];
                maxSum = Math.max(maxSum, tempSum);
            }
        }
        
        return maxSum;
    }

    //Sliding window solution // O(n), space o(1)
 public static int findMaxSum(int[] arr, int k) {
        if (arr == null || arr.length < k || k <= 0) {
            return 0; 
        }

        int windowSum = 0;

        // Loop 1: Calculate the sum of the first window
        for (int i = 0; i < k; i++) {
            windowSum = windowSum + arr[i];
        }

        int maxSum = windowSum;
        int start = 0;
        // Loop 2: Slide the window across the array
        for (int end = k; end < arr.length; end++) {
            // Simplified expansion of the sliding window math
            windowSum = windowSum + arr[end] - arr[start];
            start ++;
            
            
        maxSum = Math.max(windowSum, maxSum);
            
        }

        return maxSum;
    }
}
```
---

*Add specific EPAM questions as you encounter them in interviews.*

---
Coding question: *It will take approximately 7.27 years
for ₹100 to grow to ₹200 at 10% annual compound growth.*

```
public class CompoundGrowth {

    public static void main(String[] args) {

        double principal = 100;
        double target = 200;
        double rate = 0.10;

        int years = 0;

        while (principal < target) {
            principal = principal * (1 + rate);
            years++;
        }

        System.out.println("Years required: " + years);
    }
}

```


---
Coding question:
*2. Find longest substring without repeating character*

```
public int lengthOfLongestSubstring(String s) {

    Set<Character> set = new HashSet<>();

    int left = 0;
    int maxLength = 0;

    for (int right = 0; right < s.length(); right++) {

        while (set.contains(s.charAt(right))) {
            set.remove(s.charAt(left));
            left++;
        }

        set.add(s.charAt(right));

        maxLength = Math.max(maxLength, right - left + 1);
    }

    return maxLength;
}
```
