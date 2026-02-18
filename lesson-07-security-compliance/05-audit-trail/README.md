# 7.5 Audit Trail

## Overview

Comprehensive audit logging is essential for insurance systems to ensure accountability, support forensic investigations, and meet regulatory compliance requirements. This sub-lesson covers audit log design, implementation, and best practices.

---

## Audit Log Schema

A well-designed audit log captures all necessary information for compliance and investigation:

```sql
CREATE TABLE audit_log (
    audit_id            UUID PRIMARY KEY,

    -- What was accessed/changed
    entity_type         VARCHAR(50) NOT NULL,
    entity_id           UUID NOT NULL,
    entity_reference    VARCHAR(100),

    -- Action details
    action              VARCHAR(20) NOT NULL,
    action_category     VARCHAR(50),
    field_changes       JSONB,

    -- Who performed the action
    user_id             VARCHAR(100) NOT NULL,
    user_name           VARCHAR(255),
    user_role           VARCHAR(50),
    user_department     VARCHAR(100),

    -- Context
    ip_address          VARCHAR(45),
    user_agent          TEXT,
    session_id          VARCHAR(100),
    request_id          VARCHAR(100),
    correlation_id      VARCHAR(100),

    -- Business justification
    reason              TEXT,
    ticket_reference    VARCHAR(100),

    -- Timestamp
    timestamp           TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- Indexes
    INDEX idx_audit_entity (entity_type, entity_id),
    INDEX idx_audit_user (user_id, timestamp),
    INDEX idx_audit_timestamp (timestamp)
);
```

### Field Change Tracking

```json
{
  "field_changes": {
    "status": {
      "old_value": "PENDING",
      "new_value": "APPROVED"
    },
    "premium_amount": {
      "old_value": 1500.00,
      "new_value": 1450.00
    },
    "coverage_limit": {
      "old_value": 500000,
      "new_value": 750000
    }
  }
}
```

---

## Audit Events

Different types of events require different levels of detail:

| Action | Description | Logged Data |
|--------|-------------|-------------|
| **VIEW** | Data accessed | User, entity, fields viewed |
| **CREATE** | New record | User, entity, all values |
| **UPDATE** | Record modified | User, entity, old/new values |
| **DELETE** | Record removed | User, entity, reason |
| **EXPORT** | Data exported | User, criteria, record count |
| **LOGIN** | User login | User, IP, device, MFA used |
| **LOGOUT** | User logout | User, session duration |
| **PERMISSION_CHANGE** | Access modified | User, old/new permissions |

### Event Categories

```
+-----------------------------------------------------------------------+
|                       AUDIT EVENT CATEGORIES                           |
+-----------------------------------------------------------------------+
|                                                                        |
|  AUTHENTICATION                                                        |
|  +------------------------------------------------------------------+ |
|  | LOGIN | LOGOUT | LOGIN_FAILED | PASSWORD_CHANGE | MFA_ENABLED    | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  DATA ACCESS                                                           |
|  +------------------------------------------------------------------+ |
|  | VIEW | SEARCH | EXPORT | DOWNLOAD | PRINT                        | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  DATA MODIFICATION                                                     |
|  +------------------------------------------------------------------+ |
|  | CREATE | UPDATE | DELETE | ARCHIVE | RESTORE                     | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  ADMINISTRATIVE                                                        |
|  +------------------------------------------------------------------+ |
|  | PERMISSION_GRANT | PERMISSION_REVOKE | ROLE_ASSIGN | CONFIG_CHANGE| |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  BUSINESS OPERATIONS                                                   |
|  +------------------------------------------------------------------+ |
|  | QUOTE_CREATE | POLICY_ISSUE | CLAIM_SUBMIT | PAYMENT_PROCESS     | |
|  +------------------------------------------------------------------+ |
|                                                                        |
+-----------------------------------------------------------------------+
```

---

## Audit Log Implementation

### Audit Service

```python
from datetime import datetime
from typing import Optional, Dict, Any
import uuid

class AuditService:
    """Service for creating audit log entries"""

    def __init__(self, repository: AuditRepository):
        self.repository = repository

    def log_event(
        self,
        entity_type: str,
        entity_id: str,
        action: str,
        user_context: UserContext,
        field_changes: Optional[Dict[str, Any]] = None,
        reason: Optional[str] = None,
        ticket_reference: Optional[str] = None
    ) -> str:
        """Create an audit log entry"""

        audit_entry = AuditLog(
            audit_id=str(uuid.uuid4()),
            entity_type=entity_type,
            entity_id=entity_id,
            action=action,
            field_changes=field_changes,
            user_id=user_context.user_id,
            user_name=user_context.user_name,
            user_role=user_context.role,
            user_department=user_context.department,
            ip_address=user_context.ip_address,
            user_agent=user_context.user_agent,
            session_id=user_context.session_id,
            request_id=user_context.request_id,
            reason=reason,
            ticket_reference=ticket_reference,
            timestamp=datetime.utcnow()
        )

        return self.repository.save(audit_entry)

    def log_view(self, entity_type: str, entity_id: str, user_context: UserContext):
        """Log a data access event"""
        return self.log_event(entity_type, entity_id, "VIEW", user_context)

    def log_update(
        self,
        entity_type: str,
        entity_id: str,
        old_values: Dict,
        new_values: Dict,
        user_context: UserContext
    ):
        """Log an update event with field changes"""
        field_changes = {}
        for key in new_values:
            if key in old_values and old_values[key] != new_values[key]:
                field_changes[key] = {
                    "old_value": old_values[key],
                    "new_value": new_values[key]
                }

        return self.log_event(
            entity_type, entity_id, "UPDATE", user_context,
            field_changes=field_changes
        )
```

### Automatic Audit Logging Decorator

```python
def audit_logged(entity_type: str, action: str):
    """Decorator to automatically log function calls"""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            user_context = get_current_user_context()
            entity_id = kwargs.get('entity_id') or kwargs.get('id')

            # Execute the function
            result = await func(*args, **kwargs)

            # Log the audit event
            audit_service.log_event(
                entity_type=entity_type,
                entity_id=entity_id,
                action=action,
                user_context=user_context
            )

            return result
        return wrapper
    return decorator

# Usage
@audit_logged(entity_type="POLICY", action="VIEW")
async def get_policy(policy_id: str):
    return await policy_repository.find_by_id(policy_id)
```

---

## Audit Log Queries

### Common Audit Queries

```sql
-- Find all actions by a specific user
SELECT * FROM audit_log
WHERE user_id = 'user-12345'
ORDER BY timestamp DESC
LIMIT 100;

-- Find all changes to a specific policy
SELECT * FROM audit_log
WHERE entity_type = 'POLICY'
AND entity_id = 'policy-67890'
ORDER BY timestamp DESC;

-- Find all sensitive data access in last 24 hours
SELECT * FROM audit_log
WHERE action = 'VIEW'
AND entity_type IN ('SSN', 'MEDICAL_RECORD', 'BANK_ACCOUNT')
AND timestamp > NOW() - INTERVAL '24 hours';

-- Find failed login attempts
SELECT user_id, ip_address, COUNT(*) as attempt_count
FROM audit_log
WHERE action = 'LOGIN_FAILED'
AND timestamp > NOW() - INTERVAL '1 hour'
GROUP BY user_id, ip_address
HAVING COUNT(*) > 3;

-- Audit trail for compliance report
SELECT
    timestamp,
    user_name,
    user_role,
    action,
    entity_type,
    entity_reference,
    field_changes
FROM audit_log
WHERE timestamp BETWEEN '2024-01-01' AND '2024-03-31'
AND entity_type = 'CLAIM'
ORDER BY timestamp;
```

---

## Audit Log Security

### Immutability

- Audit logs should be **write-once, read-many**
- No UPDATE or DELETE operations allowed
- Use append-only storage or write-ahead logs

### Integrity

- Hash each log entry and chain to previous entry
- Store hashes separately for verification
- Use digital signatures for critical events

### Retention

- Follow regulatory requirements (typically 7+ years)
- Archive to cold storage after active period
- Ensure searchability of archived logs

---

## Navigation

[Back to Lesson 07](../README.md) | [Previous: Regulatory Compliance](../04-regulatory-compliance/README.md) | [Next: Security Best Practices](../06-security-best-practices/README.md)
