# 02.3 Product Configuration

## Overview

Product configuration defines how insurance products are structured in systems, including coverages, pricing rules, and underwriting criteria.

## Product Hierarchy Diagram

![Product Hierarchy](./02.1-product-hierarchy.drawio)

## Hierarchy Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCT HIERARCHY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LINE OF BUSINESS (LOB)                                         │
│     │                                                            │
│     └── PRODUCT                                                  │
│           │                                                      │
│           └── PLAN                                               │
│                 │                                                │
│                 ├── COVERAGE                                     │
│                 │     ├── Benefit                                │
│                 │     ├── Limit                                  │
│                 │     ├── Deductible                             │
│                 │     └── Waiting Period                         │
│                 │                                                │
│                 ├── PREMIUM RULES                                │
│                 │     ├── Base Rate                              │
│                 │     ├── Loading Factors                        │
│                 │     └── Discounts                              │
│                 │                                                │
│                 ├── UNDERWRITING RULES                           │
│                 │     ├── Eligibility Criteria                   │
│                 │     ├── Documentation                          │
│                 │     └── Medical Requirements                   │
│                 │                                                │
│                 └── RIDERS (Optional Add-ons)                    │
│                       ├── Rider 1                                │
│                       ├── Rider 2                                │
│                       └── Rider N                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Example: Motor Insurance Configuration

```yaml
product:
  code: "MOTOR_COMP"
  name: "Comprehensive Motor Insurance"
  lob: "MOTOR"
  status: "ACTIVE"

plans:
  - code: "MOTOR_COMP_BASIC"
    name: "Basic Plan"
    coverages:
      - code: "TPL"
        name: "Third Party Liability"
        mandatory: true
        limit: 1000000

      - code: "OD"
        name: "Own Damage"
        mandatory: true
        deductible: 500

    riders:
      - code: "RSA"
        name: "Roadside Assistance"
        premium: 50

      - code: "NCB"
        name: "No Claim Bonus Protection"
        premium: 75

  - code: "MOTOR_COMP_PREMIUM"
    name: "Premium Plan"
    coverages:
      - code: "TPL"
        limit: 2000000
      - code: "OD"
        deductible: 250
      - code: "PA"
        name: "Personal Accident"
        limit: 500000
```

## Premium Calculation Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    PREMIUM CALCULATION                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   BASE PREMIUM                                                   │
│   (From rate tables based on risk factors)                      │
│        │                                                         │
│        ↓                                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  + LOADINGS (Increase Premium)                          │   │
│   │    - Age loading: +10% (young driver)                   │   │
│   │    - Claims history: +15% (2 claims in 3 years)         │   │
│   │    - High-risk area: +5%                                │   │
│   └─────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  - DISCOUNTS (Reduce Premium)                           │   │
│   │    - No Claims Bonus: -20%                              │   │
│   │    - Multi-policy: -10%                                 │   │
│   │    - Anti-theft device: -5%                             │   │
│   └─────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  + RIDER PREMIUMS                                       │   │
│   │    - Roadside Assistance: $50                           │   │
│   │    - Windscreen Cover: $30                              │   │
│   └─────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  + TAXES & FEES                                         │   │
│   │    - Insurance Tax: 10%                                 │   │
│   │    - Policy Fee: $25                                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│   ═══════════════════════════════════════════════════════════   │
│   TOTAL PREMIUM                                                  │
│   ═══════════════════════════════════════════════════════════   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Key Configuration Elements

### Coverage Configuration
| Element | Description | Example |
|---------|-------------|---------|
| **Limit** | Maximum payout | $1,000,000 |
| **Deductible** | Customer pays first | $500 per claim |
| **Co-pay** | Customer percentage | 20% of bill |
| **Waiting Period** | Coverage delay | 30 days |

### Premium Rules
| Rule Type | Description | Example |
|-----------|-------------|---------|
| **Base Rate** | Starting premium | 2% of vehicle value |
| **Loading** | Risk multiplier | +10% for young drivers |
| **Discount** | Reduction | -20% for no claims |
| **Minimum Premium** | Floor amount | $100 minimum |

## System Implications

- Product catalog management system
- Rule engine for pricing
- Configurability without code changes
- Version control for products
- Effective dating for changes

---

[← Previous: Non-Life Insurance](../02-non-life-insurance/README.md) | [Next Lesson: Core Processes →](../../lesson-03-core-processes/README.md)
