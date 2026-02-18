# 8.2 Design Patterns

## Overview

Design patterns provide proven solutions to common software design problems. In insurance systems, specific patterns help manage complex business rules, state transitions, and data access requirements.

---

## Repository Pattern

The Repository pattern abstracts data access, allowing the domain layer to remain independent of persistence mechanisms.

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

---

## Factory Pattern

The Factory pattern encapsulates complex object creation logic, ensuring consistent initialization.

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

---

## Strategy Pattern

The Strategy pattern enables selecting algorithms at runtime, useful for product-specific calculations.

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

---

## State Pattern

The State pattern manages complex state transitions in insurance entities like policies.

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

---

## Specification Pattern

The Specification pattern encapsulates business rules for eligibility and validation.

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

## Pattern Summary

| Pattern | Use Case | Insurance Example |
|---------|----------|-------------------|
| **Repository** | Data access abstraction | Policy, Claim data access |
| **Factory** | Complex object creation | Quote creation with premium calculation |
| **Strategy** | Algorithm selection | Product-specific premium calculation |
| **State** | State machine management | Policy lifecycle transitions |
| **Specification** | Business rule validation | Eligibility and underwriting rules |

---

## Navigation

[Previous: 8.1 Code Organization](../01-code-organization/README.md) | [Back to Lesson 08](../README.md) | [Next: 8.3 Naming Conventions](../03-naming-conventions/README.md)
