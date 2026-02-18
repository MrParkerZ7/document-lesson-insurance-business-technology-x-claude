# 01 - Entity Relationships

## Overview

This section covers the core entity relationships that form the foundation of insurance data models. Understanding these relationships is crucial for designing scalable and maintainable insurance systems.

---

## Entity Relationship Diagram

```
+-----------------------------------------------------------------------------+
|                       ENTITY RELATIONSHIP DIAGRAM                            |
+-----------------------------------------------------------------------------+
|                                                                              |
|  +--------------+         +--------------+         +--------------+          |
|  |   CUSTOMER   | 1----M  |    POLICY    | M----1  |   PRODUCT    |          |
|  +--------------+         +--------------+         +--------------+          |
|        |                        |                        |                   |
|        | 1                      | 1                      | 1                 |
|        |                        |                        |                   |
|        M                        M                        M                   |
|  +--------------+         +--------------+         +--------------+          |
|  |   ADDRESS    |         |    CLAIM     |         |   COVERAGE   |          |
|  +--------------+         +--------------+         +--------------+          |
|                                 |                        |                   |
|                                 | 1                      | 1                 |
|                                 |                        |                   |
|                                 M                        M                   |
|                           +--------------+         +--------------+          |
|                           |CLAIM_PAYMENT |         |    RIDER     |          |
|                           +--------------+         +--------------+          |
|                                                                              |
|  +--------------+         +--------------+         +--------------+          |
|  |    AGENT     | 1----M  |    QUOTE     | M----1  |   CUSTOMER   |          |
|  +--------------+         +--------------+         +--------------+          |
|                                 |                                            |
|                                 | 1                                          |
|                                 |                                            |
|                                 M                                            |
|                           +--------------+                                   |
|                           |  QUOTE_ITEM  |                                   |
|                           +--------------+                                   |
|                                                                              |
+-----------------------------------------------------------------------------+
```

> **Note:** A detailed visual diagram is available in the file `05.1-entity-relationship-diagram.drawio` in this folder.

---

## Relationship Matrix

| Entity A | Relationship | Entity B | Description |
|----------|--------------|----------|-------------|
| Customer | 1:M | Policy | Customer has many policies |
| Customer | 1:M | Address | Customer has multiple addresses |
| Customer | 1:M | Quote | Customer has many quotes |
| Policy | M:1 | Product | Policy belongs to product |
| Policy | 1:M | Claim | Policy has many claims |
| Policy | 1:M | Endorsement | Policy has endorsement history |
| Policy | 1:M | Premium | Policy has premium schedule |
| Claim | 1:M | ClaimPayment | Claim has payment history |
| Product | 1:M | Coverage | Product has coverages |
| Product | 1:M | Rider | Product has optional riders |
| Agent | 1:M | Policy | Agent sells policies |
| Quote | 1:M | QuoteItem | Quote has line items |

---

## Core Entities Description

### Customer
The policyholder or insured party. Can be an individual or corporate entity.

**Key Attributes:**
- Customer ID (unique identifier)
- Customer Type (INDIVIDUAL, CORPORATE)
- Contact Information
- KYC Status
- Risk Assessment

### Policy
The insurance contract between the insurer and the customer.

**Key Attributes:**
- Policy Number
- Product and Plan
- Effective/Expiry Dates
- Sum Insured
- Premium Details
- Status

### Product
The insurance product template that defines coverages and terms.

**Key Attributes:**
- Product Code
- Product Type (LIFE, HEALTH, MOTOR, PROPERTY, LIABILITY)
- Eligibility Criteria
- Policy Terms
- Commission Rates

### Claim
A request for payment under the terms of the policy.

**Key Attributes:**
- Claim Number
- Loss Details
- Claimed/Approved Amounts
- Status
- Adjuster Assignment

---

## Relationship Types

### One-to-Many (1:M)
Most common relationship in insurance data models.
- One Customer can have Many Policies
- One Policy can have Many Claims
- One Claim can have Many Payments

### Many-to-One (M:1)
Child entities referencing parent entities.
- Many Policies belong to One Product
- Many Claims belong to One Policy

### Self-Referencing
Used for hierarchical or versioned data.
- Policy can reference Previous Policy (for renewals)
- Policy can reference Parent Policy (for versions)

---

## Navigation

[Back to Lesson 05](../README.md) | [Next: Database Schemas](../02-database-schemas/README.md)
