# 06 - Testing Best Practices

[Back to Lesson 09](../README.md)

---

## Overview

This section covers insurance-specific test scenarios and coverage guidelines to ensure comprehensive testing of insurance systems.

---

## Insurance-Specific Test Scenarios

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INSURANCE TEST SCENARIOS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  QUOTE SERVICE:                                                              │
│  ├── Premium calculation accuracy                                            │
│  ├── Discount/loading application order                                      │
│  ├── Tax calculation by region                                               │
│  ├── Quote expiration handling                                               │
│  └── Product eligibility validation                                          │
│                                                                              │
│  POLICY SERVICE:                                                             │
│  ├── Policy status transitions (state machine)                               │
│  ├── Endorsement premium adjustments                                         │
│  ├── Pro-rata refund calculations                                            │
│  ├── Renewal premium recalculation                                           │
│  ├── Grace period handling                                                   │
│  └── Reinstatement eligibility                                               │
│                                                                              │
│  CLAIMS SERVICE:                                                             │
│  ├── Coverage verification                                                   │
│  ├── Deductible application                                                  │
│  ├── Claim limit checking (per occurrence, aggregate)                        │
│  ├── Waiting period validation                                               │
│  ├── Pre-existing condition exclusion                                        │
│  └── Fraud indicator detection                                               │
│                                                                              │
│  BILLING SERVICE:                                                            │
│  ├── Payment schedule generation                                             │
│  ├── Commission calculation                                                  │
│  ├── Clawback processing                                                     │
│  ├── Failed payment retry logic                                              │
│  └── Refund processing                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Test Coverage Guidelines

| Component | Min Coverage | Focus Areas |
|-----------|-------------|-------------|
| **Domain Services** | 90% | Business logic, calculations |
| **Controllers** | 80% | Request validation, response mapping |
| **Repositories** | 70% | CRUD operations, queries |
| **Event Handlers** | 85% | Event processing, state changes |
| **External Clients** | 75% | Error handling, retries |

---

## Testing Checklist by Service

### Quote Service
- [ ] Premium calculation with all rating factors
- [ ] Multiple discount combinations
- [ ] Loading factor applications
- [ ] Tax calculations for different regions
- [ ] Quote validity period enforcement
- [ ] Product eligibility rules

### Policy Service
- [ ] All valid state transitions
- [ ] Invalid state transition rejections
- [ ] Endorsement creation and application
- [ ] Cancellation with refund calculation
- [ ] Renewal processing
- [ ] Grace period enforcement
- [ ] Reinstatement rules

### Claims Service
- [ ] FNOL registration
- [ ] Coverage verification against policy
- [ ] Deductible calculations
- [ ] Per-occurrence limit checks
- [ ] Aggregate limit tracking
- [ ] Waiting period validation
- [ ] Exclusion rule application
- [ ] Fraud flag triggers

### Billing Service
- [ ] Payment schedule generation
- [ ] Installment calculations
- [ ] Commission calculations
- [ ] Failed payment handling
- [ ] Refund processing
- [ ] Clawback scenarios

---

## Common Testing Patterns

### Testing Calculations
```java
@ParameterizedTest
@CsvSource({
    "10000, 0.025, 250",
    "50000, 0.025, 1250",
    "100000, 0.025, 2500"
})
void shouldCalculatePremiumCorrectly(int sumInsured, String rate, int expected) {
    // Test calculation logic
}
```

### Testing State Transitions
```java
@Test
void shouldFollowValidStateTransition() {
    // Given initial state
    // When transition triggered
    // Then new state is valid
}

@Test
void shouldRejectInvalidStateTransition() {
    // Given invalid initial state
    // When transition triggered
    // Then exception is thrown
}
```

### Testing Error Handling
```java
@Test
void shouldHandleExternalServiceFailure() {
    // Given external service returns error
    // When operation is attempted
    // Then graceful degradation occurs
}
```

---

## Test Quality Metrics

- **Line Coverage**: Minimum 80% for critical services
- **Branch Coverage**: Test all decision paths
- **Mutation Testing**: Aim for 70%+ mutation score
- **Test Execution Time**: Unit tests < 10ms, Integration < 5s

---

## Anti-Patterns to Avoid

1. **Testing implementation details** instead of behavior
2. **Overly broad tests** that test multiple things
3. **Flaky tests** with non-deterministic behavior
4. **Test data coupling** between tests
5. **Ignoring edge cases** in business logic
6. **Skipping negative test cases**

---

## Navigation

[Previous: Test Data Management](../05-test-data-management/README.md) | [Back to Lesson 09](../README.md)
