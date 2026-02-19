# 8.6 Code Quality

## Diagram

![Code Quality](./08.6-code-quality.drawio.png)

> Source: [08.6-code-quality.drawio](./08.6-code-quality.drawio)

---

## Overview

High-quality code is maintainable, readable, and reliable. This sub-lesson covers clean code principles that lead to sustainable software development in insurance systems.

---

## Clean Code Checklist

```
+------------------------------------------------------------------+
|                    CLEAN CODE CHECKLIST                          |
+------------------------------------------------------------------+
|                                                                  |
|  NAMING:                                                         |
|  [ ] Names reveal intent                                         |
|  [ ] Avoid abbreviations (except well-known: id, url)           |
|  [ ] Use consistent vocabulary                                   |
|  [ ] Class names are nouns, method names are verbs              |
|                                                                  |
|  FUNCTIONS:                                                      |
|  [ ] Small (< 20 lines preferred)                                |
|  [ ] Do one thing                                                |
|  [ ] Few parameters (3 or fewer)                                 |
|  [ ] No side effects                                             |
|                                                                  |
|  COMMENTS:                                                       |
|  [ ] Code is self-documenting                                    |
|  [ ] Comments explain WHY, not WHAT                              |
|  [ ] No commented-out code                                       |
|  [ ] Javadoc for public APIs                                     |
|                                                                  |
|  ERROR HANDLING:                                                 |
|  [ ] Use exceptions, not error codes                             |
|  [ ] Provide context in exceptions                               |
|  [ ] Don't return null                                           |
|  [ ] Fail fast                                                   |
|                                                                  |
|  TESTING:                                                        |
|  [ ] One assert per test                                         |
|  [ ] Test names describe behavior                                |
|  [ ] Tests are independent                                       |
|  [ ] Fast and repeatable                                         |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Naming Principles

### Good Names

```java
// Reveals intent
int daysSinceLastPayment;
List<Policy> expiredPolicies;
boolean isEligibleForRenewal;

// Bad: unclear intent
int d;
List<Policy> list1;
boolean flag;
```

### Consistency

```java
// Good: consistent vocabulary
fetchCustomer();
fetchPolicy();
fetchClaim();

// Bad: inconsistent vocabulary
getCustomer();
retrievePolicy();
fetchClaim();
```

---

## Function Design

### Single Responsibility

```java
// Good: does one thing
public BigDecimal calculateBasePremium(Policy policy) {
    return policy.getSumInsured()
        .multiply(getRateForProduct(policy.getProductCode()));
}

// Bad: does multiple things
public BigDecimal calculatePremiumAndSendNotification(Policy policy) {
    BigDecimal premium = calculatePremium(policy);
    emailService.sendQuote(policy.getCustomer(), premium);
    return premium;
}
```

### Few Parameters

```java
// Good: object parameter
public Quote createQuote(QuoteRequest request) { }

// Bad: too many parameters
public Quote createQuote(String customerId, String productCode,
    BigDecimal sumInsured, LocalDate effectiveDate,
    List<Coverage> coverages, Map<String, Object> riskDetails) { }
```

---

## Comment Guidelines

### Good Comments

```java
// Regulatory requirement: retention period mandated by Insurance Act 2020
private static final int DOCUMENT_RETENTION_YEARS = 7;

/**
 * Calculates the pro-rata refund for mid-term cancellation.
 * Uses the "short-rate" method as per company policy.
 */
public BigDecimal calculateRefund(Policy policy, LocalDate cancellationDate) { }
```

### Bad Comments

```java
// Bad: states the obvious
// Increment counter
counter++;

// Bad: commented-out code
// if (customer.getAge() > 65) {
//     applyDiscount();
// }
```

---

## Error Handling Principles

### Fail Fast

```java
// Good: validate early
public void processPayment(Payment payment) {
    Objects.requireNonNull(payment, "Payment cannot be null");
    if (payment.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
        throw new InvalidPaymentAmountException(payment.getAmount());
    }
    // Process payment...
}
```

### Avoid Null Returns

```java
// Good: use Optional
public Optional<Policy> findByPolicyNumber(String policyNumber) {
    return repository.findByPolicyNumber(policyNumber);
}

// Bad: returns null
public Policy findByPolicyNumber(String policyNumber) {
    return repository.findByPolicyNumber(policyNumber); // might return null
}
```

---

## Testing Best Practices

### Descriptive Test Names

```java
// Good: describes behavior
@Test
void shouldRejectQuoteWhenCustomerAgeExceedsMaximum() { }

@Test
void shouldApplyNoClaimsDiscountWhenHistoryIsClean() { }

// Bad: vague names
@Test
void testQuote() { }

@Test
void test1() { }
```

### Independent Tests

```java
// Good: each test sets up its own state
@Test
void shouldCalculatePremiumForMotorPolicy() {
    Policy policy = PolicyFixture.createMotorPolicy();
    BigDecimal premium = calculator.calculate(policy);
    assertThat(premium).isEqualTo(new BigDecimal("1500.00"));
}
```

---

## Code Metrics

| Metric | Target | Tool |
|--------|--------|------|
| Cyclomatic Complexity | < 10 per method | SonarQube |
| Method Length | < 20 lines | Checkstyle |
| Class Length | < 300 lines | Checkstyle |
| Test Coverage | > 80% | JaCoCo |
| Code Duplication | < 3% | SonarQube |

---

## Navigation

[Previous: 8.5 Logging Standards](../05-logging-standards/README.md) | [Back to Lesson 08](../README.md)
