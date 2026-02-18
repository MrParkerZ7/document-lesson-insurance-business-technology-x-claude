# Lesson 03: Core Insurance Processes

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand the complete policy lifecycle
- Explain underwriting principles and workflows
- Describe claims processing from FNOL to settlement
- Design billing and payment collection processes

---

## 3.1 Policy Lifecycle

The policy lifecycle represents all stages a policy goes through from initial quote to final closure.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          POLICY LIFECYCLE                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐            │
│  │ QUOTE  │ → │PROPOSAL│ → │ISSUANCE│ → │IN-FORCE│ → │ EXPIRY │            │
│  └────────┘   └────────┘   └────────┘   └────────┘   └────────┘            │
│      │            │            │            │            │                  │
│      ↓            ↓            ↓            ↓            ↓                  │
│   Premium     Document      Policy      Service      Renewal/              │
│   Calculation  Collection   Document    Requests     Lapse                 │
│   Comparison   UW Review    Number      Endorsements                       │
│   Save/Share   Approval     Effective   Claims                             │
│                Payment      Date        Billing                            │
│                                                                              │
│  ────────────────────────────────────────────────────────────────────────   │
│                                                                              │
│  POLICY TRANSACTIONS:                                                        │
│                                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ENDORSEMENT │  │  RENEWAL   │  │CANCELLATION│  │REINSTATEMENT│           │
│  │            │  │            │  │            │  │            │            │
│  │- Coverage  │  │- Re-pricing│  │- Pro-rata  │  │- Within    │            │
│  │  changes   │  │- UW review │  │  refund    │  │  grace     │            │
│  │- Sum       │  │- Auto/     │  │- Flat      │  │  period    │            │
│  │  insured   │  │  manual    │  │  cancel    │  │- Pay       │            │
│  │- Beneficiary│ │- Lapse     │  │- Non-renew │  │  arrears   │            │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Policy Status Transitions

```
                              ┌─────────────┐
                              │   QUOTED    │
                              └──────┬──────┘
                                     │ accept
                                     ↓
┌─────────────┐  approve     ┌─────────────┐
│  PROPOSAL   │ ───────────→ │   ISSUED    │
│  (Pending)  │              │  (In-Force) │
└─────────────┘              └──────┬──────┘
      │                             │
      │ reject               ┌──────┴──────┐
      ↓                      │             │
┌─────────────┐       lapse  ↓      expiry ↓
│  DECLINED   │       ┌──────────┐  ┌──────────┐
└─────────────┘       │  LAPSED  │  │ EXPIRED  │
                      └────┬─────┘  └──────────┘
                           │
                 reinstate │
                           ↓
                      ┌──────────┐
                      │  ACTIVE  │
                      └──────────┘
                           │
                   cancel  │
                           ↓
                      ┌──────────┐
                      │CANCELLED │
                      └──────────┘
```

---

## 3.2 Underwriting Process

Underwriting is the process of evaluating risk and determining whether to accept, modify, or reject an insurance application.

### Underwriting Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          UNDERWRITING WORKFLOW                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────┐                                                         │
│  │ 1. APPLICATION │                                                         │
│  │    RECEIPT     │                                                         │
│  └───────┬────────┘                                                         │
│          │                                                                   │
│          ↓                                                                   │
│  ┌────────────────┐     ┌────────────────┐                                  │
│  │ 2. DATA        │ ──→ │ External Data  │                                  │
│  │    COLLECTION  │     │ - Medical records                                 │
│  └───────┬────────┘     │ - Credit score │                                  │
│          │              │ - Claims history                                  │
│          ↓              └────────────────┘                                  │
│  ┌────────────────┐                                                         │
│  │ 3. RISK        │     Risk Factors:                                       │
│  │    ASSESSMENT  │     - Age, health, occupation                           │
│  └───────┬────────┘     - Claims history, coverage amount                   │
│          │              - Location, lifestyle                               │
│          ↓                                                                   │
│  ┌────────────────┐     Risk Classification:                                │
│  │ 4. RISK        │     - Preferred, Standard, Substandard                  │
│  │    CLASSIFICATION    - Decline                                           │
│  └───────┬────────┘                                                         │
│          │                                                                   │
│          ↓                                                                   │
│  ┌────────────────┐                                                         │
│  │ 5. RATING      │     Premium = Base Rate × Risk Factor × Coverage        │
│  └───────┬────────┘                                                         │
│          │                                                                   │
│          ↓                                                                   │
│  ┌────────────────┐                                                         │
│  │ 6. DECISION    │ ──→ Accept / Decline / Refer / Counter-offer            │
│  └────────────────┘                                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Underwriting Decision Matrix

| Risk Score | Decision | Action |
|------------|----------|--------|
| 0-30 | Auto Accept | Issue immediately |
| 31-60 | Accept with Loading | Apply premium loading |
| 61-80 | Refer | Manual review required |
| 81-90 | Accept with Exclusions | Add exclusion clauses |
| 91-100 | Decline | Reject application |

### Automated vs Manual Underwriting

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNDERWRITING TYPES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STRAIGHT-THROUGH PROCESSING (STP)                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Application → Rules Engine → Auto Decision → Issuance  │    │
│  └─────────────────────────────────────────────────────────┘    │
│  Criteria:                                                       │
│  - Standard age range (18-55)                                    │
│  - Sum insured within limits                                     │
│  - No adverse medical history                                    │
│  - Clean claims record                                           │
│                                                                  │
│  MANUAL UNDERWRITING                                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Application → Referral → UW Review → Decision → Issue  │    │
│  └─────────────────────────────────────────────────────────┘    │
│  Triggers:                                                       │
│  - High sum insured                                              │
│  - Pre-existing conditions                                       │
│  - Hazardous occupation                                          │
│  - Adverse claims history                                        │
│  - Age outside standard range                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3.3 Claims Management

### Claims Process Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CLAIMS PROCESS FLOW                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │  FNOL    │ → │  TRIAGE  │ → │ DOCUMENT │ → │ ASSESS   │ → │ DECISION │  │
│  │          │   │          │   │          │   │          │   │          │  │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
│       │              │              │              │              │         │
│       ↓              ↓              ↓              ↓              ↓         │
│   Register       Assign        Collect        Investigate    Approve/      │
│   Claim          Priority      Documents      Verify         Reject        │
│   Generate       Assign        Request        Coverage       Partial       │
│   Claim No.      Adjuster      Missing        Fraud Check    Counter       │
│                                                                              │
│                                    ↓                                        │
│                              ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│                              │ RESERVE  │ → │ SETTLE   │ → │  CLOSE   │    │
│                              │          │   │          │   │          │    │
│                              └──────────┘   └──────────┘   └──────────┘    │
│                                   │              │              │          │
│                                   ↓              ↓              ↓          │
│                              Set Reserve    Pay Claim     Close File      │
│                              Update         Deductible    Subrogation     │
│                              Adjust         Co-pay        Recovery        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### FNOL (First Notice of Loss)

| Channel | Information Collected | SLA |
|---------|----------------------|-----|
| **Online Portal** | Structured form, photos, documents | 24/7 |
| **Mobile App** | Location, photos, quick form | 24/7 |
| **Call Center** | Verbal details, transcript | Business hours |
| **Agent** | Paper form, in-person | Business hours |
| **Email** | Unstructured, attachments | 24 hours |

### Claims Status Workflow

```
┌─────────┐
│  DRAFT  │
└────┬────┘
     │ submit
     ↓
┌─────────┐
│  OPEN   │
└────┬────┘
     │ assign
     ↓
┌─────────┐                              ┌─────────────┐
│ ASSIGNED│ ───────────────────────────→ │  ON HOLD    │
└────┬────┘       insufficient docs      └──────┬──────┘
     │                                          │
     │ investigate                              │ docs received
     ↓                                          ↓
┌─────────────┐                          ┌─────────────┐
│UNDER REVIEW │ ←────────────────────────│  ASSIGNED   │
└──────┬──────┘                          └─────────────┘
       │
       ├──────────────────────┬──────────────────────┐
       ↓                      ↓                      ↓
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│  APPROVED   │        │  REJECTED   │        │   PARTIAL   │
└──────┬──────┘        └─────────────┘        └──────┬──────┘
       │                                             │
       │ process                                     │
       ↓                                             ↓
┌─────────────┐                              ┌─────────────┐
│   SETTLED   │                              │   SETTLED   │
└──────┬──────┘                              └──────┬──────┘
       │                                             │
       │ close                                       │
       ↓                                             ↓
┌─────────────┐                              ┌─────────────┐
│   CLOSED    │                              │   CLOSED    │
└─────────────┘                              └─────────────┘
```

### Fraud Detection Indicators

| Indicator Type | Examples | Action |
|---------------|----------|--------|
| **Red Flags** | Recent policy, high claim, late reporting | Investigate |
| **Pattern Matching** | Multiple claims, same adjuster | Alert |
| **Network Analysis** | Connected parties, same repair shop | Deep dive |
| **Document Anomalies** | Edited documents, mismatched dates | Forensic review |

---

## 3.4 Billing & Collections

### Billing Cycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          BILLING CYCLE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Policy                Invoice              Payment              Applied     │
│  Effective             Generated            Due Date             to Policy   │
│     │                     │                    │                    │        │
│     ↓                     ↓                    ↓                    ↓        │
│  ───●─────────────────────●────────────────────●────────────────────●───→   │
│  Day 0                 Day 1               Day 30               Day 30      │
│                                                                              │
│                                                │                             │
│                                                ↓                             │
│                                         ┌─────────────┐                     │
│                                         │ NOT PAID?   │                     │
│                                         └──────┬──────┘                     │
│                                                │                             │
│                              ┌─────────────────┼─────────────────┐          │
│                              ↓                 ↓                 ↓          │
│                        ┌──────────┐     ┌──────────┐     ┌──────────┐      │
│                        │ Reminder │     │  Grace   │     │  Lapse   │      │
│                        │  Day 35  │     │ Day 30-60│     │  Day 60  │      │
│                        └──────────┘     └──────────┘     └──────────┘      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Payment Methods

| Method | Processing Time | Fees | Auto-pay |
|--------|-----------------|------|----------|
| **Credit Card** | Instant | 2-3% | Yes |
| **Debit Card** | Instant | 1-2% | Yes |
| **Bank Transfer** | 1-3 days | Low/None | Yes |
| **Direct Debit** | 2-3 days | Low | Yes |
| **Digital Wallet** | Instant | 1-2% | Yes |
| **Cash (Agent)** | Same day | None | No |
| **Check** | 3-5 days | Low | No |

### Commission Management

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMMISSION FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Premium Collected                                               │
│        │                                                         │
│        ↓                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              COMMISSION CALCULATION                       │   │
│  │                                                           │   │
│  │  First Year Commission = Premium × FYC Rate (15-40%)     │   │
│  │  Renewal Commission = Premium × Renewal Rate (5-15%)     │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              COMMISSION HIERARCHY                         │   │
│  │                                                           │   │
│  │  Agent (70%) ← Manager (20%) ← Director (10%)            │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│        │                                                         │
│        ↓                                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              CLAWBACK RULES                               │   │
│  │                                                           │   │
│  │  If policy cancelled within 12 months:                   │   │
│  │  Clawback = Commission × (12 - Months Active) / 12       │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3.5 Document Management

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
┌─────────────────────────────────────────────────────────────────┐
│                    DOCUMENT WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐     │
│  │  UPLOAD  │ → │ VALIDATE │ → │  INDEX   │ → │  STORE   │     │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘     │
│       │              │              │              │            │
│       ↓              ↓              ↓              ↓            │
│    Capture       Verify         Extract        Secure          │
│    OCR           Format         Metadata       Archive         │
│    Classify      Completeness   Link to        Replicate       │
│                                 Entity                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. Policy lifecycle spans from **quote to expiry/renewal**
2. Underwriting balances **risk acceptance** with **profitability**
3. Claims processing requires **speed, accuracy, and fraud prevention**
4. Billing must support **multiple payment methods** and **grace periods**
5. Document management ensures **compliance** and **auditability**

---

## Exercises

1. Design a state machine for policy status transitions
2. Create underwriting rules for a health insurance product
3. Map the claims process for a motor accident claim

---

[← Previous Lesson](../02-insurance-products/README.md) | [Next Lesson: System Architecture →](../04-system-architecture/README.md)
