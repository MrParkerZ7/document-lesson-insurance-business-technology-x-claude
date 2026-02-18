# 8.3 Naming Conventions

## Overview

Consistent naming conventions make code self-documenting and reduce cognitive load. This sub-lesson covers naming standards for both code elements and database objects in insurance applications.

---

## Code Naming Standards

### Classes and Interfaces

```java
// Classes: PascalCase, nouns
public class PolicyService { }
public class ClaimAssessmentWorkflow { }
public class QuoteExpirationScheduler { }

// Interfaces: PascalCase, adjectives or nouns
public interface Cancellable { }
public interface PolicyRepository { }
public interface PremiumCalculationStrategy { }
```

### Methods

```java
// Methods: camelCase, verbs
public void calculatePremium() { }
public boolean validateCoverage() { }
public Quote createQuoteFromProposal() { }
```

### Variables

```java
// Variables: camelCase
private String policyNumber;
private BigDecimal totalPremium;
private List<Coverage> activeCoverages;
```

### Constants

```java
// Constants: UPPER_SNAKE_CASE
public static final int MAX_POLICY_TERM_MONTHS = 120;
public static final BigDecimal DEFAULT_TAX_RATE = new BigDecimal("0.10");
public static final String POLICY_NUMBER_PREFIX = "POL";
```

### Enums

```java
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
```

### Packages

```java
// Packages: lowercase
package com.insurance.policy.domain.services;
package com.insurance.claims.api.controllers;
```

---

## Database Naming Standards

### Tables

```sql
-- Tables: snake_case, singular
CREATE TABLE policy (...)
CREATE TABLE claim_payment (...)
CREATE TABLE policy_endorsement (...)
```

### Columns

```sql
-- Columns: snake_case
customer_id
effective_date
total_premium
```

### Primary Keys

```sql
-- Primary Keys: {table}_id
policy_id
claim_id
customer_id
```

### Foreign Keys

```sql
-- Foreign Keys: {referenced_table}_id
customer_id REFERENCES customer(customer_id)
product_id REFERENCES product(product_id)
```

### Indexes

```sql
-- Indexes: idx_{table}_{columns}
CREATE INDEX idx_policy_customer ON policy(customer_id);
CREATE INDEX idx_claim_status_date ON claim(status, loss_date);
```

### Constraints

```sql
-- Constraints: chk_{table}_{description}
CONSTRAINT chk_policy_status CHECK (status IN ('ACTIVE', 'LAPSED'))
CONSTRAINT chk_claim_amount_positive CHECK (claimed_amount > 0)
```

---

## Naming Convention Summary

| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase, nouns | `PolicyService` |
| Interfaces | PascalCase | `Cancellable`, `PolicyRepository` |
| Methods | camelCase, verbs | `calculatePremium()` |
| Variables | camelCase | `policyNumber` |
| Constants | UPPER_SNAKE_CASE | `MAX_POLICY_TERM` |
| Enum Values | UPPER_SNAKE_CASE | `ACTIVE`, `CANCELLED` |
| Packages | lowercase | `com.insurance.policy` |
| Tables | snake_case, singular | `policy`, `claim_payment` |
| Columns | snake_case | `effective_date` |
| Indexes | idx_{table}_{columns} | `idx_policy_customer` |

---

## Best Practices

1. **Be Descriptive**: Use names that clearly convey purpose
2. **Avoid Abbreviations**: Except for well-known ones (id, url, dto)
3. **Consistent Vocabulary**: Use the same terms throughout the codebase
4. **Domain Language**: Use insurance domain terminology
5. **Avoid Magic Numbers**: Replace with named constants

---

## Navigation

[Previous: 8.2 Design Patterns](../02-design-patterns/README.md) | [Back to Lesson 08](../README.md) | [Next: 8.4 Error Handling](../04-error-handling/README.md)
