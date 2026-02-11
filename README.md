# Allo Bank Backend Developer Take-Home Test

Thank you for applying to our team! This take-home test is designed to evaluate your practical skills in building **production-ready** Spring Boot applications within a finance domain, focusing on architectural patterns and complex data handling.

## 📝 Objective

Your task is to create a single Spring Boot REST API endpoint capable of aggregating data from multiple, distinct resources provided by the public, keyless **Frankfurter Exchange Rate API**. The primary focus is on handling Indonesian Rupiah (IDR) data.

The focus of this test is not just functional correctness, but demonstrating clean code, advanced Spring concepts, thread-safe design, and architectural clarity.

## I. Core Task: The Polymorphic API

### 1. External API Integration (Frankfurter API)

* **Base URL (Public):** `https://api.frankfurter.app/`.

* You must integrate with three distinct data resources to enforce the architectural pattern:

   1.  `/latest?base=IDR` (The latest rates relative to IDR)

   2.  **Historical Data:** Query a specific, small time series (e.g., `/2024-01-01..2024-01-05?from=IDR&to=USD`). **Note:** *Use the date range provided in this example unless a different range is communicated separately.*

   3.  `/currencies` (The list of all supported currency symbols)

### 2. Internal API Endpoint

You must expose **one single endpoint** in your application: ```GET /api/finance/data/{resourceType}```

Where `{resourceType}` can be one of the three strings: `latest_idr_rates`, `historical_idr_usd`, or `supported_currencies`.

### 3. Required Functionality & Business Logic

* **Resource Handling:** Your service must correctly map the three incoming `resourceType` values to the correct data fetching strategies.

* **Data Load:** All three resources should be fetched from the external API.

* **Data Transformation (Latest IDR Rates only) - Unique Calculation:** For the **`latest_idr_rates`** resource, you must calculate and include a new field, `"USD_BuySpread_IDR"`. This is the Rupiah selling rate to USD after applying a banking spread/margin.

  **The Spread Factor Must Be Unique :**

   1.  **Input:** Your GitHub username (e.g., `johndoe47`).
   2.  **Calculation:** Calculate the sum of the Unicode (ASCII) values of all characters in your lowercase GitHub username string.
   3.  **Spread Factor Derivation:** `Spread Factor = (Sum of Unicode Values % 1000) / 100000.0`
       *(This will yield a unique factor between 0.00000 and 0.00999, ensuring a personalized result.)*

  **Final Formula:** `USD_BuySpread_IDR = (1 / Rate_USD) * (1 + Spread Factor)` (where `Rate_USD` is the value from the API when `base=IDR`).

* **Other Resources:** The `historical_idr_usd` and `supported_currencies` resources can return their data with minimal transformation, but the final output must be a unified JSON array of results.

## II. Architectural Constraints

Meeting the core task is only one part of the solution. The following constraints must be strictly adhered to and will be heavily weighted during evaluation:

### Constraint A: The Strategy Pattern

The logic for handling the three different resources (`latest_idr_rates`, `historical_idr_usd`, `supported_currencies`) must be implemented using the **Strategy Design Pattern**.

1.  Define a clear **Strategy Interface** (e.g., `IDRDataFetcher`).

2.  Implement **three concrete strategy classes** (one for each resource).

3.  The main `Controller` should dynamically select the correct strategy implementation using a map-based lookup injected by Spring, avoiding any manual `if/else` or `switch` logic in the controller layer.

### Constraint B: Client Factory Bean

The instance of your chosen external API client (`WebClient` or `RestTemplate`) **must be defined and created within a custom implementation of Spring's `FactoryBean<T>` interface**.

* This `FactoryBean` should be responsible for externalizing the API Base URL via `@Value` or `@ConfigurationProperties` and applying any initial configuration (e.g., timeouts, shared headers).

* ***You may not define the client as a simple `@Bean` in a `@Configuration` class.***

### Constraint C: Startup Data Runner & Immutability

The aggregated data for **ALL three resources** must be fetched **exactly once on application startup** and loaded into an in-memory store.

1.  Use a Spring Boot **`ApplicationRunner`** or **`CommandLineRunner`** component to initiate the data fetching process.

2.  The API endpoint (`GET /api/finance/data/{resourceType}`) must serve the data from this **in-memory store**, not by making a new call to the external API on every request.

3.  The in-memory storage mechanism (e.g., a service holding the data) must be designed to be **thread-safe** and ensure the data is **immutable** once the `ApplicationRunner` has finished loading it.

## III. Production Readiness & Deliverables

Your final solution must demonstrate production quality through code, testing, and communication.

### 1. Robustness & Best Practices

* Graceful **Error Handling** for network failures or 4xx/5xx responses from the external API.

* Proper use of **Configuration Properties** (e.g., `application.yml`) for external service URLs.

* Clear separation of concerns (Controller, Service, Model/DTO, etc.).

### 2. Testing

* **Unit Tests** for all three `IDRDataFetcher` strategy implementations, ensuring data calculation and transformation logic is covered (using mock clients for external calls).

* **Integration Tests** to verify the `ApplicationRunner` successfully initializes and loads the data into the in-memory store before the application context is ready.

### 3. Documentation

A clear `README.md` is mandatory. It must include:

* **Setup/Run Instructions:** Clear steps to clone, build, and run the application and tests.

* **Endpoint Usage:** Example cURL commands to test the three different resource types.

* **Personalization Note:** Clearly state your GitHub username and show the exact **Spread Factor** (e.g., `0.00765`) calculated by your function.

* ---

* ### 🛠️ Architectural Rationale

  This section should contain a brief, but detailed, explanation answering the following questions:

   1.  **Polymorphism Justification:** Explain *why* the Strategy Pattern was used over a simpler conditional block in the service layer for handling the multi-resource endpoint. Discuss the benefits in terms of **extensibility** and **maintainability**.

   2.  **Client Factory:** Explain the specific role and benefit of using a **`FactoryBean`** to construct the external API client. Why is this preferable to defining the client using a standard `@Bean` method in this scenario?

   3.  **Startup Runner Choice:** Justify the choice of using an `ApplicationRunner` (or `CommandLineRunner`) for the initial data ingestion over a simpler `@PostConstruct` method.

## IV. Submission & Review Process

1.  **Fork** this repository.

2.  Implement your solution on a dedicated feature branch (e.g., `feat/idr-rate-aggregator`).

3.  When complete, submit your solution via a **Pull Request (PR)** back to the main repository.
4.  Please complete the form to submit your technical test: [Click Here](https://forms.gle/nZKQ2EjTCPfAKHog7)

**Your PR will be evaluated on the following:**

* **Commit History:** Clean, atomic, and descriptive commit messages (e.g., "feat: Implement IDR latest rates strategy," "fix: Correctly calculate IDR spread in tests").

* **PR Description:** The description must clearly summarize the solution and **must contain the full answers** to the three "Architectural Rationale" questions from Section III.

* **Code Review Readiness:** The code should be well-structured and ready for immediate review.

Good luck!

---

# Implementation

This document describes the implementation of the Spring Boot REST API for aggregating Frankfurter Exchange Rate data.

## 📋 Table of Contents

- [Setup and Run Instructions](#setup-and-run-instructions)
- [Endpoint Usage](#endpoint-usage)
- [Personalization Note](#personalization-note)
- [Architectural Rationale](#architectural-rationale)
- [Project Structure](#project-structure)

## Setup and Run Instructions

### Prerequisites

- Java 17 or higher
- Maven 3.6+ (or use Maven Wrapper)
- Internet connection (for fetching data from Frankfurter API)

### Building the Application

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd allo-backend-test
   ```

2. **Set your GitHub username (optional, defaults to "defaultuser"):**
   ```bash
   # On Windows (PowerShell)
   $env:GITHUB_USERNAME="your-github-username"
   
   # On Linux/Mac
   export GITHUB_USERNAME="your-github-username"
   ```
   
   Alternatively, you can set it in `application.yml`:
   ```yaml
   github:
     username: your-github-username
   ```

3. **Build the project:**
   ```bash
   mvn clean install
   ```

### Running the Application

1. **Run the Spring Boot application:**
   ```bash
   mvn spring-boot:run
   ```
   
   Or if you've built the JAR:
   ```bash
   java -jar target/allo-backend-test-1.0.0.jar
   ```

2. **Wait for initialization:**
   The application will fetch data from the Frankfurter API on startup. Wait for the log message:
   ```
   Data initialization completed. Success: 3, Failures: 0
   ```

3. **Verify the application is running:**
   The application will be available at `http://localhost:8080`

### Running Tests

1. **Run all tests:**
   ```bash
   mvn test
   ```

2. **Run specific test class:**
   ```bash
   mvn test -Dtest=SpreadCalculatorTest
   ```

3. **Run integration tests:**
   ```bash
   mvn test -Dtest=DataInitializationIntegrationTest
   ```

## Endpoint Usage

The application exposes a single REST endpoint that serves aggregated data from the Frankfurter API.

### Base URL
```
http://localhost:8080/api/finance/data
```

### Available Resource Types

1. **Latest IDR Rates** - `latest_idr_rates`
2. **Historical IDR to USD** - `historical_idr_usd`
3. **Supported Currencies** - `supported_currencies`

### Example cURL Commands

#### 1. Get Latest IDR Rates
```bash
curl -X GET "http://localhost:8080/api/finance/data/latest_idr_rates" \
  -H "Accept: application/json"
```

**Expected Response:**
```json
[
  {
    "amount": 1.0,
    "base": "IDR",
    "date": "2024-01-15",
    "rates": {
      "USD": 0.000064,
      "EUR": 0.000059,
      ...
    },
    "USD_BuySpread_IDR": 15625.015625
  }
]
```

#### 2. Get Historical IDR to USD Rates
```bash
curl -X GET "http://localhost:8080/api/finance/data/historical_idr_usd" \
  -H "Accept: application/json"
```

**Expected Response:**
```json
[
  {
    "amount": 1.0,
    "base": "IDR",
    "start_date": "2024-01-01",
    "end_date": "2024-01-05",
    "rates": {
      "2024-01-01": {
        "USD": 0.000064
      },
      "2024-01-02": {
        "USD": 0.000065
      },
      ...
    }
  }
]
```

#### 3. Get Supported Currencies
```bash
curl -X GET "http://localhost:8080/api/finance/data/supported_currencies" \
  -H "Accept: application/json"
```

**Expected Response:**
```json
[
  {
    "currencies": {
      "USD": "United States Dollar",
      "IDR": "Indonesian Rupiah",
      "EUR": "Euro",
      ...
    }
  }
]
```

### Error Responses

#### Invalid Resource Type (400 Bad Request)
```bash
curl -X GET "http://localhost:8080/api/finance/data/invalid_resource"
```

**Response:**
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 400,
  "error": "Invalid Resource Type",
  "message": "Resource type must be one of: latest_idr_rates, historical_idr_usd, supported_currencies",
  "path": "/api/finance/data/invalid_resource"
}
```

#### Data Not Ready (503 Service Unavailable)
This occurs if the endpoint is called before data initialization is complete.

## Personalization Note

**GitHub Username:** `defaultuser` (can be configured via environment variable `GITHUB_USERNAME` or in `application.yml`)

**Spread Factor Calculation:**
- For username `defaultuser` (lowercase): `defaultuser`
- Sum of Unicode values: `100 + 101 + 102 + 97 + 117 + 108 + 116 + 117 + 115 + 101 + 114 = 1086`
- Spread Factor: `(1086 % 1000) / 100000.0 = 86 / 100000.0 = 0.00086`

**Note:** To calculate your own spread factor, set the `GITHUB_USERNAME` environment variable or update `application.yml` with your GitHub username, then restart the application.

## Architectural Rationale

### 1. Polymorphism Justification: Why Strategy Pattern?

The Strategy Pattern was chosen over a simpler conditional block (if/else or switch) for several critical reasons:

**Extensibility:**
- Adding a new resource type requires only creating a new strategy implementation and registering it as a Spring bean. No modification to existing code is needed, following the Open/Closed Principle.
- The controller remains unchanged when new strategies are added, as Spring automatically injects all `IDRDataFetcher` implementations.

**Maintainability:**
- Each strategy encapsulates its own data fetching and transformation logic, making the codebase easier to understand and modify.
- Changes to one resource type's handling don't affect others, reducing the risk of introducing bugs.
- The clear separation of concerns makes unit testing straightforward - each strategy can be tested in isolation.

**Testability:**
- Strategies can be easily mocked and tested independently.
- The controller logic is simplified, focusing only on routing and error handling rather than business logic.

**Type Safety:**
- The interface contract ensures all strategies implement the required methods consistently.
- Compile-time checking prevents missing implementations.

**Spring Integration:**
- Leverages Spring's dependency injection to automatically discover and inject all strategy implementations.
- The map-based lookup in the controller (via Spring's `List<IDRDataFetcher>` injection) eliminates manual strategy registration code.

In contrast, a conditional block would require modifying the controller/service every time a new resource type is added, violating the Open/Closed Principle and making the code harder to maintain as it grows.

### 2. Client Factory: Why FactoryBean?

Using a `FactoryBean` to construct the `WebClient` provides several advantages over a standard `@Bean` method:

**Encapsulation of Complex Creation Logic:**
- The `FactoryBean` encapsulates all the configuration logic for creating the `WebClient` in a single, reusable component.
- This includes base URL configuration, codec settings, timeout configurations, and any future enhancements (retry logic, circuit breakers, etc.).

**Configuration Externalization:**
- The `FactoryBean` uses `@ConfigurationProperties` to externalize all API configuration (base URL, timeouts) to `application.yml`.
- This makes the configuration environment-specific and easily changeable without code modifications.

**Lifecycle Management:**
- `FactoryBean` provides fine-grained control over bean creation lifecycle through `getObject()`, `getObjectType()`, and `isSingleton()` methods.
- The singleton pattern ensures only one `WebClient` instance is created and reused across all strategies, which is more efficient than creating multiple instances.

**Testability:**
- The `FactoryBean` can be easily mocked or replaced in test configurations.
- The separation of factory logic from bean definition makes it easier to test different configurations.

**Future Extensibility:**
- If we need to create different `WebClient` instances for different purposes (e.g., different base URLs, different timeout settings), we can create multiple `FactoryBean` implementations.
- The factory pattern allows for conditional bean creation based on profiles or properties.

**Compliance with Requirements:**
- The requirement explicitly states that the client must be created within a `FactoryBean` implementation, not as a simple `@Bean` method. This ensures consistency and enforces the architectural pattern.

While a simple `@Bean` method would work functionally, the `FactoryBean` approach provides better separation of concerns, easier configuration management, and aligns with Spring's best practices for complex bean creation.

### 3. Startup Runner Choice: ApplicationRunner vs @PostConstruct

Using an `ApplicationRunner` (or `CommandLineRunner`) for initial data ingestion is preferable to `@PostConstruct` for several reasons:

**Application Context Readiness:**
- `ApplicationRunner.run()` is called **after** the entire Spring application context is fully initialized, including all beans, configurations, and lifecycle callbacks.
- `@PostConstruct` methods are called during bean initialization, which may occur before all dependencies are fully wired, potentially causing issues with complex dependency graphs.

**Error Handling and Application Startup:**
- If data initialization fails in an `ApplicationRunner`, the application can still start (depending on error handling), but we have explicit control over this behavior.
- With `@PostConstruct`, failures during bean initialization can prevent the application from starting entirely, which may be too strict for non-critical initialization tasks.

**Asynchronous Operations:**
- `ApplicationRunner` provides a natural place to handle asynchronous operations (like our reactive `WebClient` calls) with proper synchronization mechanisms (e.g., `CountDownLatch`).
- `@PostConstruct` methods are typically synchronous, making it harder to coordinate multiple async operations.

**Testing:**
- `ApplicationRunner` implementations can be easily tested in isolation or excluded from test contexts.
- Integration tests can verify that the runner executes correctly and loads data into the store.

**Lifecycle Clarity:**
- The `ApplicationRunner` makes it explicit that data loading is an application startup concern, not a bean initialization concern.
- This separation makes the codebase more maintainable and the intent clearer.

**Command-Line Arguments:**
- `ApplicationRunner` receives `ApplicationArguments`, which can be useful for conditional initialization based on command-line parameters or profiles.

**Ordering Control:**
- Multiple `ApplicationRunner` implementations can be ordered using `@Order` annotation, providing fine-grained control over initialization sequence if needed.

In our implementation, the `DataInitializationRunner` uses reactive programming with `Mono` and coordinates multiple async operations using `CountDownLatch`, which would be awkward to implement in a `@PostConstruct` method. The runner also provides better error handling and logging, ensuring that partial failures don't prevent the application from starting.

## Project Structure

```
src/
├── main/
│   ├── java/com/allobank/
│   │   ├── AlloBackendTestApplication.java       # Main Spring Boot application
│   │   ├── config/
│   │   │   ├── FrankfurterApiProperties.java     # Configuration properties
│   │   │   └── WebClientConfig.java              # WebClient bean configuration
│   │   ├── controller/
│   │   │   └── FinanceDataController.java        # REST endpoint controller
│   │   ├── dto/
│   │   │   ├── LatestRatesResponse.java          # DTOs for API responses
│   │   │   ├── HistoricalRatesResponse.java
│   │   │   ├── CurrenciesResponse.java
│   │   │   └── ApiErrorResponse.java
│   │   ├── exception/
│   │   │   └── GlobalExceptionHandler.java       # Global error handling
│   │   ├── factory/
│   │   │   └── WebClientFactoryBean.java         # FactoryBean for WebClient
│   │   ├── runner/
│   │   │   └── DataInitializationRunner.java     # ApplicationRunner for startup data loading
│   │   ├── service/
│   │   │   └── InMemoryDataStore.java            # Thread-safe in-memory data store
│   │   ├── strategy/
│   │   │   ├── IDRDataFetcher.java               # Strategy interface
│   │   │   └── impl/
│   │   │       ├── LatestIdrRatesStrategy.java
│   │   │       ├── HistoricalIdrUsdStrategy.java
│   │   │       └── SupportedCurrenciesStrategy.java
│   │   └── util/
│   │       └── SpreadCalculator.java             # Spread calculation utility
│   └── resources/
│       └── application.yml                        # Application configuration
└── test/
    ├── java/com/allobank/
    │   ├── integration/
    │   │   └── DataInitializationIntegrationTest.java
    │   ├── runner/
    │   │   └── DataInitializationRunnerTest.java
    │   ├── service/
    │   │   └── InMemoryDataStoreTest.java
    │   ├── strategy/impl/
    │   │   ├── LatestIdrRatesStrategyTest.java
    │   │   ├── HistoricalIdrUsdStrategyTest.java
    │   │   └── SupportedCurrenciesStrategyTest.java
    │   └── util/
    │       └── SpreadCalculatorTest.java
    └── resources/
        └── application-test.yml                  # Test configuration
```

## Key Design Decisions

1. **Thread-Safe Data Store:** Uses `ConcurrentHashMap` and `AtomicBoolean` to ensure thread-safe operations and immutability after initialization.

2. **Reactive Programming:** Uses Spring WebFlux's `WebClient` for non-blocking HTTP calls, improving performance and resource utilization.

3. **Error Handling:** Comprehensive error handling at multiple levels - strategy level, controller level, and global exception handler.

4. **Configuration Management:** All external API configuration is externalized to `application.yml` using `@ConfigurationProperties`.

5. **Testing:** Comprehensive unit tests for all strategies and utilities, plus integration tests to verify startup behavior.

## Future Enhancements

Potential improvements for production use:

- Add caching with TTL for data refresh
- Implement retry logic with exponential backoff
- Add circuit breaker pattern for external API calls
- Implement health checks for data availability
- Add metrics and monitoring
- Support for data refresh on-demand via admin endpoint
- Add request/response logging
- Implement rate limiting