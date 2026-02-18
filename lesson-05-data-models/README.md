# Lesson 05: Data Models & Database Design

## Learning Objectives

By the end of this lesson, you will be able to:
- Design core entity relationships for insurance systems
- Create database schemas for policy, claims, and customer data
- Implement state machines for policy and claims workflows
- Apply data modeling best practices

---

## Sub-Lessons

### [01 - Entity Relationships](./01-entity-relationships/README.md)
Core entity relationship diagrams and relationship matrix for insurance domain entities including Customer, Policy, Product, Claim, and related entities.

### [02 - Database Schemas](./02-database-schemas/README.md)
Comprehensive database schemas for Customer, Policy, Claim, and Product entities with proper constraints, indexes, and audit fields.

### [03 - State Machines](./03-state-machines/README.md)
Policy and Claim status state machines defining valid transitions and business rules for lifecycle management.

---

## Data Modeling Best Practices

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Tables | snake_case, singular | `policy`, `claim_payment` |
| Columns | snake_case | `customer_id`, `effective_date` |
| Primary Keys | `{table}_id` | `policy_id`, `claim_id` |
| Foreign Keys | `{referenced_table}_id` | `customer_id`, `product_id` |
| Indexes | `idx_{table}_{columns}` | `idx_policy_customer` |
| Constraints | `chk_{table}_{column}` | `chk_policy_status` |

### Common Patterns

```
+---------------------------------------------------------------------+
|                    DATA MODELING PATTERNS                            |
+---------------------------------------------------------------------+
|                                                                      |
|  1. TEMPORAL DATA                                                    |
|     - Use effective_from/effective_to for validity periods           |
|     - Never delete, only soft-delete with is_active flag             |
|     - Track created_at/updated_at for all records                    |
|                                                                      |
|  2. VERSIONING                                                       |
|     - Version column for optimistic locking                          |
|     - Parent reference for version history                           |
|     - Snapshot tables for point-in-time queries                      |
|                                                                      |
|  3. STATUS TRACKING                                                  |
|     - Status column with enum constraints                            |
|     - Sub_status for detailed state                                  |
|     - Status history table for transitions                           |
|                                                                      |
|  4. AMOUNTS                                                          |
|     - Use DECIMAL(18,2) for money                                    |
|     - Always store currency code                                     |
|     - Separate gross/net/tax amounts                                 |
|                                                                      |
|  5. REFERENCES                                                       |
|     - UUID for primary keys                                          |
|     - Business keys (policy_number) as UNIQUE                        |
|     - Soft references for external systems                           |
|                                                                      |
+---------------------------------------------------------------------+
```

---

## Audit Trail

All insurance systems require comprehensive audit logging for compliance and debugging:

```sql
-- Audit Log Table
CREATE TABLE audit_log (
    audit_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Entity Information
    entity_type         VARCHAR(50) NOT NULL,          -- POLICY, CLAIM, CUSTOMER
    entity_id           UUID NOT NULL,
    entity_reference    VARCHAR(100),                  -- Policy number, claim number, etc.

    -- Action Information
    action              VARCHAR(20) NOT NULL,          -- CREATE, UPDATE, DELETE, VIEW
    action_category     VARCHAR(50),                   -- STATUS_CHANGE, DATA_UPDATE, DOCUMENT_UPLOAD

    -- Change Details
    field_name          VARCHAR(100),
    old_value           TEXT,
    new_value           TEXT,
    change_summary      TEXT,

    -- User Information
    user_id             VARCHAR(100) NOT NULL,
    user_name           VARCHAR(255),
    user_role           VARCHAR(50),

    -- Session Information
    ip_address          VARCHAR(45),
    user_agent          TEXT,
    session_id          VARCHAR(100),

    -- Correlation
    correlation_id      VARCHAR(100),
    request_id          VARCHAR(100),

    -- Timestamp
    timestamp           TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for efficient querying
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_user ON audit_log(user_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp);
CREATE INDEX idx_audit_action ON audit_log(action);
CREATE INDEX idx_audit_correlation ON audit_log(correlation_id);
```

---

## Key Takeaways

1. **Entity relationships** form the backbone of insurance data models
2. **Status fields** with proper constraints ensure data integrity
3. **State machines** define valid transitions for policies and claims
4. **Audit trails** are essential for compliance and debugging
5. **Naming conventions** improve code readability and maintenance

---

## Exercises

1. Design a schema for beneficiary management in life insurance
2. Create a premium payment schedule table with installments
3. Implement a claim reserve history tracking table

---

[Previous Lesson](../lesson-04-system-architecture/README.md) | [Next Lesson](../lesson-06-api-design/README.md)
