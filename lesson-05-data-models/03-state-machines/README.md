# 03 - State Machines

## Overview

State machines define the valid status transitions for insurance entities. Understanding these transitions is critical for implementing correct business logic and ensuring data integrity throughout the policy and claim lifecycles.

---

## Policy Status State Machine

```
+-----------------------------------------------------------------------------+
|                       POLICY STATUS STATE MACHINE                            |
+-----------------------------------------------------------------------------+
|                                                                              |
|                         +----------+                                         |
|                         |  DRAFT   |                                         |
|                         +----+-----+                                         |
|                              | submit_quote                                  |
|                              v                                               |
|                         +----------+                                         |
|                         |  QUOTED  | <-----------------+                     |
|                         +----+-----+                   |                     |
|                              | convert_proposal        | requote             |
|                              v                         |                     |
|                         +----------+                   |                     |
|                         | PROPOSED | ------------------+                     |
|                         +----+-----+                   |                     |
|                              |                         |                     |
|              +---------------+---------------+         |                     |
|              v                               v         |                     |
|        +----------+                    +----------+    |                     |
|        | DECLINED |                    |  ISSUED  |    |                     |
|        +----------+                    +----+-----+    |                     |
|                                             | activate |                     |
|                                             v          |                     |
|                                        +----------+    |                     |
|              +-------------------------|  ACTIVE  |----+                     |
|              |                         +----+-----+                          |
|              |                              |                                |
|              |         +--------------------+--------------------+           |
|              |         v                    v                    v           |
|              |   +----------+        +----------+         +----------+       |
|              |   |  LAPSED  |        | EXPIRED  |         |CANCELLED |       |
|              |   +----+-----+        +----------+         +----------+       |
|              |        |                                                      |
|              |        | reinstate                                            |
|              |        v                                                      |
|              |   +----------+                                                |
|              +-->|  ACTIVE  |                                                |
|                  +----------+                                                |
|                                                                              |
+-----------------------------------------------------------------------------+
```

### Policy Status Transitions

| From Status | To Status | Trigger | Description |
|-------------|-----------|---------|-------------|
| DRAFT | QUOTED | submit_quote | Premium has been calculated |
| QUOTED | PROPOSED | convert_proposal | Documents submitted for underwriting |
| PROPOSED | ISSUED | approve | Underwriting passed, payment received |
| PROPOSED | DECLINED | reject | Underwriting failed |
| PROPOSED | QUOTED | requote | Re-calculation required |
| ISSUED | ACTIVE | activate | Effective date reached |
| ACTIVE | LAPSED | lapse | Premium not paid within grace period |
| ACTIVE | CANCELLED | cancel | Customer or insurer initiated cancellation |
| ACTIVE | EXPIRED | expire | Expiry date reached, not renewed |
| ACTIVE | QUOTED | requote | Policy modification requires re-quote |
| LAPSED | ACTIVE | reinstate | Arrears paid within reinstatement period |

### Policy Status Definitions

| Status | Description |
|--------|-------------|
| DRAFT | Initial state, policy being created |
| QUOTED | Premium calculated, awaiting proposal |
| PROPOSED | Submitted for underwriting |
| DECLINED | Underwriting rejected |
| ISSUED | Approved but not yet effective |
| ACTIVE | Policy in force |
| LAPSED | Premium payment missed |
| CANCELLED | Terminated before expiry |
| EXPIRED | Term ended normally |
| MATURED | Life policy reached maturity (for life products) |

---

## Claim Status State Machine

```
+-----------------------------------------------------------------------------+
|                       CLAIM STATUS STATE MACHINE                             |
+-----------------------------------------------------------------------------+
|                                                                              |
|                              +----------+                                    |
|                              |  DRAFT   |                                    |
|                              +----+-----+                                    |
|                                   | submit                                   |
|                                   v                                          |
|                              +----------+                                    |
|                              |   OPEN   |                                    |
|                              +----+-----+                                    |
|                                   | assign                                   |
|                                   v                                          |
|                              +----------+                                    |
|                              | ASSIGNED |                                    |
|                              +----+-----+                                    |
|                                   | investigate                              |
|                                   v                                          |
|                              +----------+       +----------+                 |
|                     +--------|  UNDER   |<------|  ON_HOLD |                 |
|                     |        |  REVIEW  |------>|          |                 |
|                     |        +----+-----+ hold  +----------+                 |
|                     |             |       resume                             |
|                     |             | decide                                   |
|        +------------+-------------+-------------+-------------------+        |
|        v            v             v             v                   v        |
|  +----------+ +----------+ +----------+ +------------+       +----------+    |
|  | APPROVED | | PARTIAL  | | REJECTED | |   FRAUD    |       |   SIU    |    |
|  +----+-----+ +----+-----+ +----+-----+ |  REFERRED  |       | REFERRAL |    |
|       |            |            |       +------------+       +----------+    |
|       | process    | process    |                                            |
|       v            v            |                                            |
|  +----------+ +----------+      |                                            |
|  | SETTLED  | | SETTLED  |      |                                            |
|  +----+-----+ +----+-----+      |                                            |
|       |            |            |                                            |
|       | close      | close      | close                                      |
|       v            v            v                                            |
|  +---------------------------------------------+                             |
|  |                   CLOSED                     |                             |
|  +---------------------------------------------+                             |
|                    |                                                         |
|                    | reopen                                                  |
|                    v                                                         |
|              +----------+                                                    |
|              | REOPENED | --> (returns to UNDER_REVIEW)                      |
|              +----------+                                                    |
|                                                                              |
+-----------------------------------------------------------------------------+
```

### Claim Status Transitions

| From Status | To Status | Trigger | Description |
|-------------|-----------|---------|-------------|
| DRAFT | OPEN | submit | Claim submitted by customer |
| OPEN | ASSIGNED | assign | Adjuster assigned |
| ASSIGNED | UNDER_REVIEW | investigate | Investigation started |
| UNDER_REVIEW | ON_HOLD | hold | Waiting for documents/information |
| ON_HOLD | UNDER_REVIEW | resume | Required information received |
| UNDER_REVIEW | APPROVED | approve | Full claim approved |
| UNDER_REVIEW | PARTIALLY_APPROVED | partial_approve | Partial claim approved |
| UNDER_REVIEW | REJECTED | reject | Claim denied |
| UNDER_REVIEW | FRAUD_REFERRED | refer_fraud | Suspicious activity detected |
| APPROVED | SETTLED | process | Payment processed |
| PARTIALLY_APPROVED | SETTLED | process | Partial payment processed |
| SETTLED | CLOSED | close | Claim finalized |
| REJECTED | CLOSED | close | Denial finalized |
| CLOSED | REOPENED | reopen | New evidence or appeal |
| REOPENED | UNDER_REVIEW | investigate | Re-investigation |

### Claim Status Definitions

| Status | Description |
|--------|-------------|
| DRAFT | Claim being created |
| OPEN | Submitted, awaiting assignment |
| ASSIGNED | Adjuster assigned |
| UNDER_REVIEW | Active investigation |
| ON_HOLD | Waiting for additional information |
| APPROVED | Full amount approved |
| PARTIALLY_APPROVED | Partial amount approved |
| REJECTED | Claim denied |
| FRAUD_REFERRED | Sent to Special Investigations Unit |
| SETTLED | Payment completed |
| CLOSED | Claim finalized |
| REOPENED | Reopened for review |

---

## Implementation Guidelines

### State Machine Implementation

```typescript
// Example TypeScript implementation
interface StateTransition {
  from: string;
  to: string;
  trigger: string;
  guard?: () => boolean;
  action?: () => void;
}

const policyTransitions: StateTransition[] = [
  { from: 'DRAFT', to: 'QUOTED', trigger: 'submit_quote' },
  { from: 'QUOTED', to: 'PROPOSED', trigger: 'convert_proposal' },
  { from: 'PROPOSED', to: 'ISSUED', trigger: 'approve' },
  { from: 'PROPOSED', to: 'DECLINED', trigger: 'reject' },
  { from: 'ISSUED', to: 'ACTIVE', trigger: 'activate' },
  { from: 'ACTIVE', to: 'LAPSED', trigger: 'lapse' },
  { from: 'ACTIVE', to: 'CANCELLED', trigger: 'cancel' },
  { from: 'ACTIVE', to: 'EXPIRED', trigger: 'expire' },
  { from: 'LAPSED', to: 'ACTIVE', trigger: 'reinstate' },
];

function canTransition(currentStatus: string, targetStatus: string): boolean {
  return policyTransitions.some(
    t => t.from === currentStatus && t.to === targetStatus
  );
}
```

### Database Constraints

```sql
-- Ensure valid status transitions at database level
CREATE OR REPLACE FUNCTION validate_policy_status_transition()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.status = 'DRAFT' AND NEW.status NOT IN ('QUOTED') THEN
    RAISE EXCEPTION 'Invalid status transition from DRAFT to %', NEW.status;
  END IF;

  IF OLD.status = 'ACTIVE' AND NEW.status NOT IN ('LAPSED', 'CANCELLED', 'EXPIRED', 'QUOTED') THEN
    RAISE EXCEPTION 'Invalid status transition from ACTIVE to %', NEW.status;
  END IF;

  -- Add more validation rules...

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_policy_status_transition
BEFORE UPDATE OF status ON policy
FOR EACH ROW
EXECUTE FUNCTION validate_policy_status_transition();
```

### Status History Tracking

```sql
-- Track all status changes
CREATE TABLE status_history (
    history_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type         VARCHAR(50) NOT NULL,          -- POLICY, CLAIM
    entity_id           UUID NOT NULL,
    old_status          VARCHAR(20),
    new_status          VARCHAR(20) NOT NULL,
    transition_trigger  VARCHAR(50),
    reason              TEXT,
    changed_by          VARCHAR(100) NOT NULL,
    changed_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_status_history_entity ON status_history(entity_type, entity_id);
```

---

## Best Practices

1. **Always validate transitions** - Never allow direct status updates without validation
2. **Track status history** - Maintain complete audit trail of all transitions
3. **Use triggers for enforcement** - Database-level validation as last line of defense
4. **Document all transitions** - Clear documentation for business and technical teams
5. **Handle edge cases** - Define behavior for invalid transitions
6. **Consider sub-statuses** - Use sub_status for granular state within main status

---

## Navigation

[Previous: Database Schemas](../02-database-schemas/README.md) | [Back to Lesson 05](../README.md)
