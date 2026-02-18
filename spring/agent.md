
# Agent Instructions for Spring Boot Enterprise Project (with Test Strategy)

## 1. Architecture & Design Pattern (DDD Based)
- **Pattern:** Domain-Driven Design (DDD).
- **Layers:** `interfaces` (API), `application` (Orchestration), `domain` (Logic/Entity), `infrastructure` (Redis/Kafka/External).

## 2. RESTful API & Error Handling
- **API Standards:** Use nouns, kebab-case, versioning (v1), and return `ResponseEntity`.
- **Global Exception:** Use `@RestControllerAdvice`. All error responses must follow the `ErrorResponse` record format with `ErrorCode` enum.

## 3. JPA & Domain Standards
- **Entity:** NO `@Setter`. Use business-logic methods. All FetchType must be `LAZY`.
- **DTO:** Use Java `record`. Separate Request/Response DTOs from Entities.

## 4. Testing Strategy (JUnit 5 Based)
Follow these specific testing patterns for each layer:

### A. Controller (Integration Test)
- **Tool:** `@SpringBootTest` + `@AutoConfigureMockMvc`
- **Scope:** Full stack integration from HTTP request to Database (or Mocked external).
- **Focus:** API Endpoints, Security, Filter, Interceptor, and Correct Response Body/Status.
- **Example:**
  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  class OrderIntegrationTest { ... }


###  B. Service (Mock Test)
- Tool: @ExtendWith(MockitoExtension.class)
- Scope: Pure business logic testing without Spring Context.
- Method: Use @Mock for Dependencies (Repositories, External Clients) and @InjectMocks for the Service under test.

Focus: Logic branching, edge cases, and interaction with dependencies.

### C. Domain (Unit Test)
Tool: Plain JUnit 5 (No Spring, No Mockito if possible).

Scope: Pure POJO testing for Entities and Value Objects.

Focus: Domain logic (e.g., total price calculation, state transition rules).

### D. Repository (H2 Data Test)
Tool: @DataJpaTest

Scope: Persistence layer testing using H2 In-memory Database.

Focus: QueryDSL/JPQL custom queries, Mapping validation, and Persistence constraints.

Config: Use @ActiveProfiles("test") to ensure H2 configuration is used.

### 5. Infrastructure Guidelines
Redis: Use StringRedisTemplate. Ensure TTL is set.

Kafka: Use KafkaTemplate for production. Mock or use @EmbeddedKafka for integration tests.

RestTemplate: Use RestTemplateBuilder with Connection Pooling. Handle errors via ResponseErrorHandler.

### 6. Critical Commands
Full Test: ./gradlew test

Build & Check: ./gradlew clean build
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

# Agent Instructions for Spring Boot Enterprise Project

## 1. Architecture & Design Pattern (DDD Based)
- **Pattern:** Follow Domain-Driven Design (DDD) principles.
- **Packages:** - `domain`: Aggregates, Entities, Repositories, Domain Services (Business Logic).
  - `application`: Application Services (Orchestration, DTO conversion).
  - `interfaces`: Controllers, DTOs (Request/Response), Mappers.
  - `infrastructure`: External API Clients (RestTemplate), Kafka, Redis implementations.

## 2. RESTful API Standards
- **Naming:** Use nouns for resources and kebab-case for URIs (e.g., `/api/v1/order-items`).
- **Methods:** - `GET`: Retrieve resources.
  - `POST`: Create resources.
  - `PUT`: Replace/Update entire resource.
  - `PATCH`: Partial update.
  - `DELETE`: Remove resource.
- **Versioning:** Use URL versioning (e.g., `/api/v1/...`).
- **Response:** Always return `ResponseEntity<T>`.

## 3. JPA Domain Entity Guidelines
- **Encapsulation:** Entities must not have public setters. Use meaningful method names (e.g., `changeAddress()` instead of `setAddress()`).
- **Immutability:** Use Value Objects (Annotated with `@Embeddable`) for grouped attributes.
- **Performance:** - All associations MUST be `FetchType.LAZY`.
  - Use `JoinColumn` for explicit mapping.
- **Audit:** Use `BaseTimeEntity` (MappedSuperclass) for `createdAt` and `updatedAt`.

## 4. DTO & Mapping
- **Records:** Use Java `record` for all DTOs (Request/Response).
- **Separation:** Never expose JPA Entities directly to the Controller. Always map to DTOs in the Application layer.
- **Validation:** Use `jakarta.validation.constraints` (e.g., `@NotBlank`, `@Min`) in Request DTOs.

## 5. Global Error Handling (ErrorAdvice)
- **Mechanism:** Use `@RestControllerAdvice` for global exception handling.
- **Error Response Structure:**
  ```json
  {
    "code": "ORDER_NOT_FOUND",
    "message": "해당 주문을 찾을 수 없습니다.",
    "status": 404,
    "timestamp": "2026-02-18T13:00:00"
  }
  
- Custom Exceptions: Create a BusinessException (RuntimeException) as a base for all domain errors.
- ErrorCode Enum: Define an ErrorCode enum to manage error codes and messages centrally.

6. Technical Components
Redis: Use for caching and session management. TTL must be specified.

Kafka: Produce events in the Service layer after DB commit (Transactional Outbox Pattern preferred).

RestTemplate: - Define as a Bean with Apache HttpClient for connection pooling.

Implement a ResponseErrorHandler to map external errors to internal BusinessException.

7. Security & Constraints
Secrets: Use AWS Parameter Store or Secrets Manager via Spring Cloud AWS.

Testing: Write JUnit 5 tests for Domain logic and @SpringBootTest for API integration.


### A. Controller (Integration Test)
- **Tool:** `@SpringBootTest` + `@AutoConfigureMockMvc`
- **Scope:** Full stack integration from HTTP request to Database (or Mocked external).
- **Focus:** API Endpoints, Security, Filter, Interceptor, and Correct Response Body/Status.
- **Example:**
  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  class OrderIntegrationTest { ... }




>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

## 1. Persona & Role
- You are an expert Java Backend Engineer specializing in Spring Boot 3.x and Java 21.
- Your goal is to write clean, secure, and performant code following modern Java idioms.

## 2. Tech Stack & Architecture
- **Language:** Java 21 (Use records, pattern matching, and virtual threads where applicable).
- **Framework:** Spring Boot 3.4+
- **Build Tool:** Gradle (Kotlin DSL)
- **Architecture:** Layered Architecture (Controller -> Service -> Repository).
- **Database:** PostgreSQL with Spring Data JPA.

## 3. Critical Commands (Run these to verify)
- **Build:** `./gradlew clean build -x test`
- **Unit Test:** `./gradlew test --tests "*TestName*"`
- **Lint/Format:** `./gradlew spotlessApply` (if using Spotless)

## 4. Coding Standards (Always Follow)
- **Dependency Injection:** Use **Constructor Injection** with `@RequiredArgsConstructor`. NEVER use `@Autowired` on fields.
- **Data Handling:** - Use `record` for DTOs and immutable data carriers.
  - Use `Optional<T>` for return types that might be empty, but never as method parameters.
- **Error Handling:** Use `@RestControllerAdvice` and custom `RuntimeExceptions`. Do not return raw Map/String for errors.
- **Naming:** - Interfaces: `OrderService`
  - Implementations: `OrderServiceImpl` (only if multiple impls exist, otherwise stick to the class).
- **Lombok:** Use `@Getter`, `@Setter`, `@Builder`, and `@Slf4j` to reduce boilerplate.


## 5. Boundaries & Constraints
- **Security:** NEVER hardcode secrets or API keys. Use `@Value("${...}")` or `ConfigurationProperties`.
- **Logs:** Use SLF4J/Logback. Do not use `System.out.println()`.
- **Scope:** Do not modify `build.gradle.kts` unless explicitly asked.