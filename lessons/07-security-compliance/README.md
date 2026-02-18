# Lesson 07: Security & Compliance

## Learning Objectives

By the end of this lesson, you will be able to:
- Implement security architecture for insurance systems
- Understand regulatory compliance requirements
- Design data protection and privacy controls
- Implement audit and access control mechanisms

---

## 7.1 Security Architecture

### Defense in Depth

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SECURITY LAYERS                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      PERIMETER SECURITY                                │  │
│  │  - WAF (Web Application Firewall)                                      │  │
│  │  - DDoS Protection (CloudFlare, AWS Shield)                           │  │
│  │  - API Gateway Security                                                │  │
│  │  - Network Firewall                                                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                         │
│                                    ↓                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                   AUTHENTICATION & AUTHORIZATION                       │  │
│  │  - OAuth 2.0 / OpenID Connect                                         │  │
│  │  - Multi-Factor Authentication (MFA)                                  │  │
│  │  - Role-Based Access Control (RBAC)                                   │  │
│  │  - API Key Management                                                 │  │
│  │  - JWT Token Validation                                               │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                         │
│                                    ↓                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      APPLICATION SECURITY                              │  │
│  │  - Input Validation & Sanitization                                    │  │
│  │  - Output Encoding                                                    │  │
│  │  - SQL Injection Prevention (Parameterized Queries)                   │  │
│  │  - XSS Prevention (Content Security Policy)                           │  │
│  │  - CSRF Protection                                                    │  │
│  │  - Secure Headers                                                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                         │
│                                    ↓                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        DATA SECURITY                                   │  │
│  │  - Encryption at Rest (AES-256)                                       │  │
│  │  - Encryption in Transit (TLS 1.3)                                    │  │
│  │  - Key Management (HSM / Cloud KMS)                                   │  │
│  │  - Data Masking                                                       │  │
│  │  - Tokenization (PCI-DSS)                                             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Authentication Flow

```
┌────────────┐      ┌────────────┐      ┌────────────┐      ┌────────────┐
│   Client   │      │    API     │      │    Auth    │      │   User     │
│            │      │  Gateway   │      │  Service   │      │ Database   │
└─────┬──────┘      └─────┬──────┘      └─────┬──────┘      └─────┬──────┘
      │                   │                   │                   │
      │  1. Login Request │                   │                   │
      │  (username/pwd)   │                   │                   │
      │ ─────────────────→│                   │                   │
      │                   │  2. Authenticate  │                   │
      │                   │ ─────────────────→│                   │
      │                   │                   │  3. Validate User │
      │                   │                   │ ─────────────────→│
      │                   │                   │                   │
      │                   │                   │  4. User Data     │
      │                   │                   │ ←─────────────────│
      │                   │                   │                   │
      │                   │                   │  5. Check MFA     │
      │                   │                   │  (if enabled)     │
      │                   │                   │                   │
      │                   │  6. JWT + Refresh │                   │
      │                   │ ←─────────────────│                   │
      │  7. Tokens        │                   │                   │
      │ ←─────────────────│                   │                   │
      │                   │                   │                   │
      │  8. API Request   │                   │                   │
      │  + Bearer Token   │                   │                   │
      │ ─────────────────→│                   │                   │
      │                   │  9. Verify JWT    │                   │
      │                   │ ─────────────────→│                   │
      │                   │                   │                   │
      │                   │  10. Token Valid  │                   │
      │                   │ ←─────────────────│                   │
      │                   │                   │                   │
      │  11. Response     │                   │                   │
      │ ←─────────────────│                   │                   │
```

---

## 7.2 Role-Based Access Control (RBAC)

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                       ROLE HIERARCHY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │  SYSTEM_ADMIN   │                          │
│                    │  (Full Access)  │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│         ┌───────────────────┼───────────────────┐               │
│         ↓                   ↓                   ↓               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  MANAGER    │    │ UNDERWRITER │    │   CLAIMS    │         │
│  │             │    │   MANAGER   │    │   MANAGER   │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         ↓                  ↓                  ↓                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   AGENT     │    │ UNDERWRITER │    │  ADJUSTER   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                  │
│  CUSTOMER-FACING ROLES:                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  CUSTOMER   │    │   BROKER    │    │  PARTNER    │         │
│  │  (Self-svc) │    │  (External) │    │    API      │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Permission Matrix

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

---

## 7.3 Data Protection

### Sensitive Data Classification

| Classification | Examples | Protection Requirements |
|---------------|----------|------------------------|
| **Critical** | SSN, National ID, Bank Account | Encrypted + Tokenized + Access Logged |
| **Highly Sensitive** | Medical Records, Credit Score | Encrypted + Access Controlled |
| **Sensitive** | DOB, Address, Phone | Encrypted + Role-Based Access |
| **PII** | Name, Email | Encrypted in Transit |
| **Internal** | Policy Details | Standard Access Control |
| **Public** | Product Catalog | No Restrictions |

### Data Masking Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                       DATA MASKING                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FIELD              ORIGINAL              MASKED                │
│  ────────────────────────────────────────────────────────────   │
│  SSN                123-45-6789          XXX-XX-6789            │
│  Credit Card        4532-1234-5678-9012  XXXX-XXXX-XXXX-9012   │
│  Bank Account       1234567890           XXXXXX7890             │
│  Email              john.doe@email.com   j***e@email.com        │
│  Phone              +1-555-123-4567      +1-555-XXX-4567        │
│  Address            123 Main Street      123 M*** S*****        │
│  Date of Birth      1985-03-15           XXXX-XX-15             │
│                                                                  │
│  MASKING BY ROLE:                                               │
│  ───────────────                                                │
│  Customer Support:  Partial masking (last 4 digits visible)     │
│  Agent:             Partial masking (with consent)              │
│  Underwriter:       Full access (logged)                        │
│  System Admin:      Full access (logged + approved)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Encryption Strategy

```yaml
# Encryption Configuration
encryption:
  at_rest:
    algorithm: AES-256-GCM
    key_management: AWS_KMS  # or Azure Key Vault, HashiCorp Vault
    key_rotation: 90_days

  in_transit:
    protocol: TLS_1.3
    cipher_suites:
      - TLS_AES_256_GCM_SHA384
      - TLS_CHACHA20_POLY1305_SHA256
    certificate_validity: 365_days

  field_level:
    enabled: true
    fields:
      - ssn
      - national_id
      - bank_account
      - credit_card
    algorithm: AES-256-GCM

  tokenization:
    provider: payment_gateway
    fields:
      - credit_card_number
      - cvv
```

---

## 7.4 Regulatory Compliance

### Key Regulations

| Regulation | Region | Key Requirements |
|------------|--------|------------------|
| **GDPR** | EU | Data protection, consent, right to erasure |
| **CCPA** | California | Consumer privacy rights, opt-out |
| **HIPAA** | US (Health) | PHI protection, breach notification |
| **PCI-DSS** | Global (Payments) | Card data security |
| **SOX** | US | Financial reporting controls |
| **Solvency II** | EU | Capital adequacy, risk management |
| **IFRS 17** | Global | Insurance contract accounting |
| **IDD** | EU | Insurance distribution standards |
| **NAIC** | US | State insurance regulations |

### GDPR Compliance Framework

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          GDPR COMPLIANCE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DATA SUBJECT RIGHTS:                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐           │    │
│  │  │  Right to │ │  Right to │ │  Right to │ │  Right to │           │    │
│  │  │  Access   │ │Rectification│ │  Erasure  │ │ Portability│          │    │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘           │    │
│  │                                                                      │    │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐           │    │
│  │  │  Right to │ │  Right to │ │  Right to │ │Automated  │           │    │
│  │  │  Object   │ │ Restrict  │ │Be Informed│ │ Decision  │           │    │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘           │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  LAWFUL BASIS:                                                               │
│  - Contract performance (policy servicing)                                  │
│  - Legal obligation (regulatory reporting)                                   │
│  - Legitimate interest (fraud prevention)                                    │
│  - Consent (marketing communications)                                        │
│                                                                              │
│  IMPLEMENTATION:                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Consent   │  │    Data     │  │   Privacy   │  │   Breach    │        │
│  │ Management  │  │  Inventory  │  │   Notices   │  │Notification │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Data Retention Policy

| Data Type | Retention Period | Legal Basis |
|-----------|-----------------|-------------|
| Policy Documents | Policy term + 10 years | Legal requirement |
| Claim Records | Settlement + 10 years | Litigation |
| Customer Data | Relationship + 7 years | Tax/Audit |
| Payment Records | 7 years | Financial regulations |
| Communication Logs | 3 years | Service quality |
| Audit Logs | 7 years | Compliance |
| Marketing Consent | Until withdrawn | GDPR |

---

## 7.5 Audit Trail

### Audit Log Schema

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

### Audit Events

| Action | Description | Logged Data |
|--------|-------------|-------------|
| VIEW | Data accessed | User, entity, fields viewed |
| CREATE | New record | User, entity, all values |
| UPDATE | Record modified | User, entity, old/new values |
| DELETE | Record removed | User, entity, reason |
| EXPORT | Data exported | User, criteria, record count |
| LOGIN | User login | User, IP, device, MFA used |
| LOGOUT | User logout | User, session duration |
| PERMISSION_CHANGE | Access modified | User, old/new permissions |

---

## 7.6 Security Best Practices

### OWASP Top 10 Mitigations

| Vulnerability | Mitigation |
|--------------|------------|
| **Injection** | Parameterized queries, input validation |
| **Broken Authentication** | MFA, session management, password policies |
| **Sensitive Data Exposure** | Encryption, masking, access controls |
| **XXE** | Disable external entities, use JSON |
| **Broken Access Control** | RBAC, principle of least privilege |
| **Security Misconfiguration** | Hardening, security headers |
| **XSS** | Output encoding, CSP |
| **Insecure Deserialization** | Input validation, integrity checks |
| **Vulnerable Components** | Dependency scanning, updates |
| **Insufficient Logging** | Comprehensive audit logging |

### Security Headers

```
# Recommended HTTP Security Headers
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

---

## Key Takeaways

1. **Defense in depth** provides multiple security layers
2. **RBAC** ensures appropriate access based on roles
3. **Data classification** determines protection requirements
4. **Regulatory compliance** is mandatory for insurance
5. **Audit trails** enable accountability and forensics

---

## Exercises

1. Design RBAC permissions for a new "Compliance Officer" role
2. Create a data retention policy for health insurance claims
3. Implement GDPR data subject request handling workflow

---

[← Previous Lesson](../06-api-design/README.md) | [Next Lesson: Development Practices →](../08-development-practices/README.md)
