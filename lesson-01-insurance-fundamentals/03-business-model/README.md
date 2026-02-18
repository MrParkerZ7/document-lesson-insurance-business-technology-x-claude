# 01.3 Insurance Business Model

## Overview

Understanding the insurance business model is essential for IT developers and product owners to build systems that support revenue generation, cost management, and financial reporting.

## Revenue Streams

| Revenue Type | Description | Example |
|--------------|-------------|---------|
| **Premium Income** | Money collected from policyholders | Annual motor premium of $1,200 |
| **Investment Income** | Returns from investing premium reserves | Interest on bonds, stock dividends |
| **Fee Income** | Administrative fees, policy fees | Policy issuance fee of $50 |
| **Commission Income** | For agents/brokers selling policies | 15% commission on premium |

## Cost Structure

| Cost Type | Description | Typical % of Premium |
|-----------|-------------|---------------------|
| **Claims Payout** | Largest expense (Loss Ratio) | 60-80% |
| **Operating Expenses** | Salaries, technology, marketing | 15-25% |
| **Commissions** | Paid to agents/brokers | 5-20% |
| **Reinsurance Costs** | Premium paid to reinsurers | 5-15% |

## Key Financial Metrics

| Metric | Formula | Target | Description |
|--------|---------|--------|-------------|
| **Loss Ratio** | Claims Paid / Premiums Earned | < 70% | Measures claims efficiency |
| **Expense Ratio** | Operating Expenses / Premiums Written | < 25% | Measures operational efficiency |
| **Combined Ratio** | Loss Ratio + Expense Ratio | < 100% | < 100% = Underwriting profit |
| **Solvency Ratio** | Assets / Liabilities | > 150% | Measures financial health |
| **Retention Ratio** | Renewed Policies / Expiring Policies | > 85% | Customer loyalty indicator |

## Profitability Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    INSURANCE PROFITABILITY                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   INCOME                           EXPENSES                      │
│   ┌─────────────────┐             ┌─────────────────┐           │
│   │ Premium Income  │             │ Claims Paid     │           │
│   │    $1,000,000   │      -      │   $650,000      │           │
│   └─────────────────┘             └─────────────────┘           │
│           +                               +                      │
│   ┌─────────────────┐             ┌─────────────────┐           │
│   │Investment Income│             │ Operating Costs │           │
│   │    $50,000      │             │   $200,000      │           │
│   └─────────────────┘             └─────────────────┘           │
│           +                               +                      │
│   ┌─────────────────┐             ┌─────────────────┐           │
│   │ Fee Income      │             │ Commissions     │           │
│   │    $25,000      │             │   $100,000      │           │
│   └─────────────────┘             └─────────────────┘           │
│                                                                  │
│   ─────────────────────────────────────────────────────────     │
│   Total Income: $1,075,000    Total Expenses: $950,000          │
│                                                                  │
│   NET PROFIT: $125,000 (11.6% margin)                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## System Implications

### For Developers
- Premium calculation engines must be accurate and auditable
- Financial transactions require ACID compliance
- Real-time reporting dashboards for management
- Integration with accounting systems (GL posting)

### For Product Owners
- Features should consider impact on combined ratio
- User journeys should optimize for retention
- Digital channels can reduce expense ratio
- Analytics capabilities for loss ratio monitoring

---

[← Previous: Value Chain](../02-value-chain/README.md) | [Next: Distribution Channels →](../04-distribution-channels/README.md)
