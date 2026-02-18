# 02.2 Non-Life (General) Insurance Products

## Overview

Non-life insurance covers property, liability, and short-term risks. Policies are typically annual and renewable.

## Motor Insurance

| Coverage Type | Description | Mandatory |
|---------------|-------------|-----------|
| **Third-Party Liability** | Covers damage/injury to others | Yes (most countries) |
| **Own Damage** | Covers damage to own vehicle | No |
| **Comprehensive** | Third-party + Own Damage + extras | No |
| **Personal Accident** | Covers driver/passenger injury | Varies |

### Motor Coverage Layers

```
┌───────────────────────────────────────────────────────────────┐
│                    COMPREHENSIVE                               │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                  OWN DAMAGE                              │  │
│  │  ┌───────────────────────────────────────────────────┐  │  │
│  │  │           THIRD-PARTY LIABILITY                   │  │  │
│  │  │  - Bodily injury to third party                   │  │  │
│  │  │  - Property damage to third party                 │  │  │
│  │  └───────────────────────────────────────────────────┘  │  │
│  │  + Accidental damage to own vehicle                     │  │
│  │  + Fire damage                                          │  │
│  │  + Theft                                                │  │
│  └─────────────────────────────────────────────────────────┘  │
│  + Natural disasters                                          │
│  + Roadside assistance                                        │
│  + Personal accident cover                                    │
│  + Windscreen cover                                           │
└───────────────────────────────────────────────────────────────┘
```

## Health Insurance

| Product Type | Description | Coverage |
|--------------|-------------|----------|
| **Individual Health** | Single person coverage | Hospitalization, surgery |
| **Family Floater** | Shared sum insured for family | All family members |
| **Group Health** | Employer-sponsored | Employees + dependents |
| **Critical Illness** | Lump sum on diagnosis | Specific diseases |
| **Personal Accident** | Accidental death/disability | AD&D benefits |
| **Top-up/Super Top-up** | Additional coverage | Above threshold |

## Property Insurance

| Product | Covered Perils | Exclusions |
|---------|---------------|------------|
| **Fire Insurance** | Fire, lightning, explosion | War, nuclear, wear & tear |
| **Burglary** | Theft, housebreaking | Employee theft, mysterious disappearance |
| **All Risk** | All perils except excluded | Intentional damage, war |
| **Business Interruption** | Loss of income | Financial loss only (not property) |

## Specialty Products

| Product | Description | Key Features |
|---------|-------------|--------------|
| **Marine Cargo** | Goods in transit | Import/export coverage |
| **Marine Hull** | Ship/vessel damage | Hull and machinery |
| **Liability** | Third-party claims | Professional, public, product |
| **Travel** | Trip-related risks | Medical, cancellation, baggage |
| **Cyber** | Digital risks | Data breach, ransomware |
| **D&O** | Directors & Officers | Management liability |

## Product Comparison

| Feature | Motor Comprehensive | Health Individual | Property Fire |
|---------|-------------------|-------------------|---------------|
| **Coverage Period** | 1 year | 1 year | 1 year |
| **Sum Insured** | Depreciated value | Fixed/Floater | Property value |
| **Claims** | Multiple | Multiple | As per loss |
| **Waiting Period** | None | 30-90 days | None |
| **Renewal** | Annual | Annual | Annual |

## System Considerations

- Non-life products require annual renewal processing
- Claims can be partial (repair) or total (replacement)
- Third-party liability requires legal compliance
- Health claims involve hospital networks and cashless processing
- Motor requires integration with RTO/DMV databases

---

[← Previous: Life Insurance](../01-life-insurance/README.md) | [Next: Product Configuration →](../03-product-configuration/README.md)
