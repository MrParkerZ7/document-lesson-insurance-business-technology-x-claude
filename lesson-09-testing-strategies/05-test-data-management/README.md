# 05 - Test Data Management

[Back to Lesson 09](../README.md)

---

## Overview

Effective test data management is crucial for maintainable and reliable tests. This section covers how to create reusable test fixtures and data factories for insurance domain testing.

---

## Test Fixtures

Test fixtures provide pre-built objects for common test scenarios:

```java
public class PolicyFixture {

    public static Policy createActivePolicy() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber("POL-2024-000001")
            .customerId(CustomerId.of("CUST-001"))
            .productCode("MOTOR_COMP")
            .status(PolicyStatus.ACTIVE)
            .effectiveDate(LocalDate.now().minusMonths(6))
            .expiryDate(LocalDate.now().plusMonths(6))
            .sumInsured(new Money("50000", "USD"))
            .totalPremium(new Money("1200", "USD"))
            .build();
    }

    public static Policy createExpiredPolicy() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber("POL-2023-000001")
            .status(PolicyStatus.EXPIRED)
            .effectiveDate(LocalDate.now().minusYears(2))
            .expiryDate(LocalDate.now().minusYears(1))
            .build();
    }

    public static Policy.PolicyBuilder policyBuilder() {
        return Policy.builder()
            .id(PolicyId.generate())
            .policyNumber(generatePolicyNumber())
            .status(PolicyStatus.DRAFT);
    }
}

public class ClaimFixture {

    public static Claim createOpenClaim(Policy policy) {
        return Claim.builder()
            .id(ClaimId.generate())
            .claimNumber("CLM-2024-000001")
            .policyId(policy.getId())
            .customerId(policy.getCustomerId())
            .status(ClaimStatus.OPEN)
            .lossDate(LocalDate.now().minusDays(3))
            .notificationDate(LocalDate.now())
            .claimedAmount(new Money("5000", "USD"))
            .lossDescription("Rear-end collision")
            .build();
    }
}
```

---

## Test Data Factory

Factories generate random but valid test data for more dynamic testing:

```java
@Component
public class TestDataFactory {

    private final Faker faker = new Faker();

    public Customer randomCustomer() {
        return Customer.builder()
            .id(CustomerId.generate())
            .firstName(faker.name().firstName())
            .lastName(faker.name().lastName())
            .email(faker.internet().emailAddress())
            .phone(faker.phoneNumber().phoneNumber())
            .dateOfBirth(randomDateOfBirth(25, 65))
            .address(randomAddress())
            .build();
    }

    public MotorRiskDetails randomMotorRisk() {
        return MotorRiskDetails.builder()
            .vehicleMake(faker.options().option("Toyota", "Honda", "Ford", "BMW"))
            .vehicleModel(faker.options().option("Sedan", "SUV", "Coupe"))
            .vehicleYear(faker.number().numberBetween(2015, 2024))
            .engineCapacity(faker.options().option(1500, 2000, 2500, 3000))
            .vehicleValue(randomMoney(20000, 100000))
            .usageType(UsageType.PRIVATE)
            .build();
    }

    private LocalDate randomDateOfBirth(int minAge, int maxAge) {
        int age = faker.number().numberBetween(minAge, maxAge);
        return LocalDate.now().minusYears(age);
    }

    private Money randomMoney(int min, int max) {
        int amount = faker.number().numberBetween(min, max);
        return new Money(String.valueOf(amount), "USD");
    }
}
```

---

## Test Data Strategies

### Fixtures vs Factories

| Approach | Use Case | Pros | Cons |
|----------|----------|------|------|
| **Fixtures** | Known, specific scenarios | Predictable, easy to debug | Can become stale |
| **Factories** | Random/edge case testing | Discovers edge cases | Harder to reproduce failures |
| **Hybrid** | Most scenarios | Best of both worlds | More code to maintain |

---

## Insurance-Specific Test Data Patterns

### Policy Test Data
```java
// Different policy states
PolicyFixture.createDraftPolicy();
PolicyFixture.createIssuedPolicy();
PolicyFixture.createActivePolicy();
PolicyFixture.createLapsedPolicy();
PolicyFixture.createCancelledPolicy();
PolicyFixture.createExpiredPolicy();
```

### Claim Test Data
```java
// Different claim states
ClaimFixture.createOpenClaim(policy);
ClaimFixture.createAssessedClaim(policy);
ClaimFixture.createApprovedClaim(policy);
ClaimFixture.createSettledClaim(policy);
ClaimFixture.createRejectedClaim(policy);
```

### Customer Test Data
```java
// Different customer types
CustomerFixture.createIndividualCustomer();
CustomerFixture.createCorporateCustomer();
CustomerFixture.createVipCustomer();
CustomerFixture.createHighRiskCustomer();
```

---

## Best Practices

1. **Keep fixtures up-to-date** with domain model changes
2. **Use builders** for flexible test object creation
3. **Avoid magic numbers** - use named constants or enums
4. **Create fixtures for each entity state** needed in tests
5. **Use factories for property-based testing**
6. **Clean test data** between tests to ensure isolation

---

## Navigation

[Previous: End-to-End Testing](../04-end-to-end-testing/README.md) | [Back to Lesson 09](../README.md) | [Next: Testing Best Practices](../06-testing-best-practices/README.md)
