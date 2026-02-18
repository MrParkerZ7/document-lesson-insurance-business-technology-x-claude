# 03.3 Claims Management

## Overview

Claims management is the process of handling insurance claims from first notification through settlement and closure.

## Claims Process Flow Diagram

![Claims Process Flow](./03.2-claims-process-flow.drawio)

## Process Overview

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  FNOL    │ → │  TRIAGE  │ → │ DOCUMENT │ → │ ASSESS   │ → │ DECISION │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
     │              │              │              │              │
     ↓              ↓              ↓              ↓              ↓
 Register       Assign        Collect        Investigate    Approve/
 Claim          Priority      Documents      Verify         Reject
 Generate       Assign        Request        Coverage       Partial
 Claim No.      Adjuster      Missing        Fraud Check    Counter

                                  ↓
                            ┌──────────┐   ┌──────────┐   ┌──────────┐
                            │ RESERVE  │ → │ SETTLE   │ → │  CLOSE   │
                            └──────────┘   └──────────┘   └──────────┘
                                 │              │              │
                                 ↓              ↓              ↓
                            Set Reserve    Pay Claim     Close File
                            Update         Deductible    Subrogation
                            Adjust         Co-pay        Recovery
```

## FNOL (First Notice of Loss)

| Channel | Information Collected | SLA |
|---------|----------------------|-----|
| **Online Portal** | Structured form, photos, documents | 24/7 |
| **Mobile App** | Location, photos, quick form | 24/7 |
| **Call Center** | Verbal details, transcript | Business hours |
| **Agent** | Paper form, in-person | Business hours |
| **Email** | Unstructured, attachments | 24 hours |

## Claims Status Workflow

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
┌─────────┐                    ┌─────────────┐
│ ASSIGNED│ ─────────────────→ │  ON HOLD    │
└────┬────┘  insufficient docs └──────┬──────┘
     │                                │ docs received
     │ investigate                    ↓
     ↓                         ┌─────────────┐
┌─────────────┐                │  ASSIGNED   │
│UNDER REVIEW │ ←──────────────┘
└──────┬──────┘
       │
       ├────────────┬────────────┐
       ↓            ↓            ↓
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  APPROVED   │ │  REJECTED   │ │   PARTIAL   │
└──────┬──────┘ └─────────────┘ └──────┬──────┘
       │                               │
       ↓                               ↓
┌─────────────┐                 ┌─────────────┐
│   SETTLED   │                 │   SETTLED   │
└──────┬──────┘                 └──────┬──────┘
       │                               │
       ↓                               ↓
┌─────────────┐                 ┌─────────────┐
│   CLOSED    │                 │   CLOSED    │
└─────────────┘                 └─────────────┘
```

## Fraud Detection

| Indicator Type | Examples | Action |
|---------------|----------|--------|
| **Red Flags** | Recent policy, high claim, late reporting | Investigate |
| **Pattern Matching** | Multiple claims, same adjuster | Alert |
| **Network Analysis** | Connected parties, same repair shop | Deep dive |
| **Document Anomalies** | Edited documents, mismatched dates | Forensic review |

## Reserve Management

| Reserve Type | Description |
|--------------|-------------|
| **Case Reserve** | Estimated cost for specific claim |
| **IBNR** | Incurred But Not Reported |
| **IBNER** | Incurred But Not Enough Reserved |

## System Requirements

- Multi-channel FNOL capture
- Automated triage and routing
- Document management integration
- Fraud detection algorithms
- Reserve calculation engine
- Payment processing
- Subrogation tracking

---

[← Previous: Underwriting](../02-underwriting/README.md) | [Next: Billing & Collections →](../04-billing-collections/README.md)
