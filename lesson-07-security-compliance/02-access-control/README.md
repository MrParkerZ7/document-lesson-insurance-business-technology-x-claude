# 7.2 Access Control

## Overview

Role-Based Access Control (RBAC) is essential for insurance systems where different stakeholders require varying levels of access to sensitive data. This sub-lesson covers role hierarchies, permission structures, and access control implementation.

---

## Role Hierarchy

Insurance systems typically have a hierarchical role structure that reflects organizational responsibilities:

```
+-----------------------------------------------------------------+
|                       ROLE HIERARCHY                             |
+-----------------------------------------------------------------+
|                                                                  |
|                    +-----------------+                          |
|                    |  SYSTEM_ADMIN   |                          |
|                    |  (Full Access)  |                          |
|                    +--------+--------+                          |
|                             |                                    |
|         +-------------------+-------------------+               |
|         v                   v                   v               |
|  +-------------+    +-------------+    +-------------+         |
|  |  MANAGER    |    | UNDERWRITER |    |   CLAIMS    |         |
|  |             |    |   MANAGER   |    |   MANAGER   |         |
|  +------+------+    +------+------+    +------+------+         |
|         |                  |                  |                 |
|         v                  v                  v                 |
|  +-------------+    +-------------+    +-------------+         |
|  |   AGENT     |    | UNDERWRITER |    |  ADJUSTER   |         |
|  +-------------+    +-------------+    +-------------+         |
|                                                                  |
|  CUSTOMER-FACING ROLES:                                         |
|  +-------------+    +-------------+    +-------------+         |
|  |  CUSTOMER   |    |   BROKER    |    |  PARTNER    |         |
|  |  (Self-svc) |    |  (External) |    |    API      |         |
|  +-------------+    +-------------+    +-------------+         |
|                                                                  |
+-----------------------------------------------------------------+
```

### Role Descriptions

| Role | Description | Scope |
|------|-------------|-------|
| **SYSTEM_ADMIN** | Full system access, configuration management | Global |
| **MANAGER** | Department management, reporting, approvals | Department |
| **UNDERWRITER_MANAGER** | Underwriting team lead, policy approvals | Underwriting |
| **CLAIMS_MANAGER** | Claims team lead, settlement approvals | Claims |
| **AGENT** | Customer interactions, quote creation | Assigned customers |
| **UNDERWRITER** | Risk assessment, policy pricing | Assigned policies |
| **ADJUSTER** | Claims investigation and processing | Assigned claims |
| **CUSTOMER** | Self-service access to own data | Own data only |
| **BROKER** | External partner with limited access | Partner scope |
| **PARTNER_API** | System-to-system integration | API scope |

---

## Permission Matrix

The following matrix defines CRUD (Create, Read, Update, Delete) permissions for each role:

| Resource | Customer | Agent | Underwriter | Claims Adjuster | Manager | Admin |
|----------|----------|-------|-------------|-----------------|---------|-------|
| **Own Profile** | CRUD | CRUD | CRUD | CRUD | CRUD | CRUD |
| **Other Profiles** | - | R | R | R | CRUD | CRUD |
| **Quotes** | CR | CRUD | CRUD | R | CRUD | CRUD |
| **Policies (Own)** | R | CRUD | CRUD | R | CRUD | CRUD |
| **Policies (All)** | - | R | CRUD | R | CRUD | CRUD |
| **Claims (Own)** | CR | R | R | CRUD | CRUD | CRUD |
| **Claims (All)** | - | R | R | CRUD | CRUD | CRUD |
| **Payments** | R | R | - | R | CRUD | CRUD |
| **Products** | R | R | R | R | R | CRUD |
| **Reports** | - | R | R | R | CRUD | CRUD |
| **Audit Logs** | - | - | - | - | R | CRUD |
| **System Config** | - | - | - | - | - | CRUD |

### Permission Legend
- **C** = Create
- **R** = Read
- **U** = Update
- **D** = Delete
- **-** = No access

---

## RBAC Implementation

### Database Schema

```sql
-- Roles table
CREATE TABLE roles (
    role_id         UUID PRIMARY KEY,
    role_name       VARCHAR(50) NOT NULL UNIQUE,
    description     TEXT,
    parent_role_id  UUID REFERENCES roles(role_id),
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Permissions table
CREATE TABLE permissions (
    permission_id   UUID PRIMARY KEY,
    resource        VARCHAR(100) NOT NULL,
    action          VARCHAR(20) NOT NULL,
    scope           VARCHAR(50),
    description     TEXT,
    UNIQUE(resource, action, scope)
);

-- Role-Permission mapping
CREATE TABLE role_permissions (
    role_id         UUID REFERENCES roles(role_id),
    permission_id   UUID REFERENCES permissions(permission_id),
    PRIMARY KEY (role_id, permission_id)
);

-- User-Role mapping
CREATE TABLE user_roles (
    user_id         UUID NOT NULL,
    role_id         UUID REFERENCES roles(role_id),
    assigned_by     UUID,
    assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at      TIMESTAMP,
    PRIMARY KEY (user_id, role_id)
);
```

### Permission Check Example

```python
def check_permission(user_id: str, resource: str, action: str) -> bool:
    """
    Check if user has permission to perform action on resource
    """
    user_roles = get_user_roles(user_id)

    for role in user_roles:
        permissions = get_role_permissions(role.role_id)
        for permission in permissions:
            if permission.resource == resource and permission.action == action:
                return True

        # Check parent roles (inheritance)
        if role.parent_role_id:
            if check_role_permission(role.parent_role_id, resource, action):
                return True

    return False
```

---

## Access Control Patterns

### Attribute-Based Access Control (ABAC)

For more granular control, combine RBAC with attributes:

```python
def can_access_claim(user: User, claim: Claim) -> bool:
    """
    Check access based on role and attributes
    """
    # System admin has full access
    if user.has_role('SYSTEM_ADMIN'):
        return True

    # Customer can access own claims
    if user.has_role('CUSTOMER'):
        return claim.customer_id == user.customer_id

    # Adjuster can access assigned claims
    if user.has_role('ADJUSTER'):
        return claim.assigned_adjuster_id == user.id

    # Claims manager can access department claims
    if user.has_role('CLAIMS_MANAGER'):
        return claim.department_id == user.department_id

    return False
```

### Resource-Level Permissions

| Pattern | Description | Example |
|---------|-------------|---------|
| **Own Data** | User can only access their own records | Customer viewing own policies |
| **Assigned Data** | User can access records assigned to them | Adjuster viewing assigned claims |
| **Department Data** | User can access department records | Manager viewing team reports |
| **Global Data** | User can access all records | Admin viewing audit logs |

---

## Navigation

[Back to Lesson 07](../README.md) | [Previous: Security Architecture](../01-security-architecture/README.md) | [Next: Data Protection](../03-data-protection/README.md)
