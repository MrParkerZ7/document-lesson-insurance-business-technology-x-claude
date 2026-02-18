# Lesson 02: Insurance Product Types

## Learning Objectives

By the end of this lesson, you will be able to:
- Differentiate between life and non-life insurance products
- Understand key features of major insurance product categories
- Explain product configuration concepts
- Design product hierarchies for insurance systems

---

## 2.1 Life Insurance Products

Life insurance provides financial protection against death or disability, often combined with savings/investment components.

| Product | Description | Key Features | Target Customer |
|---------|-------------|--------------|-----------------|
| **Term Life** | Coverage for specific period | Low premium, no cash value, pure protection | Young families, mortgage holders |
| **Whole Life** | Lifetime coverage | Cash value accumulation, level premium | Long-term planners, estate planning |
| **Endowment** | Savings + protection | Maturity benefit, guaranteed returns | Education planning, savings |
| **ULIP** | Unit Linked Insurance Plan | Investment + insurance, market-linked | Investors seeking growth |
| **Annuity** | Regular income stream | Pension product, lifetime income | Retirees |
| **Term with Return of Premium** | Term + premium refund | Survival benefit | Risk-averse customers |

### Life Insurance Timeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIFE INSURANCE TIMELINE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TERM LIFE (20 Years)                                           │
│  ├────────────────────────────────────────┤                     │
│  Issue                               Expiry                      │
│  Age 30                              Age 50                      │
│  ↓                                   ↓                          │
│  Premium: $500/year                  Coverage Ends               │
│  Sum Assured: $500,000               No Payout (if alive)       │
│                                                                  │
│  WHOLE LIFE                                                      │
│  ├────────────────────────────────────────────────────────────┤ │
│  Issue                                                   Death   │
│  Age 30                                                  Age 85  │
│  ↓                                                       ↓      │
│  Premium: $2,000/year                Payout: $500,000           │
│  Cash Value builds over time         + Cash Value               │
│                                                                  │
│  ENDOWMENT (25 Years)                                           │
│  ├────────────────────────────────────────────────┤             │
│  Issue                                         Maturity          │
│  Age 30                                        Age 55            │
│  ↓                                             ↓                │
│  Premium: $3,000/year                  Maturity Payout:         │
│                                        $100,000 (guaranteed)    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Non-Life (General) Insurance Products

### Motor Insurance

| Coverage Type | Description | Mandatory |
|---------------|-------------|-----------|
| **Third-Party Liability** | Covers damage/injury to others | Yes (most countries) |
| **Own Damage** | Covers damage to own vehicle | No |
| **Comprehensive** | Third-party + Own Damage + extras | No |
| **Personal Accident** | Covers driver/passenger injury | Varies |

```
┌─────────────────────────────────────────────────────────────────┐
│                    MOTOR INSURANCE COVERAGE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    COMPREHENSIVE                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                  OWN DAMAGE                          │  │  │
│  │  │  ┌───────────────────────────────────────────────┐  │  │  │
│  │  │  │           THIRD-PARTY LIABILITY               │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  │  - Bodily injury to third party               │  │  │  │
│  │  │  │  - Property damage to third party             │  │  │  │
│  │  │  │                                               │  │  │  │
│  │  │  └───────────────────────────────────────────────┘  │  │  │
│  │  │  + Accidental damage to own vehicle                 │  │  │
│  │  │  + Fire damage                                      │  │  │
│  │  │  + Theft                                            │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │  + Natural disasters                                      │  │
│  │  + Roadside assistance                                    │  │
│  │  + Personal accident cover                                │  │
│  │  + Windscreen cover                                       │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Health Insurance

| Product Type | Description | Coverage |
|--------------|-------------|----------|
| **Individual Health** | Single person coverage | Hospitalization, surgery |
| **Family Floater** | Shared sum insured for family | All family members |
| **Group Health** | Employer-sponsored | Employees + dependents |
| **Critical Illness** | Lump sum on diagnosis | Specific diseases |
| **Personal Accident** | Accidental death/disability | AD&D benefits |
| **Top-up/Super Top-up** | Additional coverage | Above threshold |

### Property Insurance

| Product | Covered Perils | Exclusions |
|---------|---------------|------------|
| **Fire Insurance** | Fire, lightning, explosion | War, nuclear, wear & tear |
| **Burglary** | Theft, housebreaking | Employee theft, mysterious disappearance |
| **All Risk** | All perils except excluded | Intentional damage, war |
| **Business Interruption** | Loss of income | Financial loss only (not property) |

### Other Non-Life Products

| Product | Description | Key Features |
|---------|-------------|--------------|
| **Marine Cargo** | Goods in transit | Import/export coverage |
| **Marine Hull** | Ship/vessel damage | Hull and machinery |
| **Liability** | Third-party claims | Professional, public, product |
| **Travel** | Trip-related risks | Medical, cancellation, baggage |
| **Cyber** | Digital risks | Data breach, ransomware |
| **D&O** | Directors & Officers | Management liability |

---

## 2.3 Product Configuration

### Product Hierarchy

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

### Example: Motor Insurance Product Configuration

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

### Premium Calculation Components

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

---

## 2.4 Product Comparison Matrix

| Feature | Term Life | Motor Comprehensive | Health Individual |
|---------|-----------|-------------------|-------------------|
| **Premium Type** | Level | Annual | Annual |
| **Coverage Period** | Fixed term | 1 year | 1 year |
| **Sum Insured** | Fixed | Depreciated value | Fixed/Floater |
| **Claims** | Single (death) | Multiple | Multiple |
| **Waiting Period** | None | None | 30-90 days |
| **Pre-existing** | Underwritten | N/A | Excluded/Waiting |
| **Renewal** | At end of term | Annual | Annual |
| **Cash Value** | No | No | No |

---

## Key Takeaways

1. Life insurance focuses on **mortality risk** with optional savings
2. Non-life insurance covers **property and liability** risks
3. Products are configured with **plans, coverages, and riders**
4. Premium calculation involves **base rates, loadings, and discounts**
5. Understanding product structure is essential for system design

---

## Exercises

1. Design a product configuration for a travel insurance product
2. Calculate premium for a motor policy with given risk factors
3. Compare term life vs whole life for a 35-year-old customer

---

[← Previous Lesson](../01-insurance-fundamentals/README.md) | [Next Lesson: Core Processes →](../03-core-processes/README.md)
