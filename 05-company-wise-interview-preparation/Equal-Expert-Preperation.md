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
