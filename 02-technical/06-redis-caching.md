# Redis & Caching

---

## Q1: Explain Redis Cluster Mode. When do you enable it vs. single-node?

**Answer:**

**Cluster Mode:**
- Data is **sharded** across multiple primary nodes using hash slots (16384 slots total)
- Each shard has a primary + replicas
- Client library (e.g., ioredis) handles slot-based routing automatically
- **Scales horizontally** : add shards to increase memory and throughput

**Single Node (non-cluster):**
- One primary, optional replicas for read scaling
- Simpler to operate
- Limited by single node's memory and CPU

**When to use Cluster Mode:**
- Dataset > single node memory
- Need horizontal write scaling
- High throughput requiring parallel operations across shards

**When single node is fine:**
- Dataset fits in one node's memory
- Read-heavy workload (replicas handle reads)
- Simpler operational requirements

**My experience:** For a context API cache handling 1800+ concurrent requests, I used cluster mode with 1 shard, 1 replica, multi-AZ. Even with 1 shard, cluster mode gives you the **flexibility to scale horizontally later** without client-side changes.

---

## Q2: How do you handle cache stampede (thundering herd)?

**Answer:**

Cache stampede happens when many concurrent requests hit a cache miss simultaneously, all go to the backend, overwhelming it.

**Mitigation strategies:**

1. **Distributed lock (mutex)** : On cache miss, acquire a lock (Redis `SET NX EX`). Only the lock holder fetches from backend and populates cache. Others wait or use stale data.
2. **Early expiration (probabilistic)** : Refresh the cache before TTL expires using a background process. Cache is never cold for popular keys.
3. **Request coalescing** : Deduplicate in-flight requests. If key X is already being fetched, queue subsequent requests for key X behind the first one.
4. **Stale-while-revalidate** : Serve slightly stale data while refreshing in the background. Acceptable when data freshness tolerance is > 0.

**What I've used:** For burst traffic scenarios, the combination of short TTL + timestamp-based keys naturally mitigates stampede. Since the key includes the entity's modified timestamp, a cache miss only happens once per version of the data.

---

## Q3: What are the key Redis configuration parameters you tune for production?

**Answer:**

| Parameter | Purpose | Typical Value |
|-----------|---------|---------------|
| `commandTimeout` | Max wait for a Redis command | 5000ms |
| `connectTimeout` | Max wait for initial connection | 15000ms |
| `maxRetriesPerRequest` | Retries before failing | 3 |
| `retryStrategy` | Backoff between retries | `min(times * 200, 2000)ms` |
| `scaleReads` | Read distribution | `all` (primary + replicas) |
| `enableReadyCheck` | Wait for Redis READY before commands | `true` |
| `lazyConnect` | Connect on first command vs. immediately | `false` for production |

**Monitoring metrics:**
- **Cache hit ratio** : Target > 70%. Below that, review key design.
- **Evictions** : Non-zero means memory pressure. Scale up or review TTL.
- **Connection count** : Ensure not hitting `maxclients` limit.
- **Latency** : p99 should be < 5ms for ElastiCache.
- **Memory usage** : Stay below 80% of max to avoid eviction storms.

---

## Q4: How do you design cache keys for a multi-tenant API?

**Answer:**

**Format:** `service-name:{tenantId}:{entityId}:{versionOrTimestamp}`

**Example:** `context-api:org123:fieldOp456:1714500000`

**Principles:**
- **Tenant isolation** : Tenant ID in key prevents cross-tenant data leakage
- **Version/timestamp suffix** : Natural invalidation. When data updates, the modified timestamp changes --> new key --> old key expires via TTL
- **Predictable structure** : Easy to debug (`redis-cli KEYS "context-api:org123:*"`)
- **Avoid dynamic sorting in keys** : For multi-entity keys, sort IDs deterministically (e.g., ascending) so the same set always produces the same key

**Anti-patterns to avoid:**
- Including request-specific data (like pagination offset) in keys unless necessary
- Using `KEYS *` in production (use `SCAN` instead)
- Keys that are too long (> 1KB) : wastes memory on key storage

---

## Q5: Redis Cache-Aside Pattern — Implementation, TTL, and Invalidation

**Answer:**

The **cache-aside (lazy loading)** pattern is the most common Redis caching strategy: the application checks Redis first; on a miss, it fetches from DB, writes to Redis, and returns the result.

**Implementation — Service Layer (never Controller or DAO):**

```java
@Service
public class OrderService {
    private final StringRedisTemplate redis;
    private final OrderRepository repo;
    private final ObjectMapper mapper;
    private static final Duration TTL = Duration.ofMinutes(5);

    @Transactional(readOnly = true)
    public List<Order> getOrdersByCustomer(String customerId) {
        String cacheKey = "orders:customer:" + customerId;

        // 1. Check cache
        String cached = redis.opsForValue().get(cacheKey);
        if (cached != null) {
            return mapper.readValue(cached, new TypeReference<>() {}); // cache hit
        }

        // 2. Cache miss — fetch from DB
        List<Order> orders = repo.findByCustomerId(customerId);

        // 3. Populate cache with TTL
        redis.opsForValue().set(cacheKey, mapper.writeValueAsString(orders), TTL);
        return orders;
    }
}
```

**Cache Invalidation (critical — maintain consistency):**

```java
@Transactional
public Order createOrder(String customerId, OrderRequest req) {
    Order saved = repo.save(req.toOrder(customerId));

    // Invalidate cache — stale data must not be served
    redis.delete("orders:customer:" + customerId);
    return saved;
}

@Transactional
public Order updateOrder(Long orderId, OrderRequest req) {
    Order updated = repo.save(req.toOrder(orderId));
    redis.delete("orders:customer:" + updated.getCustomerId()); // remove stale
    return updated;
}
```

**Spring Cache annotation alternative (simpler for reads):**

```java
@Service
public class ProductService {
    @Cacheable(value = "products", key = "#productId", unless = "#result == null")
    public Product getProduct(Long productId) { return repo.findById(productId).orElse(null); }

    @CacheEvict(value = "products", key = "#productId")
    public void updateProduct(Long productId, ProductRequest req) { repo.save(...); }

    @CachePut(value = "products", key = "#result.id")
    public Product createProduct(ProductRequest req) { return repo.save(...); }
}
```

**Key design decisions:**

| Decision | Recommendation |
|----------|---------------|
| Key naming | `entity:scope:id` (e.g., `orders:customer:123`) |
| TTL | Always set TTL — never cache indefinitely |
| Granularity | Cache paginated results, not entire tables |
| Invalidation | Delete on write (vs update) — simpler, avoids update race conditions |
| Stale data | Acceptable for non-critical reads; use write-through for financial data |

**Trade-offs:**
- Cache-aside is resilient (DB works even if Redis is down)
- Write-through is consistent but adds latency on writes
- Never cache the entire table result — cache paginated keys: `orders:page:1`, `orders:page:2`


---

## Q6: Redis Distributed Locking — SETNX for Cache Stampede Prevention

**Answer:**

Distributed locking with Redis ensures only one process/thread executes a critical section (e.g., DB fetch on cache miss) across all application instances.

**Core Redis command:**
```bash
SET lock:order:123 uniqueValue NX EX 10
# NX = only set if key does NOT exist (acquire lock)
# EX 10 = auto-expire in 10 seconds (prevent deadlocks if holder crashes)
```

**Cache stampede prevention flow:**
```
100 concurrent cache misses → only 1 acquires lock → fetches DB → populates cache
                            → other 99 retry or serve stale data
```

**Java implementation (Spring Redis):**
```java
@Service
public class OrderService {
    private final StringRedisTemplate redis;
    private final OrderRepository repo;
    private final ObjectMapper mapper;

    public List<Order> getOrdersPage(int page) {
        String cacheKey = "orders:page:" + page;
        String lockKey  = "lock:orders:page:" + page;
        String lockVal  = UUID.randomUUID().toString(); // unique per caller

        // 1. Try to acquire lock (only on cache miss)
        String cached = redis.opsForValue().get(cacheKey);
        if (cached != null) return deserialize(cached);

        Boolean acquired = redis.opsForValue().setIfAbsent(lockKey, lockVal, Duration.ofSeconds(10));

        if (Boolean.TRUE.equals(acquired)) {
            try {
                // 2. Re-check cache (another thread may have populated it)
                cached = redis.opsForValue().get(cacheKey);
                if (cached != null) return deserialize(cached);

                // 3. Fetch from DB, populate cache
                List<Order> orders = repo.findAll(PageRequest.of(page, 50)).getContent();
                redis.opsForValue().set(cacheKey, serialize(orders), Duration.ofMinutes(5));
                return orders;
            } finally {
                // 4. Safe unlock — only delete if we own the lock
                if (lockVal.equals(redis.opsForValue().get(lockKey))) {
                    redis.delete(lockKey);
                }
            }
        } else {
            // 5. Another process holds the lock — return stale or wait briefly
            Thread.sleep(50);
            String staleCached = redis.opsForValue().get(cacheKey);
            return staleCached != null ? deserialize(staleCached) : repo.findAll(PageRequest.of(page, 50)).getContent();
        }
    }
}
```

**Why safe unlock matters:**
```
Without value check:
Process A acquires lock (EX=10s)
Process A takes 11s → lock auto-expired
Process B acquired new lock
Process A wakes up → DEL lock (deletes B's lock!) ❌ race condition

With value check:
Process A DEL only if redis.get(key) == its own UUID ✅
```

**TTL Jitter — prevents synchronized cache stampede:**
```java
// Bad: all keys expire at same time → stampede
redis.opsForValue().set(key, data, Duration.ofMinutes(5));

// Good: spread expiry with jitter
int jitterSeconds = ThreadLocalRandom.current().nextInt(0, 60); // 0-60s random
redis.opsForValue().set(key, data, Duration.ofMinutes(5).plusSeconds(jitterSeconds));
```

**Production-grade alternative (Redisson):**
```java
RLock lock = redisson.getLock("lock:orders:page:" + page);
try {
    if (lock.tryLock(100, 10000, TimeUnit.MILLISECONDS)) { // wait 100ms, expire 10s
        // critical section
    }
} finally {
    if (lock.isHeldByCurrentThread()) lock.unlock();
}
```

**When to use which approach:**

| Technique | Complexity | Consistency | Use When |
|-----------|------------|-------------|----------|
| TTL + jitter | Low | Eventual | Most cache scenarios |
| Double delete (async) | Medium | Near-strong | High concurrency writes |
| Distributed lock (SETNX) | High | Strong | Hot keys with expensive recompute |
| Redisson / RedLock | High | Strong | Production multi-node Redis |
| Event-driven invalidation | High | Strong (eventual) | Microservices, multi-service caches |

**Interview one-liner:** "Use Redis SETNX with expiry for distributed locking; always use unique lock values for safe unlock to avoid race conditions; prefer Redisson in production for reliability."


---

## Q7: Cache Stampede Prevention and Layered Caching (L1 + L2)

**Answer:**

A **cache stampede** (thundering herd) occurs when a popular cache key expires and many concurrent requests simultaneously miss the cache, all hitting the database at once to recompute the same value.

**Why it happens:**
```
Synchronized expiry + high traffic + slow recomputation = DB spike → potential outage
```

**Prevention strategies:**

*1. Distributed locking — only one thread recomputes:*
```java
@Service
public class OrderCacheService {

    private final RedisTemplate<String, String> redisTemplate;
    private final RedissonClient redisson;

    public OrderPage getOrderPage(int page) {
        String key = "orders:page:" + page;
        String cached = redisTemplate.opsForValue().get(key);
        if (cached != null) return deserialize(cached);

        // Only one thread recomputes — others wait
        RLock lock = redisson.getLock("lock:orders:page:" + page);
        try {
            if (lock.tryLock(2, 5, TimeUnit.SECONDS)) {
                // Double-check after acquiring lock
                cached = redisTemplate.opsForValue().get(key);
                if (cached != null) return deserialize(cached);

                OrderPage data = db.findOrderPage(page);
                redisTemplate.opsForValue().set(key, serialize(data), 60, TimeUnit.SECONDS);
                return data;
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock.isHeldByCurrentThread()) lock.unlock();
        }
        return db.findOrderPage(page); // fallback
    }
}
```

*2. Jittered TTL — prevent synchronized expiry:*
```java
// Instead of fixed TTL, add random jitter so keys don't all expire at once
Random random = new Random();
int ttl = 60 + random.nextInt(30); // 60-90 seconds → staggered expiry
redisTemplate.opsForValue().set(key, value, ttl, TimeUnit.SECONDS);
```

*3. Stale-while-revalidate — serve stale data while refreshing:*
```java
// Return slightly stale data while async background refresh runs
public OrderPage getWithStaleWhileRevalidate(int page) {
    CacheEntry entry = getEntry("orders:page:" + page);
    if (entry != null && entry.isExpired() && !entry.isRefreshing()) {
        entry.markRefreshing();
        CompletableFuture.runAsync(() -> refreshCache(page)); // async refresh
    }
    return entry != null ? entry.getData() : db.findOrderPage(page);
}
```

*4. Cache warming / pre-loading:*
```java
@EventListener(ApplicationReadyEvent.class)
public void warmCache() {
    // Pre-populate hot keys before first request hits
    for (int page = 1; page <= 5; page++) {
        OrderPage data = db.findOrderPage(page);
        redisTemplate.opsForValue().set("orders:page:" + page, serialize(data), 60, SECONDS);
    }
}
```

**Layered Caching (L1 + L2):**

Best practice: local in-memory cache (L1) in front of Redis (L2):
```java
@Configuration
public class CacheConfig {
    @Bean
    public Cache<String, OrderPage> localCache() {
        return Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterWrite(10, TimeUnit.SECONDS) // short TTL
            .build();
    }
}

@Service
public class LayeredCacheService {
    private final Cache<String, OrderPage> l1; // Caffeine (in-process)
    private final RedisTemplate<String, String> l2; // Redis (distributed)

    public OrderPage getOrderPage(int page) {
        String key = "orders:page:" + page;

        // L1 hit (microseconds)
        OrderPage cached = l1.getIfPresent(key);
        if (cached != null) return cached;

        // L2 hit (milliseconds)
        String l2Val = l2.opsForValue().get(key);
        if (l2Val != null) {
            OrderPage page2 = deserialize(l2Val);
            l1.put(key, page2); // populate L1
            return page2;
        }

        // DB miss (tens of milliseconds)
        OrderPage data = db.findOrderPage(page);
        l2.opsForValue().set(key, serialize(data), 60, SECONDS);
        l1.put(key, data);
        return data;
    }
}
```

**Caching options by layer:**

| Layer | Tool | Latency | Shared? | Use When |
|-------|------|---------|---------|----------|
| L1 (in-process) | Caffeine | ~1ÃŽÂ¼s | ❌ Per instance | Hottest, smallest data |
| L2 (distributed) | Redis/ElastiCache | ~1ms | ✅ All instances | Shared across microservices |
| DB query cache | Hibernate L2 | ~10ms | Per app | ORM entity caching |
| Edge cache | CloudFront | <50ms | Global | Static/API GET responses |
| DynamoDB | DAX | ~microseconds | ✅ | DynamoDB-heavy apps |

**Trade-offs:**

| Strategy | Consistency | Complexity | Best For |
|----------|-------------|------------|----------|
| TTL jitter | Eventual | Low | Prevent synchronized expiry |
| Distributed lock | Strong | Medium | Single-writer recompute |
| Stale-while-revalidate | Eventual | High | Maximize availability |
| Cache warming | Strong | Medium | Predictable hot keys |

**Interview one-liner:** "Prevent cache stampede by combining distributed locking (only one recomputes), TTL jitter (staggered expiry), and stale-while-revalidate (serve stale during refresh); use L1 Caffeine + L2 Redis for ultra-low latency layered caching."


---

