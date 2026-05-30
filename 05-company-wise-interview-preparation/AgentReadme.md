# Equal Experts Shopping Cart

## Overview

This project implements a simple shopping cart library in Java 17.

Supported capabilities:

- Add products to a shopping cart
- Retrieve product pricing from the provided Price API
- Maintain cart state
- Calculate:
  - Subtotal
  - Tax (12.5%)
  - Total payable

The implementation focuses on simplicity, testability, and clean design while avoiding unnecessary abstractions.

---

## Technology Stack

- Java 17
- Gradle
- JUnit 5
- Mockito
- Jackson
- MockWebServer

---

## Project Structure

```text
src/main/java/com/equalexperts/cart

├── model
│   ├── Product
│   ├── CartItem
│   └── CartSummary
│
├── service
│   └── ShoppingCart
│
├── client
│   ├── PriceClient
│   └── HttpPriceClient
│
└── exception
    └── ProductNotFoundException
```

---

## Functional Requirements Implemented

### Add Product

Products can be added to the cart using:

```java
shoppingCart.addItem("cornflakes", 1);
```

When a product is added:

- Product price is retrieved from the external Price API
- Product is stored in the cart
- Existing quantities are aggregated

---

### Cart State

Current cart state is available through:

```java
shoppingCart.getItems();
```

---

### Cart Summary

Summary calculations are available through:

```java
shoppingCart.getSummary();
```

The summary contains:

- Subtotal
- Tax
- Total

---

## Price API

Base URL:

```text
https://equalexperts.github.io/
```

Endpoint:

```text
GET /backend-take-home-test-data/{product}.json
```

Example:

```text
GET /backend-take-home-test-data/cornflakes.json
```

---

## Design Principles

The solution intentionally follows:

### KISS (Keep It Simple)

The implementation avoids unnecessary frameworks, layers, and abstractions.

### YAGNI (You Aren't Gonna Need It)

Only functionality required by the exercise has been implemented.

Examples intentionally omitted:

- Database persistence
- Caching
- Retry mechanisms
- Concurrency support
- Tax strategy abstractions

### SOLID

#### Single Responsibility Principle (SRP)

- ShoppingCart manages cart behavior
- HttpPriceClient handles API communication
- Models represent domain data

#### Dependency Inversion Principle (DIP)

ShoppingCart depends on:

```java
PriceClient
```

instead of:

```java
HttpPriceClient
```

This improves testability and reduces coupling.

---

## Design Decisions

### PriceClient Abstraction

A PriceClient interface is used to separate business logic from the HTTP implementation.

Benefits:

- Dependency inversion
- Lower coupling
- Easier testing
- Clear external dependency boundary

Although there is currently only one implementation, the abstraction provides immediate value by isolating business logic from infrastructure concerns.

---

### ShoppingCart

ShoppingCart is responsible for:

- Maintaining cart state
- Aggregating quantities
- Calculating totals
- Coordinating price retrieval

The cart internally stores items using:

```java
Map<String, CartItem>
```

This provides:

- O(1) lookup
- O(1) updates
- Natural quantity aggregation

---

### Product Model

The external API returns a field named:

```json
{
  "title": "Corn Flakes"
}
```

The domain model uses:

```java
Product.name
```

because the assignment consistently refers to products by name.

The API response is mapped into the domain model rather than exposing API-specific terminology throughout the application.

---

### Constructor Injection

Dependencies are supplied through constructors.

Examples:

```java
ShoppingCart(PriceClient priceClient)
```

```java
HttpPriceClient(
    String baseUrl,
    HttpClient httpClient,
    ObjectMapper objectMapper)
```

Benefits:

- Explicit dependencies
- Better testability
- Reduced coupling

---

### HTTP Client Configuration

The API base URL is injected through the constructor.

A configuration file was intentionally not introduced because:

- The exercise has a single environment
- The exercise has a single endpoint
- Constructor injection already provides configurability and testability

This keeps the solution simple while remaining easy to test.

---

### Money Calculations

All monetary calculations use:

```java
BigDecimal
```

Floating point types such as:

```java
double
float
```

are intentionally avoided because they can introduce precision issues.

Example:

```java
0.1 + 0.2 != 0.3
```

when using double.

---

### Tax Calculation

Tax is calculated at a fixed rate of:

```text
12.5%
```

The tax logic remains inside ShoppingCart because introducing a TaxCalculator abstraction would add complexity without providing immediate value.

---

## Assumptions

1. Product quantity must be greater than zero.
2. Quantity less than or equal to zero results in IllegalArgumentException.
3. Product prices are retrieved when addItem() is called.
4. Product not found results in ProductNotFoundException.
5. Cart state is maintained in memory.
6. No caching is implemented.
7. No persistence is implemented.
8. No concurrency support is implemented.
9. The external Price API returns valid product pricing data.

---

## Testing Strategy

Tests are divided into two categories.

### ShoppingCartTest

Verifies business logic:

- Product addition
- Quantity aggregation
- Subtotal calculation
- Tax calculation
- Total calculation
- Empty cart behavior
- Validation scenarios

### HttpPriceClientTest

Verifies client behavior:

- Successful product retrieval
- 404 handling
- Invalid JSON handling
- Unexpected HTTP status handling

MockWebServer is used to avoid dependency on external network calls while exercising real HTTP request and JSON parsing behavior.

---

## Test Naming Convention

Tests follow:

```text
method_whenCondition_thenExpectedResult
```

Examples:

```text
addItem_whenSameProductAddedTwice_thenQuantityAggregated

getSummary_whenCartContainsItems_thenReturnsCorrectTax
```

This makes test intent explicit and improves readability of test reports.

The convention is inspired by Given/When/Then behavior specifications.

---

## Running Tests

Execute:

```bash
./gradlew test
```

All tests should pass successfully.

---

## Trade-offs

### Why No Spring Boot?

The assignment explicitly requests a solution rather than an application.

Adding Spring Boot would introduce framework complexity without solving a requirement.

### Why No Database?

Persistence is not required.

An in-memory implementation satisfies all requirements.

### Why No TaxCalculator?

The tax rule is currently a single fixed calculation.

A separate abstraction would add complexity without providing value.

### Why POJOs Instead of Records?

Java 17 records were considered for Product and CartSummary.

Traditional POJOs were selected because they provide:

- Explicit constructors
- Explicit validation support
- Familiarity for most Java teams
- Consistency across model classes

Both approaches would be valid for this exercise.

---

## Scope Deliberately Excluded

The following scenarios were intentionally not implemented because they are outside the requirements:

- Product price validation beyond the API contract
- Negative product pricing scenarios
- Caching
- Retry mechanisms
- Persistence
- Concurrency handling
- Configurable tax strategies
- Additional application layers

These can be introduced if future requirements justify them.

---

## AI Tool Usage

### Specific Use Cases

AI tools were used for:

- Architecture review
- Design discussions
- Test case generation assistance
- Code generation assistance
- README generation
- Trade-off analysis

### Estimated AI Generated Code

Approximately 60–80% of the implementation was initially AI-assisted.

### Review Process

All AI-generated content was manually reviewed and validated.

The review process included:

- Requirement verification
- Design validation
- Logic verification
- Test verification
- Refactoring decisions
- Trade-off analysis

All final implementation decisions were reviewed and adjusted where necessary before submission.
