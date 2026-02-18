# Lesson 08: Development Best Practices

## Learning Objectives

By the end of this lesson, you will be able to:
- Apply design patterns for insurance systems
- Implement proper code organization and structure
- Follow naming conventions and coding standards
- Handle errors and exceptions effectively

---

## 8.1 Code Organization

### Project Structure

```
insurance-platform/
├── services/
│   ├── quote-service/
│   │   ├── src/
│   │   │   ├── api/
│   │   │   │   ├── controllers/
│   │   │   │   │   ├── QuoteController.java
│   │   │   │   │   └── QuoteComparisonController.java
│   │   │   │   ├── dto/
│   │   │   │   │   ├── request/
│   │   │   │   │   │   └── CreateQuoteRequest.java
│   │   │   │   │   └── response/
│   │   │   │   │       └── QuoteResponse.java
│   │   │   │   ├── mappers/
│   │   │   │   │   └── QuoteMapper.java
│   │   │   │   └── validators/
│   │   │   │       └── QuoteRequestValidator.java
│   │   │   │
│   │   │   ├── domain/
│   │   │   │   ├── models/
│   │   │   │   │   ├── Quote.java
│   │   │   │   │   ├── QuoteItem.java
│   │   │   │   │   └── PremiumBreakdown.java
│   │   │   │   ├── services/
│   │   │   │   │   ├── QuoteService.java
│   │   │   │   │   ├── PremiumCalculationService.java
│   │   │   │   │   └── QuoteConversionService.java
│   │   │   │   ├── repositories/
│   │   │   │   │   └── QuoteRepository.java
│   │   │   │   └── events/
│   │   │   │       ├── QuoteCreatedEvent.java
│   │   │   │       └── QuoteExpiredEvent.java
│   │   │   │
│   │   │   ├── infrastructure/
│   │   │   │   ├── persistence/
│   │   │   │   │   ├── entities/
│   │   │   │   │   │   └── QuoteEntity.java
│   │   │   │   │   └── repositories/
│   │   │   │   │       └── JpaQuoteRepository.java
│   │   │   │   ├── messaging/
│   │   │   │   │   ├── publishers/
│   │   │   │   │   │   └── QuoteEventPublisher.java
│   │   │   │   │   └── consumers/
│   │   │   │   │       └── ProductUpdateConsumer.java
│   │   │   │   └── external/
│   │   │   │       ├── ProductServiceClient.java
│   │   │   │       └── CustomerServiceClient.java
│   │   │   │
│   │   │   └── config/
│   │   │       ├── ApplicationConfig.java
│   │   │       ├── SecurityConfig.java
│   │   │       └── KafkaConfig.java
│   │   │
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   ├── integration/
│   │   │   └── e2e/
│   │   │
│   │   ├── resources/
│   │   │   ├── application.yml
│   │   │   └── db/migration/
│   │   │
│   │   ├── Dockerfile
│   │   └── pom.xml
│   │
│   ├── policy-service/
│   ├── claims-service/
│   └── customer-service/
│
├── shared/
│   ├── common-lib/
│   │   ├── exceptions/
│   │   ├── utils/
│   │   └── constants/
│   ├── event-schemas/
│   └── api-contracts/
│
├── infrastructure/
│   ├── terraform/
│   ├── kubernetes/
│   └── scripts/
│
└── docs/
    ├── api/
    ├── architecture/
    └── runbooks/
```

---

## 8.2 Design Patterns

### Repository Pattern

```java
// Domain Repository Interface
public interface PolicyRepository {
    Optional<Policy> findById(PolicyId id);
    Optional<Policy> findByPolicyNumber(String policyNumber);
    List<Policy> findByCustomerId(CustomerId customerId);
    List<Policy> findByStatus(PolicyStatus status);
    Policy save(Policy policy);
    void delete(PolicyId id);
}

// Infrastructure Implementation
@Repository
public class JpaPolicyRepository implements PolicyRepository {

    private final PolicyJpaRepository jpaRepository;
    private final PolicyMapper mapper;

    @Override
    public Optional<Policy> findById(PolicyId id) {
        return jpaRepository.findById(id.getValue())
            .map(mapper::toDomain);
    }

    @Override
    public Policy save(Policy policy) {
        PolicyEntity entity = mapper.toEntity(policy);
        PolicyEntity saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }
}
```

### Factory Pattern

```java
// Quote Factory
public class QuoteFactory {

    private final ProductService productService;
    private final RatingEngine ratingEngine;

    public Quote createQuote(CreateQuoteRequest request) {
        // Validate product exists and is active
        Product product = productService.getProduct(request.getProductCode());

        // Build quote with calculated premium
        Quote quote = Quote.builder()
            .quoteId(QuoteId.generate())
            .quoteNumber(generateQuoteNumber())
            .productCode(product.getCode())
            .customerId(request.getCustomerId())
            .riskDetails(mapRiskDetails(request))
            .status(QuoteStatus.DRAFT)
            .createdAt(Instant.now())
            .validUntil(calculateValidUntil())
            .build();

        // Calculate premium
        PremiumBreakdown premium = ratingEngine.calculatePremium(
            product,
            quote.getRiskDetails()
        );
        quote.setPremiumBreakdown(premium);

        return quote;
    }
}
```

### Strategy Pattern

```java
// Premium Calculation Strategy Interface
public interface PremiumCalculationStrategy {
    PremiumBreakdown calculate(Product product, RiskDetails riskDetails);
    boolean supports(ProductType productType);
}

// Motor Insurance Strategy
@Component
public class MotorPremiumStrategy implements PremiumCalculationStrategy {

    @Override
    public PremiumBreakdown calculate(Product product, RiskDetails riskDetails) {
        MotorRiskDetails motorRisk = (MotorRiskDetails) riskDetails;

        BigDecimal basePremium = calculateBasePremium(product, motorRisk);
        BigDecimal loadings = calculateLoadings(motorRisk);
        BigDecimal discounts = calculateDiscounts(motorRisk);

        return PremiumBreakdown.builder()
            .basePremium(basePremium)
            .loadings(loadings)
            .discounts(discounts)
            .netPremium(basePremium.add(loadings).subtract(discounts))
            .build();
    }

    @Override
    public boolean supports(ProductType productType) {
        return productType == ProductType.MOTOR;
    }
}

// Strategy Selector
@Component
public class PremiumCalculationStrategySelector {

    private final List<PremiumCalculationStrategy> strategies;

    public PremiumCalculationStrategy selectStrategy(ProductType productType) {
        return strategies.stream()
            .filter(s -> s.supports(productType))
            .findFirst()
            .orElseThrow(() -> new UnsupportedProductException(productType));
    }
}
```

### State Pattern

```java
// Policy State Interface
public interface PolicyState {
    PolicyState activate(Policy policy);
    PolicyState cancel(Policy policy, CancellationReason reason);
    PolicyState lapse(Policy policy);
    PolicyState reinstate(Policy policy);
    PolicyState expire(Policy policy);
}

// Active State
public class ActivePolicyState implements PolicyState {

    @Override
    public PolicyState cancel(Policy policy, CancellationReason reason) {
        policy.setCancellationDate(LocalDate.now());
        policy.setCancellationReason(reason);
        policy.calculateRefund();
        return new CancelledPolicyState();
    }

    @Override
    public PolicyState lapse(Policy policy) {
        if (policy.isInGracePeriod()) {
            throw new PolicyInGracePeriodException(policy.getId());
        }
        policy.setLapseDate(LocalDate.now());
        return new LapsedPolicyState();
    }

    @Override
    public PolicyState expire(Policy policy) {
        if (LocalDate.now().isBefore(policy.getExpiryDate())) {
            throw new PolicyNotExpiredException(policy.getId());
        }
        return new ExpiredPolicyState();
    }

    // Other transitions throw IllegalStateTransitionException
}
```

### Specification Pattern

```java
// Eligibility Specification
public interface EligibilitySpecification {
    boolean isSatisfiedBy(Customer customer, Product product);
    String getViolationMessage();
}

// Age Eligibility
public class AgeEligibilitySpecification implements EligibilitySpecification {

    @Override
    public boolean isSatisfiedBy(Customer customer, Product product) {
        int age = customer.getAge();
        return age >= product.getMinEntryAge()
            && age <= product.getMaxEntryAge();
    }

    @Override
    public String getViolationMessage() {
        return "Customer age is outside eligible range";
    }
}

// Composite Specification
public class CompositeEligibilitySpecification implements EligibilitySpecification {

    private final List<EligibilitySpecification> specifications;

    @Override
    public boolean isSatisfiedBy(Customer customer, Product product) {
        return specifications.stream()
            .allMatch(spec -> spec.isSatisfiedBy(customer, product));
    }

    public List<String> getViolations(Customer customer, Product product) {
        return specifications.stream()
            .filter(spec -> !spec.isSatisfiedBy(customer, product))
            .map(EligibilitySpecification::getViolationMessage)
            .collect(Collectors.toList());
    }
}
```

---

## 8.3 Naming Conventions

### Code Naming Standards

```java
// Classes: PascalCase, nouns
public class PolicyService { }
public class ClaimAssessmentWorkflow { }
public class QuoteExpirationScheduler { }

// Interfaces: PascalCase, adjectives or nouns
public interface Cancellable { }
public interface PolicyRepository { }
public interface PremiumCalculationStrategy { }

// Methods: camelCase, verbs
public void calculatePremium() { }
public boolean validateCoverage() { }
public Quote createQuoteFromProposal() { }

// Variables: camelCase
private String policyNumber;
private BigDecimal totalPremium;
private List<Coverage> activeCoverages;

// Constants: UPPER_SNAKE_CASE
public static final int MAX_POLICY_TERM_MONTHS = 120;
public static final BigDecimal DEFAULT_TAX_RATE = new BigDecimal("0.10");
public static final String POLICY_NUMBER_PREFIX = "POL";

// Enums: PascalCase class, UPPER_SNAKE_CASE values
public enum PolicyStatus {
    DRAFT,
    QUOTED,
    PROPOSED,
    ISSUED,
    ACTIVE,
    LAPSED,
    CANCELLED,
    EXPIRED
}

// Packages: lowercase
package com.insurance.policy.domain.services;
package com.insurance.claims.api.controllers;
```

### Database Naming Standards

```sql
-- Tables: snake_case, singular
CREATE TABLE policy (...)
CREATE TABLE claim_payment (...)
CREATE TABLE policy_endorsement (...)

-- Columns: snake_case
customer_id
effective_date
total_premium

-- Primary Keys: {table}_id
policy_id
claim_id
customer_id

-- Foreign Keys: {referenced_table}_id
customer_id REFERENCES customer(customer_id)
product_id REFERENCES product(product_id)

-- Indexes: idx_{table}_{columns}
CREATE INDEX idx_policy_customer ON policy(customer_id);
CREATE INDEX idx_claim_status_date ON claim(status, loss_date);

-- Constraints: chk_{table}_{description}
CONSTRAINT chk_policy_status CHECK (status IN ('ACTIVE', 'LAPSED'))
CONSTRAINT chk_claim_amount_positive CHECK (claimed_amount > 0)
```

---

## 8.4 Error Handling

### Exception Hierarchy

```java
// Base Exception
public abstract class InsuranceException extends RuntimeException {
    private final String errorCode;
    private final Map<String, Object> context;

    protected InsuranceException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }

    public InsuranceException withContext(String key, Object value) {
        this.context.put(key, value);
        return this;
    }
}

// Business Exceptions (4xx)
public class ValidationException extends InsuranceException {
    private final List<FieldError> fieldErrors;

    public ValidationException(List<FieldError> errors) {
        super("VALIDATION_ERROR", "Request validation failed");
        this.fieldErrors = errors;
    }
}

public class PolicyNotFoundException extends InsuranceException {
    public PolicyNotFoundException(String policyId) {
        super("POLICY_NOT_FOUND", "Policy not found: " + policyId);
        withContext("policyId", policyId);
    }
}

public class InvalidStateTransitionException extends InsuranceException {
    public InvalidStateTransitionException(String currentState, String targetState) {
        super("INVALID_STATE_TRANSITION",
            String.format("Cannot transition from %s to %s", currentState, targetState));
        withContext("currentState", currentState);
        withContext("targetState", targetState);
    }
}

// Technical Exceptions (5xx)
public class ExternalServiceException extends InsuranceException {
    public ExternalServiceException(String service, String message) {
        super("EXTERNAL_SERVICE_ERROR", message);
        withContext("service", service);
    }
}
```

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.warn("Validation failed: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .details(ex.getFieldErrors())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(PolicyNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(PolicyNotFoundException ex) {
        log.info("Resource not found: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }

    @ExceptionHandler(InvalidStateTransitionException.class)
    public ResponseEntity<ErrorResponse> handleStateTransition(InvalidStateTransitionException ex) {
        log.warn("Invalid state transition: {}", ex.getMessage());

        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getErrorCode())
            .message(ex.getMessage())
            .context(ex.getContext())
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);

        ErrorResponse response = ErrorResponse.builder()
            .code("INTERNAL_ERROR")
            .message("An unexpected error occurred")
            .timestamp(Instant.now())
            .build();

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

### Error Response Structure

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "effectiveDate",
        "code": "INVALID_DATE",
        "message": "Effective date must be in the future"
      },
      {
        "field": "sumInsured",
        "code": "EXCEEDS_LIMIT",
        "message": "Sum insured exceeds product maximum of $1,000,000"
      }
    ],
    "correlation_id": "corr_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

---

## 8.5 Logging Standards

### Log Levels

| Level | Usage | Example |
|-------|-------|---------|
| **ERROR** | System errors requiring attention | Database connection failed |
| **WARN** | Potential issues, degraded state | External service timeout, retrying |
| **INFO** | Business events, milestones | Policy issued, claim settled |
| **DEBUG** | Detailed technical information | Request/response payloads |
| **TRACE** | Very detailed debugging | Method entry/exit |

### Structured Logging

```java
@Service
@Slf4j
public class PolicyService {

    public Policy issuePolicy(IssuePolicyRequest request) {
        log.info("Issuing policy",
            kv("customerId", request.getCustomerId()),
            kv("productCode", request.getProductCode()),
            kv("sumInsured", request.getSumInsured()));

        try {
            Policy policy = createPolicy(request);

            log.info("Policy issued successfully",
                kv("policyId", policy.getId()),
                kv("policyNumber", policy.getPolicyNumber()),
                kv("premium", policy.getTotalPremium()));

            return policy;

        } catch (UnderwritingRejectedException e) {
            log.warn("Policy issuance rejected by underwriting",
                kv("customerId", request.getCustomerId()),
                kv("reason", e.getReason()));
            throw e;

        } catch (Exception e) {
            log.error("Failed to issue policy",
                kv("customerId", request.getCustomerId()),
                kv("error", e.getMessage()),
                e);
            throw e;
        }
    }
}
```

---

## 8.6 Code Quality

### Clean Code Principles

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLEAN CODE CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NAMING:                                                         │
│  □ Names reveal intent                                           │
│  □ Avoid abbreviations (except well-known: id, url)             │
│  □ Use consistent vocabulary                                     │
│  □ Class names are nouns, method names are verbs                │
│                                                                  │
│  FUNCTIONS:                                                      │
│  □ Small (< 20 lines preferred)                                  │
│  □ Do one thing                                                  │
│  □ Few parameters (3 or fewer)                                   │
│  □ No side effects                                               │
│                                                                  │
│  COMMENTS:                                                       │
│  □ Code is self-documenting                                      │
│  □ Comments explain WHY, not WHAT                                │
│  □ No commented-out code                                         │
│  □ Javadoc for public APIs                                       │
│                                                                  │
│  ERROR HANDLING:                                                 │
│  □ Use exceptions, not error codes                               │
│  □ Provide context in exceptions                                 │
│  □ Don't return null                                             │
│  □ Fail fast                                                     │
│                                                                  │
│  TESTING:                                                        │
│  □ One assert per test                                           │
│  □ Test names describe behavior                                  │
│  □ Tests are independent                                         │
│  □ Fast and repeatable                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Consistent code organization** improves maintainability
2. **Design patterns** solve common problems elegantly
3. **Naming conventions** make code self-documenting
4. **Proper error handling** enables debugging and user feedback
5. **Structured logging** enables effective troubleshooting

---

## Exercises

1. Implement Observer pattern for policy status changes
2. Create a specification for claim eligibility validation
3. Design error handling for payment processing failures

---

[← Previous Lesson](../07-security-compliance/README.md) | [Next Lesson: Testing Strategies →](../09-testing-strategies/README.md)
