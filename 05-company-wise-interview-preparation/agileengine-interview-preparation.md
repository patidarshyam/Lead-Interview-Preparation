# AgileEngine — Coding Test Questions

**Role:** Java Full Stack Developer  
**Interview Type:** Codility-style online coding test

---

## Q1: VisitCounter — Aggregate user visits across microservices

**Problem:** Given an array of `Map<String, UserStats>` (each map = visits per user for one microservice), aggregate total visits per user.

**Constraints:**
1. Map key (String) must be parseable to Long — skip invalid
2. UserStats may be null — skip
3. `visitCount` is `Optional<Long>` — skip if empty
4. Handle null input, empty maps gracefully

**Solution (Java 8 Streams):**

```java
public Map<Long, Long> count(Map<String, UserStats>... visits) {
    if (visits == null || visits.length == 0) {
        return Collections.emptyMap();
    }

    return Arrays.stream(visits)
        .filter(Objects::nonNull)
        .flatMap(map -> map.entrySet().stream())
        .filter(entry -> isValidKey(entry.getKey())
                      && isValidUserStats(entry.getValue()))
        .collect(Collectors.groupingBy(
            entry -> Long.parseLong(entry.getKey()),
            Collectors.summingLong(entry ->
                entry.getValue().getVisitCount().orElse(0L))
        ));
}

private boolean isValidKey(String key) {
    try { Long.parseLong(key); return true; }
    catch (NumberFormatException e) { return false; }
}

private boolean isValidUserStats(UserStats stats) {
    return stats != null && stats.getVisitCount().isPresent();
}
```

**Java 7 equivalent (no streams):**

```java
public Map<Long, Long> count(Map<String, UserStats>... visits) {
    Map<Long, Long> result = new HashMap<>();
    if (visits == null) return result;

    for (Map<String, UserStats> map : visits) {
        if (map == null) continue;
        for (Map.Entry<String, UserStats> entry : map.entrySet()) {
            if (!isValidKey(entry.getKey())) continue;
            if (!isValidUserStats(entry.getValue())) continue;

            Long userId = Long.parseLong(entry.getKey());
            Long count = entry.getValue().getVisitCount().get();
            result.merge(userId, count, Long::sum);
        }
    }
    return result;
}
```

**Test cases to cover:**
- Normal input with multiple maps
- Null input → empty map
- Null map in array → skip
- Invalid string key → skip
- Null UserStats → skip
- Empty Optional visitCount → skip
- Duplicate user IDs across maps → sum

---

*Add more coding test questions as you encounter them.*
