1. File: build.gradle

plugins {
    id 'java'
}

group = 'com.equalexperts'
version = '1.0.0'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.20.0'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.13.4'
    testImplementation 'org.mockito:mockito-core:5.19.0'
    testImplementation 'com.squareup.okhttp3:mockwebserver:5.1.0'
}

test {
    useJUnitPlatform()
}

2. Product.java

src/main/java/com/equalexperts/cart/model/Product.java

package com.equalexperts.cart.model;

import java.math.BigDecimal;
import java.util.Objects;

public final class Product {

    private final String name;
    private final BigDecimal price;

    public Product(String name, BigDecimal price) {
        this.name = Objects.requireNonNull(name, "name must not be null");
        this.price = Objects.requireNonNull(price, "price must not be null");
    }

    public String getName() {
        return name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }

        if (!(o instanceof Product product)) {
            return false;
        }

        return Objects.equals(name, product.name)
                && Objects.equals(price, product.price);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, price);
    }

    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}

3. CartItem.java

src/main/java/com/equalexperts/cart/model/CartItem.java
package com.equalexperts.cart.model;

import java.util.Objects;

public class CartItem {

    private final Product product;
    private int quantity;

    public CartItem(Product product, int quantity) {
        this.product = Objects.requireNonNull(product, "product must not be null");

        if (quantity <= 0) {
            throw new IllegalArgumentException("quantity must be greater than zero");
        }

        this.quantity = quantity;
    }

    public Product getProduct() {
        return product;
    }

    public int getQuantity() {
        return quantity;
    }

    public void increaseQuantity(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("quantity must be greater than zero");
        }

        this.quantity += quantity;
    }
}

4. CartSummary.java

src/main/java/com/equalexperts/cart/model/CartSummary.java
package com.equalexperts.cart.model;

import java.math.BigDecimal;
import java.util.Objects;

public final class CartSummary {

    private final BigDecimal subtotal;
    private final BigDecimal tax;
    private final BigDecimal total;

    public CartSummary(BigDecimal subtotal,
                       BigDecimal tax,
                       BigDecimal total) {

        this.subtotal = Objects.requireNonNull(subtotal,
                "subtotal must not be null");
        this.tax = Objects.requireNonNull(tax,
                "tax must not be null");
        this.total = Objects.requireNonNull(total,
                "total must not be null");
    }

    public BigDecimal getSubtotal() {
        return subtotal;
    }

    public BigDecimal getTax() {
        return tax;
    }

    public BigDecimal getTotal() {
        return total;
    }

    @Override
    public String toString() {
        return "CartSummary{" +
                "subtotal=" + subtotal +
                ", tax=" + tax +
                ", total=" + total +
                '}';
    }
}

5. PriceClient.java
src/main/java/com/equalexperts/cart/client/PriceClient.java
package com.equalexperts.cart.client;

import com.equalexperts.cart.model.Product;

public interface PriceClient {

    Product getProduct(String productName);
}

6. ProductNotFoundException.java
src/main/java/com/equalexperts/cart/exception/ProductNotFoundException.java

package com.equalexperts.cart.exception;

public class ProductNotFoundException extends RuntimeException {

    public ProductNotFoundException(String productName) {
        super("Product not found: " + productName);
    }

    public ProductNotFoundException(String productName, Throwable cause) {
        super("Product not found: " + productName, cause);
    }
}

7. HttpPriceClient.java
package com.equalexperts.cart.client;

import com.equalexperts.cart.exception.ProductNotFoundException;
import com.equalexperts.cart.model.Product;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;
import java.math.BigDecimal;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class HttpPriceClient implements PriceClient {

    private final String baseUrl;
    private final HttpClient httpClient;
    private final ObjectMapper objectMapper;

    public HttpPriceClient(String baseUrl,
                           HttpClient httpClient,
                           ObjectMapper objectMapper) {
        this.baseUrl = baseUrl;
        this.httpClient = httpClient;
        this.objectMapper = objectMapper;
    }

    @Override
    public Product getProduct(String productName) {

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(
                        baseUrl +
                        "/backend-take-home-test-data/" +
                        productName +
                        ".json"))
                .GET()
                .build();

        try {
            HttpResponse<String> response =
                    httpClient.send(
                            request,
                            HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() == 404) {
                throw new ProductNotFoundException(productName);
            }

            if (response.statusCode() != 200) {
                throw new IllegalStateException(
                        "Unexpected response status: "
                                + response.statusCode());
            }

            ProductResponse productResponse =
                    objectMapper.readValue(
                            response.body(),
                            ProductResponse.class);

            return new Product(
                    productName,
                    BigDecimal.valueOf(productResponse.price()));
        }
        catch (IOException e) {
            throw new IllegalStateException(
                    "Failed to parse product response",
                    e);
        }
        catch (InterruptedException e) {
            Thread.currentThread().interrupt();

            throw new IllegalStateException(
                    "Request interrupted",
                    e);
        }
    }

    private record ProductResponse(
            String title,
            double price) {
    }
}

8. ShoppingCart.java
src/main/java/com/equalexperts/cart/service/ShoppingCart.java

package com.equalexperts.cart.service;

import com.equalexperts.cart.client.PriceClient;
import com.equalexperts.cart.model.CartItem;
import com.equalexperts.cart.model.CartSummary;
import com.equalexperts.cart.model.Product;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

public class ShoppingCart {

    private static final BigDecimal TAX_RATE =
            new BigDecimal("0.125");

    private final PriceClient priceClient;

    private final Map<String, CartItem> items =
            new HashMap<>();

    public ShoppingCart(PriceClient priceClient) {
        this.priceClient =
                Objects.requireNonNull(
                        priceClient,
                        "priceClient must not be null");
    }

    public void addItem(String productName, int quantity) {

        if (quantity <= 0) {
            throw new IllegalArgumentException(
                    "quantity must be greater than zero");
        }

        Product product =
                priceClient.getProduct(productName);

        CartItem existingItem =
                items.get(productName);

        if (existingItem != null) {
            existingItem.increaseQuantity(quantity);
            return;
        }

        items.put(
                productName,
                new CartItem(product, quantity));
    }

    public Map<String, CartItem> getItems() {
        return Collections.unmodifiableMap(items);
    }

    public CartSummary getSummary() {

        BigDecimal subtotal = items.values()
                .stream()
                .map(item ->
                        item.getProduct()
                                .getPrice()
                                .multiply(
                                        BigDecimal.valueOf(
                                                item.getQuantity())))
                .reduce(
                        BigDecimal.ZERO,
                        BigDecimal::add);

        subtotal = subtotal.setScale(
                2,
                RoundingMode.HALF_UP);

        BigDecimal tax = subtotal
                .multiply(TAX_RATE)
                .setScale(
                        2,
                        RoundingMode.HALF_UP);

        BigDecimal total = subtotal
                .add(tax)
                .setScale(
                        2,
                        RoundingMode.HALF_UP);

        return new CartSummary(
                subtotal,
                tax,
                total);
    }
}

9. ShoppingCartTest.java
src/test/java/com/equalexperts/cart/service/ShoppingCartTest.java

package com.equalexperts.cart.service;

import com.equalexperts.cart.client.PriceClient;
import com.equalexperts.cart.exception.ProductNotFoundException;
import com.equalexperts.cart.model.CartSummary;
import com.equalexperts.cart.model.Product;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class ShoppingCartTest {

    private PriceClient priceClient;
    private ShoppingCart shoppingCart;

    @BeforeEach
    void setUp() {
        priceClient = mock(PriceClient.class);
        shoppingCart = new ShoppingCart(priceClient);
    }

    @Test
    void addItem_whenValidProduct_thenProductAddedToCart() {

        // Given
        Product product =
                new Product(
                        "cornflakes",
                        new BigDecimal("2.52"));

        when(priceClient.getProduct("cornflakes"))
                .thenReturn(product);

        // When
        shoppingCart.addItem("cornflakes", 1);

        // Then
        assertEquals(
                1,
                shoppingCart.getItems()
                        .get("cornflakes")
                        .getQuantity());
    }

    @Test
    void addItem_whenSameProductAddedTwice_thenQuantityAggregated() {

        // Given
        Product product =
                new Product(
                        "cornflakes",
                        new BigDecimal("2.52"));

        when(priceClient.getProduct("cornflakes"))
                .thenReturn(product);

        // When
        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("cornflakes", 1);

        // Then
        assertEquals(
                2,
                shoppingCart.getItems()
                        .get("cornflakes")
                        .getQuantity());
    }

    @Test
    void getSummary_whenCartContainsItems_thenReturnsCorrectSubtotal() {

        // Given
        Product cornflakes =
                new Product(
                        "cornflakes",
                        new BigDecimal("2.52"));

        Product weetabix =
                new Product(
                        "weetabix",
                        new BigDecimal("9.98"));

        when(priceClient.getProduct("cornflakes"))
                .thenReturn(cornflakes);

        when(priceClient.getProduct("weetabix"))
                .thenReturn(weetabix);

        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("weetabix", 1);

        // When
        CartSummary summary =
                shoppingCart.getSummary();

        // Then
        assertEquals(
                new BigDecimal("15.02"),
                summary.getSubtotal());
    }

    @Test
    void getSummary_whenCartContainsItems_thenReturnsCorrectTax() {

        // Given
        Product cornflakes =
                new Product(
                        "cornflakes",
                        new BigDecimal("2.52"));

        Product weetabix =
                new Product(
                        "weetabix",
                        new BigDecimal("9.98"));

        when(priceClient.getProduct("cornflakes"))
                .thenReturn(cornflakes);

        when(priceClient.getProduct("weetabix"))
                .thenReturn(weetabix);

        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("weetabix", 1);

        // When
        CartSummary summary =
                shoppingCart.getSummary();

        // Then
        assertEquals(
                new BigDecimal("1.88"),
                summary.getTax());
    }

    @Test
    void getSummary_whenCartContainsItems_thenReturnsCorrectTotal() {

        // Given
        Product cornflakes =
                new Product(
                        "cornflakes",
                        new BigDecimal("2.52"));

        Product weetabix =
                new Product(
                        "weetabix",
                        new BigDecimal("9.98"));

        when(priceClient.getProduct("cornflakes"))
                .thenReturn(cornflakes);

        when(priceClient.getProduct("weetabix"))
                .thenReturn(weetabix);

        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("cornflakes", 1);
        shoppingCart.addItem("weetabix", 1);

        // When
        CartSummary summary =
                shoppingCart.getSummary();

        // Then
        assertEquals(
                new BigDecimal("16.90"),
                summary.getTotal());
    }

    @Test
    void addItem_whenQuantityIsZero_thenThrowsIllegalArgumentException() {

        // When / Then
        assertThrows(
                IllegalArgumentException.class,
                () -> shoppingCart.addItem(
                        "cornflakes",
                        0));
    }

    @Test
    void addItem_whenQuantityIsNegative_thenThrowsIllegalArgumentException() {

        // When / Then
        assertThrows(
                IllegalArgumentException.class,
                () -> shoppingCart.addItem(
                        "cornflakes",
                        -1));
    }

    @Test
    void addItem_whenProductDoesNotExist_thenThrowsProductNotFoundException() {

        // Given
        when(priceClient.getProduct("unknown-product"))
                .thenThrow(
                        new ProductNotFoundException(
                                "unknown-product"));

        // When / Then
        assertThrows(
                ProductNotFoundException.class,
                () -> shoppingCart.addItem(
                        "unknown-product",
                        1));
    }
}

10. HttpPriceClientTest.java
src/test/java/com/equalexperts/cart/client/HttpPriceClientTest.java

package com.equalexperts.cart.client;

import com.equalexperts.cart.exception.ProductNotFoundException;
import com.equalexperts.cart.model.Product;
import com.fasterxml.jackson.databind.ObjectMapper;
import okhttp3.mockwebserver.MockResponse;
import okhttp3.mockwebserver.MockWebServer;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.net.http.HttpClient;

import static org.junit.jupiter.api.Assertions.*;

class HttpPriceClientTest {

    private MockWebServer mockWebServer;
    private HttpPriceClient priceClient;

    @BeforeEach
    void setUp() throws Exception {

        mockWebServer = new MockWebServer();
        mockWebServer.start();

        priceClient = new HttpPriceClient(
                mockWebServer.url("/").toString(),
                HttpClient.newHttpClient(),
                new ObjectMapper());
    }

    @AfterEach
    void tearDown() throws Exception {
        mockWebServer.shutdown();
    }

    @Test
    void getProduct_whenProductExists_thenReturnsProduct() {

        // Given
        mockWebServer.enqueue(
                new MockResponse()
                        .setResponseCode(200)
                        .setBody("""
                                {
                                  "title":"Corn Flakes",
                                  "price":2.52
                                }
                                """));

        // When
        Product product =
                priceClient.getProduct("cornflakes");

        // Then
        assertEquals(
                "cornflakes",
                product.getName());

        assertEquals(
                0,
                product.getPrice()
                        .compareTo(
                                new java.math.BigDecimal("2.52")));
    }

    @Test
    void getProduct_whenApiReturns404_thenThrowsProductNotFoundException() {

        // Given
        mockWebServer.enqueue(
                new MockResponse()
                        .setResponseCode(404));

        // When / Then
        assertThrows(
                ProductNotFoundException.class,
                () -> priceClient.getProduct(
                        "unknown-product"));
    }

    @Test
    void getProduct_whenApiReturnsInvalidJson_thenThrowsException() {

        // Given
        mockWebServer.enqueue(
                new MockResponse()
                        .setResponseCode(200)
                        .setBody("invalid-json"));

        // When / Then
        assertThrows(
                IllegalStateException.class,
                () -> priceClient.getProduct(
                        "cornflakes"));
    }

    @Test
    void getProduct_whenApiReturnsUnexpectedStatus_thenThrowsException() {

        // Given
        mockWebServer.enqueue(
                new MockResponse()
                        .setResponseCode(500));

        // When / Then
        assertThrows(
                IllegalStateException.class,
                () -> priceClient.getProduct(
                        "cornflakes"));
    }
}

---

---


1. addItem_whenValidProduct_thenProductAddedToCart

2. addItem_whenSameProductAddedTwice_thenQuantityAggregated

3. getSummary_whenCartContainsItems_thenReturnsCorrectSubtotal

4. getSummary_whenCartContainsItems_thenReturnsCorrectTax

5. getSummary_whenCartContainsItems_thenReturnsCorrectTotal

6. getSummary_whenCartIsEmpty_thenReturnsZeroValues

7. addItem_whenQuantityIsZero_thenThrowsIllegalArgumentException

8. addItem_whenQuantityIsNegative_thenThrowsIllegalArgumentException

9. addItem_whenProductDoesNotExist_thenThrowsProductNotFoundException

10. addItem_whenQuantityIsZero_thenDoesNotCallPriceClient
