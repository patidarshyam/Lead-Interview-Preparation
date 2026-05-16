LT Preperation : 
-----------
Question 1: What is the exact console output of the following Java snippet executing different behaviors of the Optional API?


String str1 = "abcds";
String str2 = null;

Optional<String> opt1 = Optional.empty();
Optional<String> opt2 = Optional.of(str1);

System.out.println(opt1);
System.out.println(opt2);
System.out.println(Optional.ofNullable(str2));

The correct answer is  - Optional.empty | Optional[abcds] | Optional.empty
-----------------
Question 2: Given a stream of letters ("A", "B", "C", "D"), what is the expected console output of the following Java code?

import java.util.stream.Stream;

public class StreamTest {
    public static void main(String[] args) {
        String result = Stream.of("A", "B", "C", "D")
                              .peek(System.out.print)
                              .findAny()
                              .orElse("Empty");
                              
        System.out.print(" -> " + result);
    }
}

Options:
1 - A -> A
2 - ABCD -> A
3 - A -> Empty
4 - ABCD -> ABCD

The correct answer is 1 - A -> A
---------------------
Question 3: You are optimizing a large enterprise web application to improve initial page load times. To implement code splitting, you need to refactor a static import inside app.js to a lazy-loaded dynamic import.

Original Code:
javascript// app.js
import { myFunction } from './myStuff';

console.log(myFunction());

Which of the following modifications correctly implements the dynamic import syntax?

Options:
1 - import("./myStuff").then(module => { console.log(module.myFunction()); });

2 - const module = import("./myStuff"); console.log(module.myFunction());

3 - import("./myStuff").then(module => { console.log(myFunction()); });

4 - import("./myStuff").then({ myFunction } => { console.log(myFunction()); });Answer Key & ExplanationsThe correct answer is 1 - import("./myStuff").then(module => { console.log(module.myFunction()); });

The correct answer is 1 - import("./myStuff").then(module => { console.log(module.myFunction()); });
--------------------

Question 4: You need to return multiple adjacent JSX sibling elements from a React 16 component's render function. You want to accomplish this cleanly without introducing unnecessary, bloated HTML wrapper elements (like a <div>) into the rendered DOM tree. Which of the following is the most optimal way to handle this scenario?

Options:
1 - Wrap the adjacent elements inside a traditional <div> wrapper.
2 - Wrap the elements in a standard JavaScript array inside the render function.
3 - Use a React Fragment (<React.Fragment> or <></>).
4 - Wrap the elements using a High-Order Component (HOC).

The correct answer is 3 - Use a React Fragment (<React.Fragment> or <></>)
------------------

Question 5: Which of the following behaviors is expected when a component subscribes to a context object created via createContext?

Options:
1 - The component will read the current context value from the closest matching provider in the tree above it.
2 - The component will read the current context value from the first matching provider in the tree from the top down.
3 - The component will read the current context value from the closest matching provider in the tree below it.
4 - The component will look for a provider below it, and fallback to using the default value as an inline provider

The correct answer is 1 - The component will read the current context value from the closest matching provider in the tree above it
---------------
Question 6: Create calculator with using exceptional handling.

Code:
```
import java.util.*;

public class Main {

    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        try {

            String input = sc.nextLine();
            String[] arr = input.split(" ");

            if (arr.length != 3) {
                throw new IllegalArgumentException("Invalid operation.");
            }

            double num1 = Double.parseDouble(arr[0]);
            String op = arr[1];
            double num2 = Double.parseDouble(arr[2]);

            double result;

            switch (op) {

                case "+":
                    result = num1 + num2;
                    break;

                case "-":
                    result = num1 - num2;
                    break;

                case "*":
                    result = num1 * num2;
                    break;

                case "/":
                    if (num2 == 0) {
                        throw new ArithmeticException("Division by zero is not allowed.");
                    }
                    result = num1 / num2;
                    break;

                default:
                    throw new IllegalArgumentException("Invalid operation.");
            }

            System.out.println("Result: " + result);

        } catch (ArithmeticException e) {

            System.out.println("ArithmeticException: " + e.getMessage());

        } catch (NumberFormatException e) {

            System.out.println("NumberFormatException: Invalid input. Please enter valid numbers.");

        } catch (IllegalArgumentException e) {

            System.out.println("IllegalArgumentException: " + e.getMessage());

        } finally {

            sc.close();
        }
    }
}
```
-----------------------

Question 7: When migrating a legacy XML-based Spring MVC application (mvc-dispatcher-servlet.xml) to a modern Java-based configuration class using a custom configuration adapter, which annotation and method overrides should be paired together to correctly enable the MVC framework, register static asset handlers, and delegate unhandled paths back to the container's default servlet?

Options:
1 - 
@Component
public class WebConfig {
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    }
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}

2 - 
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    }
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}

3 - 
@Configuration
@AutoConfigureMvc
public class WebConfig extends WebMvcAdapter {
    @Override
    public void addResourceLocations(ResourceRegistry registry) {
        registry.addPath("/static/**").toFolder("/static/");
    }
    @Override
    public void enableDefaultServlet(ServletConfigurer configurer) {
        configurer.activate();
    }
}

4 - 
@Component
@EnableMvcSupport
public class WebConfig implements MvcConfigurerAdapter {
    public void addResourceHandler(ResourceRegistry registry) {
        registry.addHandler("/static/**");
    }
    public void configureServletHandling(DefaultServletConfigurer configurer) {
        configurer.enable();
    }
}


The correct answer is 2 - 
Option 2 is correct because it provides a complete, modern Java configuration class:
It uses @Configuration to register as a bean definition source.
It includes @EnableWebMvc, which triggers the import of foundational Spring MVC components.
It correctly implements the WebMvcConfigurer interface (which replaces the deprecated WebMvcConfigurerAdapter in modern Spring frameworks).
The methods addResourceHandlers and configureDefaultServletHandling are perfectly named and use correct parameter registries to map static resources and enable the container's default servlet.
---------------

Question 8: You need to configure a global CORS policy for your Spring Boot backend application to handle incoming cross-origin requests. The requirements state that it must apply to all API endpoints (/api/**), allow requests originating from any domain, and support all standard HTTP methods.
Which of the following code snippets implements this global configuration correctly using Spring MVC?

1 - 
@Configuration
public class CorsConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                        .allowedOrigins("*")
                        .allowedMethods("*");
            }
        };
    }
}

2 - 
@Component
public class CorsConfig {
    @Bean
    public void configureCors(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .setOrigins("*")
                .setMethods("*");
    }
}

3 - 
@Configuration
public class CorsConfig implements CorsMappingAdapter {
    @Override
    public CorsConfiguration proceedCors() {
        CorsConfiguration config = new CorsConfiguration();
        config.addMapping("/api/**");
        config.allowAllOrigins();
        return config;
    }
}

4 -
@Component
@EnableCorsRegistry
public class CorsConfig {
    @Override
    public void addCorsMapping(CorsRegistry registry) {
        registry.register("/api/**")
                .allowOrigin("*")
                .allowMethod("*");
    }
}

The correct answer is 1.
--------------------

Question 9: You are building a notification engine that must listen to webhook status updates from Meta, process them in Express.js, and immediately fan out those alerts to multiple AWS SQS queues and third-party endpoints. Which AWS architectural pattern natively supports this decoupled "Fan-Out" strategy?

Options:
1 - Send the payloads directly to an Amazon SQS FIFO queue and poll it from multiple instances.
2 - Publish the event message to an Amazon SNS Topic and subscribe the targeted endpoints/queues to it.
3 - Insert the JSON documents into an Amazon DynamoDB Table and track changes with Streams.
4 - Pipe the network stream through an AWS Transit Gateway configured for UDP multicasting

Correct Answer: 2 - Publish the event message to an Amazon SNS Topic and subscribe the targeted endpoints/queues to it.
-----------------

Question 10: A large e-commerce platform is built using a microservices architecture. During a peak traffic event, the product recommendation service (Service XYZ) encounters an unhandled exception and crashes completely, causing its API endpoints to time out.To maintain maximum system availability and adhere to resilient system design principles, how should the main website application handle this microservice failure?

Options:
1 - The website should be made unavailable by routing users to a global maintenance page until Service XYZ is fully restored.
2 - The website should remain operational, and the product recommendation functionality should be gracefully degraded (e.g., displaying static popular items or hiding the section).
3 - The website should remain operational by dynamically switching Service XYZ into a mock testing mode that returns dummy hardcoded database payloads to live production users.
4 - The website should remain operational by automatically compiling and deploying a brand-new alternative microservice with the same functionality into the production cluster.


The correct answer is 2 - The website should remain operational, and the product recommendation functionality should be gracefully degraded (e.g., displaying static popular items or hiding the section).
-------------

Question 11: Avoid a system failure, it is given that there is an unresponsive remote call in your microservice-oriented application which has to make a call to the service. This has somehow lead to execution of critical resources which could cause potential system failure. Implement which of the architecture patterns will help you to avoid the system failure?
Patterns:
1. Client-side discovery pattern
2. Client-side load balancer pattern
3. Events and event-driven async pattern

Options:
choice 1
choice 2
choice 1 and 2
choice 2 and 3

The correct answer is choice 2 and 3.
------------------

Question 12: You want to improve the latency performance of your microservices application, which frequently makes a large number of remote service calls. What architectural optimization can you implement to achieve this goal without introducing significant complexity to your programming model?

Options:
1 - Use asynchronous remote calls and reduce the granularity of remote calls by batching multiple requests into a single call.
2 - Use synchronous remote calls and increase the granularity of remote calls by making individual sequential requests.
3 - Switch the entire infrastructure to a single monolithic database instance to eliminate network serialization overhead.
4 - Implement a distributed global dynamic lock across all microservice instances to synchronize data lookups.

The correct answer is 1 - Use asynchronous remote calls and reduce the granularity of remote calls by batching multiple requests into a single call.

----------------

Question 13: You are optimizing a relational database query that checks for non-existent records across two tables. The original query is written as:
SELECT * FROM t1 WHERE NOT EXISTS (SELECT id FROM t2 WHERE t1.id = t2.id);
Which of the following alternatives represents a valid, equivalent query optimization technique?

Options:
Option A: SELECT t1.* FROM t1 LEFT JOIN t2 ON t1.id = t2.id WHERE t2.id IS NULL;
Option B: SELECT * FROM t1 WHERE id NOT IN (SELECT id FROM t2 WHERE id IS NOT NULL);
Option C: SELECT t1.* FROM t1 INNER JOIN t2 ON t1.id = t2.id WHERE t2.id IS NULL;
Option D: SELECT * FROM t1 WHERE id NOT IN (SELECT id FROM t2);

The correct answer is Option A.
------------------

Question 14: You have created a student table that contains individual subject marks, a total column, and an average column. You want to create a database trigger that automatically calculates and populates the total and average values immediately before a new student record is inserted into the database.Which of the following syntax structures represents the correct way to configure this trigger statement?

Options:

1 -
CREATE TRIGGER calc_marks 
BEFORE INSERT ON student 
FOR EACH ROW 
BEGIN 
    SET NEW.total = NEW.mark1 + NEW.mark2 + NEW.mark3;
    SET NEW.average = NEW.total / 3;
END;

2 - 
CREATE TRIGGER calc_marks 
AFTER INSERT ON student 
FOR EACH ROW 
BEGIN 
    SET TRIGGER.total = TRIGGER.mark1 + TRIGGER.mark2;
    SET TRIGGER.average = TRIGGER.total / 2;
END;

3 -
CREATE TRIGGER calc_marks 
BEFORE INSERT ON student 
FOR EACH TRIGGERED STUDENT 
BEGIN 
    SET total = mark1 + mark2;
    SET average = total / 2;
END;

4 -
CREATE TRIGGER calc_marks 
INSTEAD OF INSERT ON student 
FOR EACH ROW 
BEGIN 
    UPDATE student SET total = mark1 + mark2;
END;

The correct answer is 1.
----------------

Question 15: You are writing a unit test suite in JUnit 5. You define a test method annotated with @RepeatedTest(5) to execute a specific test case five times consecutively. You also include a setup method in the same test class that needs to execute immediately before each individual repetition to ensure a clean state.
Which annotation should be used on this setup method to achieve this behavior?

Options:
1 - @BeforeEach
2 - @Before
3 - @BeforeAll
4 - @BeforeRepeated

correct answer is 1 - @BeforeEach
---------------

Question 16: You are annotating your test class with @TestInstance(TestInstance.Lifecycle.PER_CLASS). Changing the test instance lifecycle to "per-class" allows JUnit 5 to instantiate the test class exactly once for executing all its test cases. In this specific configuration, which of the following lifecycle annotations can be used on non-static methods within the class?

1) @BeforeAll
2) @BeforeClass
3) @BeforeEach

Options:
1. only one, two and three
2. one and two
3. one and three
4. only two

The correct answer 3) is one and three
---------------------

Question 17: You are writing a JUnit 5 test class. The test lifecycle requires connecting to a database resource, performing test assertions, and then completely cleaning up or tearing down the generated test data and activities to maintain database integrity.Which of the following annotations should be used on a method to ensure it runs to clean up and clear out data after test execution?

Options:
1 - @AfterEach (or @AfterAll depending on scope)
2 - @CleanUp
3 - @Teardown
4 - @ClearData

The correct answer is 1 - @AfterEach (or @AfterAll depending on scope).
-------------------

Question 18: You are designing a JUnit 5 test class. The execution lifecycle requires connecting to resources, executing assertions, and then running a method to completely flush out and clean up the test data.Assuming you are using the correct JUnit 5 lifecycle annotations (@AfterEach or @AfterAll), which of the following represents standard, idiomatic method names typically implemented by developers to perform this teardown logic?

Options:
1 - cleanUp() or tearDown()
2 - clear() or destroy()
3 - flushData() or reset()
4 - purge() or remove()

correct answer is 1 - cleanUp() or tearDown().
-----------

Question 19: You are setting up a state management layer in a React application using the Context API. You instantiate a new context object using the following declaration pattern:

const MyContext = React.createContext(defaultValue);

Which of the following values can be passed as a valid defaultValue or runtime provider value in React?

null, undefined, 0 , props

Options:
only 1 and 2
only 1, 2, and 3
1, 2, 3, and 4
only 4
 
The correct answer is 1, 2, 3, and 4.
--------------

Question 20: You are introducing code splitting into your web application via native ES dynamic import() expressions. To maintain compatibility with older runtime environments, you need to configure your Babel pipeline. The strategy requires first allowing Babel's parser to accept the dynamic syntax without transpiling it, followed by applying a plugin to handle the final module format transformation.

Which of the following plugin pairings correctly configures your .babelrc configuration file to accomplish this split lifecycle?
Options:
1 - Use @babel/plugin-syntax-dynamic-import to parse the code, followed by @babel/plugin-transform-dynamic-import to transpile it.

2 - Use @babel/plugin-parse-import to parse the code, followed by @babel/plugin-compile-dynamic-import to transpile it.

3 - Use @babel/plugin-ignore-transform to parse the code, followed by @babel/plugin-execute-lazy-import to transpile it.

4 - Use @babel/plugin-extract-syntax to parse the code, followed by @babel/plugin-convert-modules to transpile it

The correct answer is 1 - Use @babel/plugin-syntax-dynamic-import to parse the code, followed by @babel/plugin-transform-dynamic-import to transpile it.
-----------

Question 21: You are deploying a Server-Side Rendered (SSR) web application using modern React (such as Next.js). You notice that dynamic text content (like a local client timestamp or dark mode theme attribute) causes non-breaking but noisy hydration warnings in the browser console. Additionally, you want to selectively disable other static code rules during the component lifecycle.

Which of the following built-in element properties can be configured to selectively ignore these native UI mismatch warnings without disabling logs globally?

Options:
1 - suppressHydrationWarning={true}
2 - removeAllWarnings={true}
3 - suppressAddition={true}
4 - removeHighResolution={true}

The correct answer is 1 - suppressHydrationWarning={true}.

--------------------------
=====================================
