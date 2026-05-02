# Testing with Mockito & JUnit

Unit testing patterns using Mockito and JUnit 4/5 for Spring Boot applications.

---

## Q1: How do you test a void method using Mockito?

**Answer:**

**Mockito's `verify()`** is used to assert that a void method was called with expected parameters. Unlike methods that return a value, void methods can't be asserted on a return — instead you verify the interaction occurred. `doNothing()` or `doThrow()` can also stub void methods.

```java
import static org.mockito.Mockito.*;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class NotificationServiceTest {

    @Mock
    private EmailSender emailSender;

    @Test
    void testSendNotification() {
        // void method — nothing to stub by default (does nothing on mock)
        notificationService.sendAlert("user@example.com", "Alert message");

        // Verify the void method was called with exact arguments
        verify(emailSender).send("user@example.com", "Alert message");

        // Verify it was called exactly once
        verify(emailSender, times(1)).send(anyString(), anyString());

        // Verify it was never called with wrong args
        verify(emailSender, never()).send("wrong@example.com", anyString());
    }

    @Test
    void testVoidMethodThrowsException() {
        // Stub void method to throw
        doThrow(new RuntimeException("SMTP failure"))
            .when(emailSender).send(anyString(), anyString());

        assertThrows(RuntimeException.class,
            () -> emailSender.send("user@example.com", "msg"));
    }
}
```

**Benefits / Trade-offs:** `verify()` checks behavior not state — appropriate for side-effect-only methods. Use `doNothing()` when you need to explicitly suppress a stubbed void call. Overusing verify can make tests brittle; verify only the interactions your test cares about.

---

## Q2: What annotations are needed to run Mockito tests in JUnit 5 vs JUnit 4?

**Answer:**

**JUnit 5 uses extensions; JUnit 4 uses runners.** Mockito integrates with both but through different mechanisms. Getting this wrong is a common cause of mocks being `null` at test time.

```java
// JUnit 5 — use @ExtendWith(MockitoExtension.class)
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void testFindUser() {
        when(userRepository.findById(1L)).thenReturn(Optional.of(new User("Alice")));
        User user = userService.findById(1L);
        assertEquals("Alice", user.getName());
    }
}

// JUnit 4 — use @RunWith(MockitoJUnitRunner.class)
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.MockitoJUnitRunner;

@RunWith(MockitoJUnitRunner.class)
public class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    public void testFindUser() { ... }
}
```

**Benefits / Trade-offs:** `@ExtendWith(MockitoExtension.class)` is the modern JUnit 5 approach — it initialises `@Mock` and `@InjectMocks` automatically, detects unused stubs as errors (strictness), and integrates with Spring's `@SpringExtension`. Alternative: call `MockitoAnnotations.openMocks(this)` in `@BeforeEach` to avoid the annotation entirely.

---

## Q3: How do you write a Spring Boot integration test vs a unit test with mocks?

**Answer:**

**Spring Boot offers two test layers**: `@SpringBootTest` loads the full application context (integration test) and is slow; `@WebMvcTest` / `@DataJpaTest` slice tests load partial context; pure Mockito unit tests load nothing and are fastest. Choose the narrowest scope for the behaviour you're testing.

```java
// Full integration test — loads entire Spring context
@SpringBootTest
@ExtendWith(SpringExtension.class)
class OrderServiceIntegrationTest {
    @Autowired
    private OrderService orderService; // real bean from context

    @Test
    void testCreateOrder() { ... }
}

// Controller slice test — only web layer
@WebMvcTest(OrderController.class)
class OrderControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean  // replaces real bean in Spring context with mock
    private OrderService orderService;

    @Test
    void testGetOrder() throws Exception {
        when(orderService.findById(1L)).thenReturn(new Order(1L, "item"));
        mockMvc.perform(get("/orders/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.item").value("item"));
    }
}

// Pure unit test — no Spring context at all
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock private OrderRepository orderRepository;
    @InjectMocks private OrderService orderService;
    // fastest — milliseconds per test
}
```

**Benefits / Trade-offs:** Unit tests (pure Mockito) run in milliseconds and isolate logic perfectly. `@WebMvcTest` is ideal for controller tests including request mapping, validation, and serialization. `@SpringBootTest` is for end-to-end flows but is slow and should be limited. Use `@MockBean` (not `@Mock`) when you need to replace a Spring-managed dependency.

---

## Q4: How do you mock an `EntityManager.createQuery()` that returns a `Query` object?

**Answer:**

**JPA's `EntityManager` and `Query` are interfaces**, making them straightforward to mock with Mockito. Chain `when()` stubs so `createQuery()` returns a mocked `Query`, and `getResultList()` returns your test data. This pattern is essential for testing repository classes that use raw JPQL.

```java
import javax.persistence.EntityManager;
import javax.persistence.Query;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class ReportRepositoryTest {

    @Mock
    private EntityManager entityManager;

    @InjectMocks
    private ReportRepository reportRepository;

    @Test
    void testFindReports() {
        // Create mock Query
        Query mockQuery = mock(Query.class);

        // Stub return data
        List<Report> fakeResults = List.of(new Report(1L), new Report(2L));
        when(mockQuery.getResultList()).thenReturn(fakeResults);

        // Stub EntityManager to return mock Query
        when(entityManager.createQuery(anyString())).thenReturn(mockQuery);

        // Execute
        List<Report> results = reportRepository.findAll();

        // Verify
        assertEquals(2, results.size());
        verify(entityManager).createQuery(anyString());
        verify(mockQuery).getResultList();
    }
}
```

**Benefits / Trade-offs:** Mocking `EntityManager` decouples repository tests from the database entirely. If your query uses `setParameter()`, add `when(mockQuery.setParameter(anyString(), any())).thenReturn(mockQuery)` to keep the chain fluent. For Spring Data JPA repositories, prefer `@DataJpaTest` with an in-memory H2 database instead — it tests real query behaviour without a full context.

---

## Q5: What is `@InjectMocks` and when does it fail silently?

**Answer:**

**`@InjectMocks`** creates an instance of the class under test and injects all `@Mock` / `@Spy` fields into it via constructor injection, setter injection, or field injection — in that order. It silently skips injection if no matching field exists, which can leave dependencies `null` without any error.

```java
@ExtendWith(MockitoExtension.class)
class PaymentServiceTest {

    @Mock
    private PaymentGateway paymentGateway;

    @Mock
    private AuditLogger auditLogger;

    @InjectMocks
    private PaymentService paymentService;
    // Mockito creates new PaymentService() and injects both mocks

    @Test
    void testProcessPayment() {
        when(paymentGateway.charge(any())).thenReturn(true);
        boolean result = paymentService.process(new Payment(100.0));
        assertTrue(result);
        verify(auditLogger).log(any());
    }
}
```

**Common failure modes:**
```java
// PROBLEM: PaymentService has a required constructor arg not mocked
// @InjectMocks will fail silently — paymentGateway stays null

// FIX: Prefer explicit construction when constructor is complex
@BeforeEach
void setUp() {
    paymentService = new PaymentService(paymentGateway, auditLogger);
}
```

**Benefits / Trade-offs:** `@InjectMocks` reduces boilerplate for simple classes. However, it uses field injection by default which hides missing dependencies — a common source of `NullPointerException` in tests. For production-grade tests, prefer explicit constructor injection in both your class and test setup.
