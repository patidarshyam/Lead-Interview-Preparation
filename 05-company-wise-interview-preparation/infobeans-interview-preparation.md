# Infobeans — Interview Questions & Answers

**Role:** Java Full Stack Developer (Java 8, Angular 9)  
**Interview Type:** Technical + Live Coding (2 rounds)

---

## Round 1: Technical Q&A

### Q1: SOAP vs REST

| Aspect | SOAP | REST |
|--------|------|------|
| Protocol | XML-based protocol | Architectural style |
| Format | XML only | JSON, XML, plain text |
| Contract | WSDL mandatory | Optional (OpenAPI/Swagger) |
| State | Can be stateful | Stateless |
| Performance | Heavier (XML parsing) | Lighter (JSON) |
| Security | WS-Security | HTTPS + OAuth |

---

### Q2: AngularJS vs Angular 2+

| Aspect | AngularJS | Angular 2+ |
|--------|-----------|------------|
| Language | JavaScript | TypeScript |
| Architecture | MVC | Component-based |
| Data Binding | Two-way (scope) | Two-way + one-way |
| Mobile Support | No | Yes (Ionic, NativeScript) |
| Performance | Digest cycle (slower) | Change detection (faster) |
| Dependency Injection | Yes (basic) | Yes (hierarchical) |

---

### Q3: TypeScript vs JavaScript

| Feature | TypeScript | JavaScript |
|---------|-----------|------------|
| Typing | Static + strong | Dynamic + weak |
| Compilation | Compiled to JS | Interpreted |
| Interfaces | Yes | No |
| Generics | Yes | No |
| Tooling | Better IDE support | Basic |

---

### Q4: Angular Observables — types and differences

| Type | Behavior |
|------|----------|
| **Subject** | No initial value, emits to current subscribers only |
| **BehaviorSubject** | Has initial value, new subscribers get last emitted value |
| **ReplaySubject** | Replays N previous values to new subscribers |
| **AsyncSubject** | Emits only the last value, only on completion |

---

### Q5: Observables vs Promises

| Aspect | Observable | Promise |
|--------|-----------|---------|
| Values | Multiple over time | Single value |
| Lazy | Yes (executes on subscribe) | No (executes immediately) |
| Cancellable | Yes (unsubscribe) | No |
| Operators | map, filter, switchMap, etc. | .then(), .catch() |

---

### Q6: Angular Lifecycle Hooks

```
constructor → ngOnChanges → ngOnInit → ngDoCheck →
ngAfterContentInit → ngAfterContentChecked →
ngAfterViewInit → ngAfterViewChecked → ngOnDestroy
```

**Most used:** `ngOnInit` (data fetch), `ngOnDestroy` (cleanup subscriptions).

---

### Q7: Lazy Loading in Angular

```typescript
// app-routing.module.ts
{ path: 'admin', loadChildren: () =>
    import('./admin/admin.module').then(m => m.AdminModule)
}
```

**Benefit:** Loads module only when route is accessed → faster initial load.

---

### Q8: Angular ViewChild

```typescript
@ViewChild('myInput') inputRef: ElementRef;

ngAfterViewInit() {
    this.inputRef.nativeElement.focus();
}
```

**Use case:** Access DOM elements or child component instances from parent.

---

### Q9: LazyInitializationException in Spring/Hibernate

**Cause:** Accessing lazy-loaded entity outside the Hibernate session.

**Solutions:**
1. `@Transactional` on the service method (keeps session open)
2. `FetchType.EAGER` (not recommended for large collections)
3. `JOIN FETCH` in HQL query
4. Open Session in View pattern (anti-pattern but works)

---

### Q10: Transaction Propagation in Spring

| Type | Behavior |
|------|----------|
| `REQUIRED` (default) | Join existing or create new |
| `REQUIRES_NEW` | Always create new, suspend existing |
| `NESTED` | Create savepoint within existing |
| `SUPPORTS` | Use existing if available, else non-transactional |
| `MANDATORY` | Must have existing, else exception |
| `NOT_SUPPORTED` | Suspend existing, run non-transactional |
| `NEVER` | Exception if transaction exists |

---

### Q11: Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| READ_UNCOMMITTED | Yes | Yes | Yes |
| READ_COMMITTED | No | Yes | Yes |
| REPEATABLE_READ | No | No | Yes |
| SERIALIZABLE | No | No | No |

---

### Q12: Database Views vs Materialized Views vs Triggers vs Procedures

| Object | Description | Refreshed |
|--------|-------------|-----------|
| **View** | Virtual table (SQL query) | Every query |
| **Materialized View** | Physically stored result | On demand / scheduled |
| **Trigger** | Auto-executes on DML event | Event-driven |
| **Stored Procedure** | Precompiled SQL block | On call |
| **Function** | Returns value, usable in SQL | On call |

---

## Round 2: Live Coding

### Q13: Count deck of cards from infinite stream of strings

```java
public static Map<String, Long> countCards(Stream<String> cards) {
    Set<String> validCards = Set.of(
        "A","2","3","4","5","6","7","8","9","10","J","Q","K");
    Set<String> validSuits = Set.of(
        "Hearts","Diamonds","Clubs","Spades");

    return cards
        .filter(card -> {
            String[] parts = card.split(" of ");
            return parts.length == 2
                && validCards.contains(parts[0])
                && validSuits.contains(parts[1]);
        })
        .collect(Collectors.groupingBy(
            Function.identity(), Collectors.counting()));
}
```

---

### Q14: Passing data between Angular components

| Method | Direction | Use Case |
|--------|-----------|----------|
| `@Input` | Parent → Child | Pass data down |
| `@Output` + `EventEmitter` | Child → Parent | Emit events up |
| `@ViewChild` | Parent → Child | Direct reference |
| Service + Subject | Any ↔ Any | Shared state |
| Route params | Route → Component | URL-driven |

---

### Q15: SQL Joins — quick reference

| Join | Returns |
|------|---------|
| INNER JOIN | Only matching rows in both tables |
| LEFT JOIN | All from left + matching from right |
| RIGHT JOIN | All from right + matching from left |
| FULL OUTER JOIN | All from both, NULL where no match |
| CROSS JOIN | Cartesian product |
| SELF JOIN | Table joined with itself |

**Join vs Subquery:** Joins are generally faster for large datasets. Subqueries are simpler for existence checks (`EXISTS`).
