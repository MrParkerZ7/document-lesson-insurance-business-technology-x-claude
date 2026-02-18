# 03.2 Underwriting Process

## Overview

Underwriting is the process of evaluating risk and determining whether to accept, modify, or reject an insurance application.

## Underwriting Workflow

```
┌────────────────┐
│ 1. APPLICATION │
│    RECEIPT     │
└───────┬────────┘
        │
        ↓
┌────────────────┐     ┌────────────────┐
│ 2. DATA        │ ──→ │ External Data  │
│    COLLECTION  │     │ - Medical records
└───────┬────────┘     │ - Credit score │
        │              │ - Claims history
        ↓              └────────────────┘
┌────────────────┐
│ 3. RISK        │     Risk Factors:
│    ASSESSMENT  │     - Age, health, occupation
└───────┬────────┘     - Claims history, coverage
        │
        ↓
┌────────────────┐     Risk Classification:
│ 4. RISK        │     - Preferred, Standard
│    CLASSIFICATION    - Substandard, Decline
└───────┬────────┘
        │
        ↓
┌────────────────┐
│ 5. RATING      │     Premium = Base × Risk Factor × Coverage
└───────┬────────┘
        │
        ↓
┌────────────────┐
│ 6. DECISION    │ ──→ Accept / Decline / Refer / Counter-offer
└────────────────┘
```

## Decision Matrix

| Risk Score | Decision | Action |
|------------|----------|--------|
| 0-30 | Auto Accept | Issue immediately |
| 31-60 | Accept with Loading | Apply premium loading |
| 61-80 | Refer | Manual review required |
| 81-90 | Accept with Exclusions | Add exclusion clauses |
| 91-100 | Decline | Reject application |

## Straight-Through Processing (STP)

```
Application → Rules Engine → Auto Decision → Issuance
```

**STP Criteria:**
- Standard age range (18-55)
- Sum insured within limits
- No adverse medical history
- Clean claims record

## Manual Underwriting Triggers

| Trigger | Reason |
|---------|--------|
| High sum insured | Increased risk exposure |
| Pre-existing conditions | Medical underwriting needed |
| Hazardous occupation | Occupational risk assessment |
| Adverse claims history | Loss experience review |
| Age outside standard range | Mortality risk evaluation |

## Risk Factors by Product

| Product | Key Risk Factors |
|---------|------------------|
| **Life Insurance** | Age, health, occupation, lifestyle |
| **Motor Insurance** | Vehicle type, driver age, claims history |
| **Health Insurance** | Age, pre-existing conditions, lifestyle |
| **Property Insurance** | Location, construction, occupancy |

## System Requirements

- Rules engine for automated decisions
- Integration with external data providers
- Workflow for manual referrals
- Decision audit trail
- Loading/discount calculation

---

[← Previous: Policy Lifecycle](../01-policy-lifecycle/README.md) | [Next: Claims Management →](../03-claims-management/README.md)
