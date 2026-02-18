# 02.1 Life Insurance Products

## Overview

Life insurance provides financial protection against death or disability, often combined with savings/investment components.

## Product Types

| Product | Description | Key Features | Target Customer |
|---------|-------------|--------------|-----------------|
| **Term Life** | Coverage for specific period | Low premium, no cash value, pure protection | Young families, mortgage holders |
| **Whole Life** | Lifetime coverage | Cash value accumulation, level premium | Long-term planners, estate planning |
| **Endowment** | Savings + protection | Maturity benefit, guaranteed returns | Education planning, savings |
| **ULIP** | Unit Linked Insurance Plan | Investment + insurance, market-linked | Investors seeking growth |
| **Annuity** | Regular income stream | Pension product, lifetime income | Retirees |
| **Term with Return of Premium** | Term + premium refund | Survival benefit | Risk-averse customers |

## Life Insurance Timeline

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

## Key Characteristics

### Term Life
- Pure risk coverage with no savings component
- Most affordable life insurance option
- Coverage for specific period (10, 20, 30 years)
- No cash value accumulation
- Premium increases at renewal

### Whole Life
- Lifetime coverage guaranteed
- Premium remains level throughout
- Builds cash value over time
- Can borrow against cash value
- Higher initial premium than term

### ULIP (Unit Linked Insurance Plan)
- Combines insurance with investment
- Premium split between charges and investment
- Market-linked returns (equity/debt funds)
- Policyholder bears investment risk
- Lock-in period typically 5 years

### Annuity
- Converts lump sum into regular income
- Immediate or deferred annuity
- Life annuity or period certain
- Protects against longevity risk

## System Considerations

- Life insurance requires complex mortality calculations
- Cash value tracking for whole life products
- Fund NAV calculations for ULIPs
- Annuity payment scheduling
- Nomination and assignment tracking

---

[← Back to Lesson](../README.md) | [Next: Non-Life Insurance →](../02-non-life-insurance/README.md)
