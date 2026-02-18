# 03.4 Billing & Collections

## Overview

Billing and collections manage premium invoicing, payment processing, commission distribution, and document retention.

## Billing Cycle

```
Policy                Invoice              Payment              Applied
Effective             Generated            Due Date             to Policy
   │                     │                    │                    │
   ↓                     ↓                    ↓                    ↓
───●─────────────────────●────────────────────●────────────────────●───→
Day 0                 Day 1               Day 30               Day 30

                                              │
                                              ↓
                                       ┌─────────────┐
                                       │ NOT PAID?   │
                                       └──────┬──────┘
                                              │
                            ┌─────────────────┼─────────────────┐
                            ↓                 ↓                 ↓
                      ┌──────────┐     ┌──────────┐     ┌──────────┐
                      │ Reminder │     │  Grace   │     │  Lapse   │
                      │  Day 35  │     │ Day 30-60│     │  Day 60  │
                      └──────────┘     └──────────┘     └──────────┘
```

## Payment Methods

| Method | Processing Time | Fees | Auto-pay |
|--------|-----------------|------|----------|
| **Credit Card** | Instant | 2-3% | Yes |
| **Debit Card** | Instant | 1-2% | Yes |
| **Bank Transfer** | 1-3 days | Low/None | Yes |
| **Direct Debit** | 2-3 days | Low | Yes |
| **Digital Wallet** | Instant | 1-2% | Yes |
| **Cash (Agent)** | Same day | None | No |
| **Check** | 3-5 days | Low | No |

## Commission Management

```
Premium Collected
      │
      ↓
┌──────────────────────────────────────────────────────────┐
│              COMMISSION CALCULATION                       │
│                                                           │
│  First Year Commission = Premium × FYC Rate (15-40%)     │
│  Renewal Commission = Premium × Renewal Rate (5-15%)     │
└──────────────────────────────────────────────────────────┘
      │
      ↓
┌──────────────────────────────────────────────────────────┐
│              COMMISSION HIERARCHY                         │
│                                                           │
│  Agent (70%) ← Manager (20%) ← Director (10%)            │
└──────────────────────────────────────────────────────────┘
      │
      ↓
┌──────────────────────────────────────────────────────────┐
│              CLAWBACK RULES                               │
│                                                           │
│  If policy cancelled within 12 months:                   │
│  Clawback = Commission × (12 - Months Active) / 12       │
└──────────────────────────────────────────────────────────┘
```

## Document Management

### Document Types

| Category | Documents | Retention |
|----------|-----------|-----------|
| **Application** | Proposal form, KYC documents | Policy term + 7 years |
| **Policy** | Policy document, schedule, endorsements | Policy term + 7 years |
| **Claims** | Claim form, invoices, photos, reports | Settlement + 7 years |
| **Billing** | Invoices, receipts, statements | 7 years |
| **Correspondence** | Emails, letters, call recordings | 3-7 years |

### Document Workflow

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  UPLOAD  │ → │ VALIDATE │ → │  INDEX   │ → │  STORE   │
└──────────┘   └──────────┘   └──────────┘   └──────────┘
     │              │              │              │
     ↓              ↓              ↓              ↓
  Capture       Verify         Extract        Secure
  OCR           Format         Metadata       Archive
  Classify      Completeness   Link to        Replicate
                               Entity
```

## System Requirements

- Payment gateway integration
- Multi-currency support
- Commission calculation engine
- Clawback tracking
- Document capture and OCR
- Retention policy enforcement
- Audit trail

---

[← Previous: Claims Management](../03-claims-management/README.md) | [Next Lesson: System Architecture →](../../lesson-04-system-architecture/README.md)
