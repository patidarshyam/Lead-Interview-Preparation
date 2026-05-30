Equal Experts Shopping Cart - Interview Follow-up Questions & Expected Answers

1. Walk me through your solution

Expected Answer

The solution is a small Java 17 library that implements shopping cart functionality.

The core components are:

- ShoppingCart
- Product
- CartItem
- CartSummary
- PriceClient
- HttpPriceClient

ShoppingCart contains the business logic for:

- adding products
- maintaining cart state
- aggregating quantities
- calculating subtotal, tax and total

Price retrieval is delegated to PriceClient.

HttpPriceClient is responsible for calling the external Price API and converting the response into Product objects.

I intentionally kept the design small and focused because the exercise explicitly discourages unnecessary architectural complexity.

Principles:

- KISS
- SRP
- YAGNI

---

2. Why did you create PriceClient?

Expected Answer

PriceClient represents an external dependency boundary.

ShoppingCart should not depend directly on an HTTP implementation.

Instead it depends on an abstraction.

This follows Dependency Inversion Principle (DIP).

Benefits:

- reduced coupling
- easier testing
- future flexibility

Example:

Today:

HttpPriceClient

Potential future implementations:

- CachedPriceClient
- FilePriceClient
- DatabasePriceClient

I did not implement those because of YAGNI.

---

3. Why didn't you directly inject HttpPriceClient?

Expected Answer

That would also work.

Mockito can mock concrete classes.

The reason I introduced PriceClient was not primarily for testing.

It was to create a clean boundary between business logic and infrastructure concerns.

ShoppingCart should know about price retrieval capability, not HTTP details.

Principles:

- DIP
- Low Coupling

---

4. Why didn't you create ShoppingCartService and ShoppingCartServiceImpl?

Expected Answer

There is only one implementation.

Creating:

- ShoppingCartService
- ShoppingCartServiceImpl

would add abstraction without delivering value.

This would violate YAGNI.

The exercise is intentionally small.

I preferred a simpler design.

Principles:

- KISS
- YAGNI

---

5. Why did you use Map instead of List?

Expected Answer

The requirement states that adding the same product multiple times should result in quantity aggregation.

Map provides:

- O(1) lookup
- O(1) update

Using List would require searching for existing items each time.

Map naturally models unique products inside the cart.

---

6. Why BigDecimal?

Expected Answer

Money calculations should never use float or double.

Floating point types introduce precision issues.

BigDecimal provides predictable decimal arithmetic and proper rounding behavior.

Example:

0.1 + 0.2 != 0.3 using double

BigDecimal avoids this problem.

---

7. Why HALF_UP rounding?

Expected Answer

The assignment requires values rounded to two decimal places.

HALF_UP is the most common financial rounding strategy.

Example:

1.875

becomes

1.88

---

8. Why not store prices as double?

Expected Answer

double is a binary floating point representation.

Many decimal values cannot be represented exactly.

This leads to rounding errors.

Currency calculations should use BigDecimal.

---

9. Why did you not implement caching?

Expected Answer

Caching was not a requirement.

The exercise focuses on correctness and simplicity.

Adding caching introduces:

- cache invalidation concerns
- additional complexity
- more tests

I chose not to add functionality that was not required.

Principle:

YAGNI

---

10. Why did you not add a database?

Expected Answer

Persistence is not required.

The assignment explicitly discourages unnecessary architectural layers.

An in-memory implementation satisfies all stated requirements.

Principles:

- KISS
- YAGNI

---

11. Why no Spring Boot?

Expected Answer

The assignment explicitly says not to submit a web API or application.

Spring Boot would introduce framework complexity without adding value.

The focus of the exercise is business logic and design rather than framework usage.

Principles:

- KISS
- YAGNI

---

12. Which SOLID principles are demonstrated?

Expected Answer

SRP

- ShoppingCart handles cart logic
- HttpPriceClient handles price retrieval

OCP

- PriceClient allows new implementations without changing ShoppingCart

LSP

- Any PriceClient implementation can replace HttpPriceClient

ISP

- PriceClient is intentionally small

DIP

- ShoppingCart depends on PriceClient abstraction

---

13. Why is ShoppingCart in service package?

Expected Answer

ShoppingCart contains business behavior.

It:

- adds products
- aggregates quantities
- calculates totals
- coordinates price retrieval

Product, CartItem and CartSummary are data models.

ShoppingCart represents the business service of the solution.

---

14. Why is CartSummary separate from ShoppingCart?

Expected Answer

CartSummary represents a calculation result.

It separates:

Cart State

from

Calculation Output

This improves readability and keeps responsibilities focused.

Principle:

SRP

---

15. How did you make the code testable?

Expected Answer

Business logic is isolated from external dependencies.

ShoppingCart depends on PriceClient.

Tests mock PriceClient and focus only on cart behavior.

HttpPriceClient is tested separately using MockWebServer.

This prevents unit tests from making real HTTP calls.

Principles:

- DIP
- Separation of Concerns

---

16. Explain your testing strategy

Expected Answer

I separated tests into two categories.

Business Logic Tests

- ShoppingCartTest

Infrastructure Tests

- HttpPriceClientTest

ShoppingCart tests use Mockito.

HttpPriceClient tests use MockWebServer.

This ensures:

- fast unit tests
- isolated business logic
- independent HTTP verification

---

17. How would you implement TDD for this solution?

Expected Answer

I would follow Red-Green-Refactor.

Example sequence:

1. addItem_whenValidProduct_thenProductAddedToCart
2. addItem_whenSameProductAddedTwice_thenQuantityAggregated
3. getSummary_whenCartContainsItems_thenReturnsCorrectSubtotal
4. getSummary_whenCartContainsItems_thenReturnsCorrectTax
5. getSummary_whenCartContainsItems_thenReturnsCorrectTotal

After cart functionality is complete:

6. HttpPriceClient tests

The design supports TDD because dependencies are injected and can be mocked.

---

18. What would you change if the product catalog became large?

Expected Answer

Current complexity is:

Add Product

O(1)

Summary Calculation

O(n)

where n is the number of distinct products.

For a very large catalog I would consider:

- caching prices
- pre-calculated totals
- distributed storage

I did not implement these because current requirements do not justify them.

Principle:

YAGNI

---

19. What if tax rates become configurable?

Expected Answer

Currently tax calculation is simple and fixed.

If multiple tax strategies were required I would introduce:

TaxCalculator

and potentially:

StandardTaxCalculator

At the current scale this would be unnecessary abstraction.

I intentionally kept the tax logic inside ShoppingCart.

Principles:

- KISS
- YAGNI

---

20. Which principle influenced your design the most?

Expected Answer

KISS and YAGNI.

The challenge is intentionally small.

My goal was to create the simplest design that satisfies the requirements while remaining testable and maintainable.

I avoided adding abstractions unless they provided immediate value.

The only abstraction I introduced was PriceClient because it forms a natural boundary around an external dependency.

---

### Follow up questions:

Q2: Why did you create PriceClient?
Follow-up
If there is only one implementation, why create an interface?
Expected Answer:
Normally I avoid interface-for-everything. Here the interface represents an external dependency boundary. It reduces coupling between business logic and infrastructure and makes the dependency explicit.
Follow-up
Could you have used HttpPriceClient directly?
Expected Answer:
Yes. The solution would still work. I considered the interface justified because it separates business logic from infrastructure concerns while adding minimal complexity.
Follow-up
Is this violating YAGNI?
Expected Answer:
It would if the interface existed only for hypothetical implementations. Here it also serves as an architectural boundary around an external dependency, which provides immediate value.

---

Q4: Why didn't you create ShoppingCartService?
Follow-up
Why is ShoppingCart not an interface?
Expected Answer:
There is only one implementation and no requirement suggesting multiple implementations. An interface would add complexity without delivering value.
Follow-up
What would make you introduce an interface?
Expected Answer:
Multiple implementations, separate deployment concerns, plugin architecture, or a clear need for extension.

---

Q5: Why Map instead of List?
Follow-up
What is the complexity difference?
Expected Answer:
Plain text
Map lookup = O(1)

List lookup = O(n)
Since duplicate products must aggregate quantities, Map naturally supports the requirement.
Follow-up
What if product ordering became important?
Expected Answer:
I would consider LinkedHashMap which preserves insertion order while maintaining O(1) lookup.

---


Q11: Why no Spring Boot?
Follow-up
Could Spring Boot have improved the solution?
Expected Answer:
Not for this exercise. The assignment explicitly discourages applications. Spring would mainly add framework complexity rather than solving a requirement.
Follow-up
What if this became a real production service?
Expected Answer:
Then I would likely expose the cart functionality through a REST API using Spring Boot, but I would still keep the domain logic separated from the framework layer.

---

Q12: Which SOLID principles are demonstrated?
Follow-up
Show me where DIP is used.
Expected Answer:
Java
ShoppingCart(PriceClient priceClient)
ShoppingCart depends on the abstraction PriceClient rather than HttpPriceClient.
Follow-up
Where is OCP demonstrated?
Expected Answer:
New implementations of PriceClient can be introduced without modifying ShoppingCart.
Follow-up
Which SOLID principle is weakest in this design?
Expected Answer:
OCP. The requirements are small, so there are limited extension points. I intentionally avoided adding abstractions solely to demonstrate OCP because that would violate YAGNI.
This is a strong senior-level answer.
Q15: How did you make the code testable?
Follow-up
Why constructor injection?
Expected Answer:
Constructor injection makes dependencies explicit, supports immutability, simplifies testing and aligns with Dependency Inversion.
Follow-up
Why not instantiate HttpPriceClient inside ShoppingCart?
Expected Answer:
Java
this.priceClient = new HttpPriceClient();
would tightly couple business logic to infrastructure and make testing harder.

---

Q17: How would you apply TDD?
Follow-up
Did you actually follow TDD?
Expected Answer:
Be honest.
If yes:
I implemented functionality incrementally using red-green-refactor.
If not:
I designed the code to be TDD-friendly and structured the tests around behavior-first scenarios.
Never falsely claim TDD.

---

Q19: Why not TaxCalculator?
Follow-up
Would introducing TaxCalculator be better design?
Expected Answer:
Only if tax calculation became more complex or configurable. For a single fixed calculation, introducing an additional abstraction would increase complexity without adding value.
Follow-up
At what point would you introduce TaxCalculator?
Expected Answer:
Examples:
country-specific taxes
configurable tax rates
promotional tax rules
tax exemptions
At that point the abstraction becomes justified.

---

Equal Experts Shopping Cart - Deep Dive Interview Questions (Level 2 & Level 3)

These questions typically come after the initial design discussion. The goal is to evaluate design reasoning, trade-off analysis, and engineering maturity.

---

PriceClient Design

Q1

Why is PriceClient an interface and not an abstract class?

Expected Answer

PriceClient defines behavior, not shared implementation.

Currently there is no common code between implementations.

An interface is simpler and more appropriate.

If multiple implementations later shared common logic, an abstract class could be considered.

Principles:

- ISP
- KISS

---

Q2

Why not remove PriceClient entirely and inject HttpPriceClient directly?

Expected Answer

That would still be a valid solution.

I introduced PriceClient because it creates a boundary between business logic and infrastructure.

ShoppingCart should depend on the capability of retrieving prices rather than a specific HTTP implementation.

Principles:

- DIP
- Low Coupling

---

Q3

Could Mockito mock HttpPriceClient directly?

Expected Answer

Yes.

Mockito can mock concrete classes.

The interface was not introduced solely for mocking.

The main reason was dependency inversion and architectural separation.

This is important because many developers incorrectly assume interfaces are required for testing.

---

Q4

What would make you remove PriceClient?

Expected Answer

If this remained a very small exercise and there was no requirement to demonstrate dependency inversion, injecting HttpPriceClient directly would be acceptable.

The abstraction should justify its existence.

Otherwise it becomes accidental complexity.

Principles:

- YAGNI
- KISS

---

ShoppingCart Design

Q5

Why did you choose ShoppingCart instead of ShoppingCartService?

Expected Answer

The requirement asks for a shopping cart.

ShoppingCart is the core business concept.

Using ShoppingCartService would introduce service terminology without providing additional value.

Principles:

- Ubiquitous Language
- KISS

---

Q6

Should ShoppingCart be immutable?

Expected Answer

Not necessarily.

The cart is stateful by nature.

Items are added over time.

Making it immutable would require creating a new cart instance after every operation.

For this exercise the added complexity is not justified.

Trade-off:

- Simplicity over immutability

---

Q7

Could ShoppingCart be thread-safe?

Expected Answer

Not currently.

The requirements do not mention concurrent access.

If required I would evaluate:

- synchronization
- ConcurrentHashMap
- immutable snapshots

I intentionally avoided concurrency concerns because they are outside the scope.

Principles:

- YAGNI

---

Q8

Why did you use Map<String, CartItem>?

Expected Answer

Products are uniquely identified by name.

Map provides:

- O(1) lookup
- O(1) update

The requirements require quantity aggregation.

Map naturally supports that behavior.

---

Q9

Why not Map<Product, CartItem>?

Expected Answer

That would require Product equality and hashCode semantics.

Since products are uniquely identified by name and retrieved externally, String keys keep the implementation simpler.

Principles:

- KISS

---

Tax Calculation

Q10

Why is tax calculation inside ShoppingCart?

Expected Answer

The tax rule is simple and fixed.

Adding a separate abstraction would increase complexity without delivering value.

Principles:

- KISS
- YAGNI

---

Q11

When would you introduce TaxCalculator?

Expected Answer

If tax rules became variable.

Examples:

- country-specific tax
- configurable tax rate
- promotional tax exemptions
- business-specific tax strategies

At that point the abstraction would have immediate value.

---

Q12

Would TaxCalculator violate YAGNI today?

Expected Answer

Yes.

There is only one tax rule and one multiplication.

The abstraction would exist solely for hypothetical future requirements.

---

Testing

Q13

Why separate ShoppingCartTest and HttpPriceClientTest?

Expected Answer

They verify different concerns.

ShoppingCartTest verifies business rules.

HttpPriceClientTest verifies:

- HTTP communication
- response parsing
- error handling

This improves test isolation.

Principles:

- Separation of Concerns

---

Q14

Why not call the real API in tests?

Expected Answer

Real network calls introduce:

- flakiness
- slower execution
- external dependency failures

Tests should be deterministic and repeatable.

MockWebServer gives full control over responses.

---

Q15

What is the difference between unit test and integration test in this solution?

Expected Answer

Unit Test:

ShoppingCartTest

Dependencies mocked.

Tests business behavior only.

Integration Test:

HttpPriceClientTest with MockWebServer.

Tests real HTTP request flow and JSON parsing.

---

Q16

Why not mock HttpClient instead of using MockWebServer?

Expected Answer

Mocking HttpClient tests implementation details.

MockWebServer tests behavior closer to real execution.

It provides greater confidence while remaining deterministic.

---

Q17

How much test coverage is enough?

Expected Answer

Coverage percentage alone is not the goal.

I focus on covering:

- business rules
- edge cases
- error handling
- critical paths

Meaningful coverage is more important than numerical coverage.

---

Error Handling

Q18

Why create ProductNotFoundException?

Expected Answer

404 represents a business-level problem.

A custom exception communicates intent better than generic exceptions.

It improves readability and error handling.

Principles:

- Expressive Design

---

Q19

Why not return Optional<Product>?

Expected Answer

The cart cannot proceed without a valid product.

A missing product is an exceptional condition rather than an optional value.

Exception handling better communicates that failure.

---

Q20

What should happen if the API returns 500?

Expected Answer

The client should fail fast and surface an exception.

The cart cannot reliably continue without price information.

Principles:

- Fail Fast

---

API Integration

Q21

What if the API becomes slow?

Expected Answer

Potential solutions:

- caching
- retries
- timeout configuration

I intentionally omitted them because they are not required.

Principles:

- YAGNI

---

Q22

Would you implement retries?

Expected Answer

Not automatically.

Retries depend on:

- failure type
- idempotency
- business requirements

Adding retries without requirements can create hidden issues.

---

Q23

Would you cache product prices?

Expected Answer

Possibly in a real system.

However caching introduces:

- invalidation complexity
- stale data concerns

For this exercise simplicity is preferable.

---

SOLID Deep Dive

Q24

Which SOLID principle provided the most value?

Expected Answer

Dependency Inversion.

PriceClient separates business logic from infrastructure.

This improves both testability and maintainability.

---

Q25

Which SOLID principle did you intentionally not maximize?

Expected Answer

Open Closed Principle.

I avoided introducing extension points that were not required.

Over-applying OCP can lead to unnecessary abstractions.

Principles balanced:

- OCP
- YAGNI
- KISS

---

Q26

Can too much SOLID be harmful?

Expected Answer

Yes.

Excessive abstractions can make code:

- harder to understand
- harder to navigate
- harder to maintain

Good design balances SOLID with simplicity.

---

Refactoring Questions

Q27

If the cart supported discount coupons, what would change?

Expected Answer

Potentially introduce:

DiscountPolicy

Examples:

- PercentageDiscount
- FixedAmountDiscount

This is a legitimate extension point because multiple behaviors now exist.

---

Q28

If products had categories and category-specific taxes?

Expected Answer

Tax calculation would likely move into a dedicated strategy abstraction.

Different tax behaviors would justify separation.

---

Q29

If products were identified by productId instead of name?

Expected Answer

Replace:

Map<String, CartItem>

with:

Map<ProductId, CartItem>

Domain identity becomes explicit.

---

Q30

What would you refactor first if requirements doubled tomorrow?

Expected Answer

I would first identify areas with changing business rules.

Likely candidates:

- pricing retrieval
- discounts
- taxation

I would avoid speculative refactoring until actual requirements emerge.

Principles:

- YAGNI
- Evolutionary Design

---

Senior-Level Closing Question

Q31

What design principle influenced your implementation most?

Expected Answer

KISS.

The assignment is intentionally small.

I wanted the simplest solution that:

- satisfies requirements
- remains testable
- demonstrates good separation of concerns

Whenever there was a trade-off between flexibility and simplicity, I favored simplicity unless there was immediate value from the abstraction.

That decision was guided by:

- KISS
- YAGNI
- SOLID where justified

---
