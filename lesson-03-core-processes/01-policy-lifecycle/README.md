# 03.1 Policy Lifecycle

## Overview

The policy lifecycle represents all stages a policy goes through from initial quote to final closure.

## Policy Lifecycle Diagram

![Policy Lifecycle](./03.1-policy-lifecycle.drawio)

## Lifecycle Stages

```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│ QUOTE  │ → │PROPOSAL│ → │ISSUANCE│ → │IN-FORCE│ → │ EXPIRY │
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘
    │            │            │            │            │
    ↓            ↓            ↓            ↓            ↓
 Premium     Document      Policy      Service      Renewal/
 Calculation  Collection   Document    Requests     Lapse
 Comparison   UW Review    Number      Endorsements
 Save/Share   Approval     Effective   Claims
              Payment      Date        Billing
```

## Policy Transactions

| Transaction | Description | Premium Impact |
|-------------|-------------|----------------|
| **Endorsement** | Coverage changes, sum insured adjustment | Pro-rata adjustment |
| **Renewal** | Continue coverage for new term | Re-pricing applied |
| **Cancellation** | Terminate coverage early | Refund (pro-rata/flat) |
| **Reinstatement** | Restore lapsed policy | Pay arrears + fees |

## Status Transitions

```
                          ┌─────────────┐
                          │   QUOTED    │
                          └──────┬──────┘
                                 │ accept
                                 ↓
┌─────────────┐  approve  ┌─────────────┐
│  PROPOSAL   │ ────────→ │   ISSUED    │
│  (Pending)  │           │  (In-Force) │
└─────────────┘           └──────┬──────┘
      │                          │
      │ reject            ┌──────┴──────┐
      ↓                   │             │
┌─────────────┐    lapse  ↓      expiry ↓
│  DECLINED   │    ┌──────────┐  ┌──────────┐
└─────────────┘    │  LAPSED  │  │ EXPIRED  │
                   └────┬─────┘  └──────────┘
                        │
              reinstate │
                        ↓
                   ┌──────────┐
                   │  ACTIVE  │
                   └────┬─────┘
                        │
                cancel  │
                        ↓
                   ┌──────────┐
                   │CANCELLED │
                   └──────────┘
```

## System Requirements

### State Machine
- Valid transitions only
- Audit trail for all changes
- Effective dating support
- Rollback capability

### Quote to Issue
- Premium calculation engine
- Document generation
- Payment integration
- Policy number generation

### Servicing
- Endorsement processing
- Pro-rata premium calculation
- Renewal notification
- Grace period management

---

[← Back to Lesson](../README.md) | [Next: Underwriting →](../02-underwriting/README.md)
