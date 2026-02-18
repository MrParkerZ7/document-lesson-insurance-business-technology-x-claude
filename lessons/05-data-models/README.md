# Lesson 05: Data Models & Database Design

## Learning Objectives

By the end of this lesson, you will be able to:
- Design core entity relationships for insurance systems
- Create database schemas for policy, claims, and customer data
- Implement state machines for policy and claims workflows
- Apply data modeling best practices

---

## 5.1 Core Entity Relationships

### Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ENTITY RELATIONSHIP DIAGRAM                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐        │
│  │   CUSTOMER   │ 1────M  │    POLICY    │ M────1  │   PRODUCT    │        │
│  └──────────────┘         └──────────────┘         └──────────────┘        │
│        │                        │                        │                  │
│        │ 1                      │ 1                      │ 1                │
│        │                        │                        │                  │
│        M                        M                        M                  │
│  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐        │
│  │   ADDRESS    │         │    CLAIM     │         │   COVERAGE   │        │
│  └──────────────┘         └──────────────┘         └──────────────┘        │
│                                 │                        │                  │
│                                 │ 1                      │ 1                │
│                                 │                        │                  │
│                                 M                        M                  │
│                           ┌──────────────┐         ┌──────────────┐        │
│                           │CLAIM_PAYMENT │         │    RIDER     │        │
│                           └──────────────┘         └──────────────┘        │
│                                                                              │
│  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐        │
│  │    AGENT     │ 1────M  │    QUOTE     │ M────1  │   CUSTOMER   │        │
│  └──────────────┘         └──────────────┘         └──────────────┘        │
│                                 │                                           │
│                                 │ 1                                         │
│                                 │                                           │
│                                 M                                           │
│                           ┌──────────────┐                                  │
│                           │  QUOTE_ITEM  │                                  │
│                           └──────────────┘                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Relationship Matrix

| Entity A | Relationship | Entity B | Description |
|----------|--------------|----------|-------------|
| Customer | 1:M | Policy | Customer has many policies |
| Customer | 1:M | Address | Customer has multiple addresses |
| Customer | 1:M | Quote | Customer has many quotes |
| Policy | M:1 | Product | Policy belongs to product |
| Policy | 1:M | Claim | Policy has many claims |
| Policy | 1:M | Endorsement | Policy has endorsement history |
| Policy | 1:M | Premium | Policy has premium schedule |
| Claim | 1:M | ClaimPayment | Claim has payment history |
| Product | 1:M | Coverage | Product has coverages |
| Product | 1:M | Rider | Product has optional riders |
| Agent | 1:M | Policy | Agent sells policies |
| Quote | 1:M | QuoteItem | Quote has line items |

---

## 5.2 Database Schemas

### Customer Schema

```sql
-- Customer Table
CREATE TABLE customer (
    customer_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_number     VARCHAR(20) UNIQUE NOT NULL,
    customer_type       VARCHAR(20) NOT NULL,       -- INDIVIDUAL, CORPORATE

    -- Personal Details (Individual)
    first_name          VARCHAR(100),
    middle_name         VARCHAR(100),
    last_name           VARCHAR(100),
    date_of_birth       DATE,
    gender              VARCHAR(10),                -- MALE, FEMALE, OTHER
    marital_status      VARCHAR(20),
    nationality         VARCHAR(50),

    -- Corporate Details
    company_name        VARCHAR(255),
    registration_number VARCHAR(100),
    industry            VARCHAR(100),

    -- Contact Information
    email               VARCHAR(255) NOT NULL,
    phone_primary       VARCHAR(20),
    phone_secondary     VARCHAR(20),

    -- Identity Documents
    id_type             VARCHAR(50),                -- PASSPORT, NATIONAL_ID, DRIVING_LICENSE
    id_number           VARCHAR(100),
    id_expiry_date      DATE,

    -- KYC Status
    kyc_status          VARCHAR(20) DEFAULT 'PENDING',  -- PENDING, VERIFIED, REJECTED
    kyc_verified_at     TIMESTAMP,
    kyc_verified_by     VARCHAR(100),

    -- Risk Assessment
    risk_score          DECIMAL(5,2),
    risk_category       VARCHAR(20),                -- LOW, MEDIUM, HIGH

    -- Audit Fields
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100),
    updated_by          VARCHAR(100),
    is_active           BOOLEAN DEFAULT TRUE,

    -- Constraints
    CONSTRAINT chk_customer_type CHECK (customer_type IN ('INDIVIDUAL', 'CORPORATE')),
    CONSTRAINT chk_kyc_status CHECK (kyc_status IN ('PENDING', 'VERIFIED', 'REJECTED', 'EXPIRED'))
);

-- Customer Address Table
CREATE TABLE customer_address (
    address_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id         UUID NOT NULL REFERENCES customer(customer_id),
    address_type        VARCHAR(20) NOT NULL,       -- HOME, OFFICE, MAILING
    address_line_1      VARCHAR(255) NOT NULL,
    address_line_2      VARCHAR(255),
    city                VARCHAR(100) NOT NULL,
    state               VARCHAR(100),
    postal_code         VARCHAR(20),
    country             VARCHAR(100) NOT NULL,
    is_primary          BOOLEAN DEFAULT FALSE,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_customer_email ON customer(email);
CREATE INDEX idx_customer_phone ON customer(phone_primary);
CREATE INDEX idx_customer_id_number ON customer(id_type, id_number);
CREATE INDEX idx_customer_kyc_status ON customer(kyc_status);
```

### Policy Schema

```sql
-- Policy Table
CREATE TABLE policy (
    policy_id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    policy_number       VARCHAR(50) UNIQUE NOT NULL,
    quote_id            UUID REFERENCES quote(quote_id),
    customer_id         UUID NOT NULL REFERENCES customer(customer_id),
    product_id          UUID NOT NULL REFERENCES product(product_id),
    plan_id             UUID REFERENCES plan(plan_id),
    agent_id            UUID REFERENCES agent(agent_id),

    -- Policy Status
    status              VARCHAR(20) NOT NULL DEFAULT 'DRAFT',
    sub_status          VARCHAR(50),

    -- Policy Dates
    proposal_date       DATE,
    issue_date          DATE,
    effective_date      DATE NOT NULL,
    expiry_date         DATE NOT NULL,
    cancellation_date   DATE,

    -- Coverage Details
    sum_insured         DECIMAL(18,2) NOT NULL,
    currency            VARCHAR(3) DEFAULT 'USD',
    deductible          DECIMAL(18,2) DEFAULT 0,

    -- Premium Details
    gross_premium       DECIMAL(18,2) NOT NULL,
    net_premium         DECIMAL(18,2) NOT NULL,
    tax_amount          DECIMAL(18,2) DEFAULT 0,
    total_premium       DECIMAL(18,2) NOT NULL,
    premium_frequency   VARCHAR(20) DEFAULT 'ANNUAL',  -- MONTHLY, QUARTERLY, SEMI_ANNUAL, ANNUAL

    -- Payment
    payment_method      VARCHAR(50),
    payment_status      VARCHAR(20) DEFAULT 'PENDING',

    -- Underwriting
    underwriting_status VARCHAR(20) DEFAULT 'PENDING',
    underwriting_notes  TEXT,
    risk_score          DECIMAL(5,2),

    -- Renewal
    is_renewal          BOOLEAN DEFAULT FALSE,
    previous_policy_id  UUID REFERENCES policy(policy_id),
    renewal_count       INT DEFAULT 0,
    auto_renewal        BOOLEAN DEFAULT TRUE,

    -- Cancellation
    cancellation_reason VARCHAR(255),
    cancellation_type   VARCHAR(20),                   -- FLAT, PRO_RATA, SHORT_RATE
    refund_amount       DECIMAL(18,2),

    -- Version Control
    version             INT DEFAULT 1,
    parent_policy_id    UUID REFERENCES policy(policy_id),

    -- Audit Fields
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100),
    updated_by          VARCHAR(100),

    -- Constraints
    CONSTRAINT chk_policy_status CHECK (status IN ('DRAFT', 'QUOTED', 'PROPOSED', 'ISSUED', 'ACTIVE', 'LAPSED', 'CANCELLED', 'EXPIRED', 'MATURED')),
    CONSTRAINT chk_dates CHECK (expiry_date > effective_date)
);

-- Policy Coverage Table
CREATE TABLE policy_coverage (
    coverage_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    policy_id           UUID NOT NULL REFERENCES policy(policy_id),
    coverage_code       VARCHAR(50) NOT NULL,
    coverage_name       VARCHAR(255) NOT NULL,
    coverage_type       VARCHAR(50),                   -- STANDARD, OPTIONAL
    sum_insured         DECIMAL(18,2),
    limit_type          VARCHAR(20),                   -- PER_OCCURRENCE, AGGREGATE, LIFETIME
    deductible          DECIMAL(18,2) DEFAULT 0,
    deductible_type     VARCHAR(20),                   -- FLAT, PERCENTAGE
    waiting_period_days INT DEFAULT 0,
    premium             DECIMAL(18,2) NOT NULL,
    is_active           BOOLEAN DEFAULT TRUE,
    effective_date      DATE,
    expiry_date         DATE,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Policy Endorsement Table
CREATE TABLE policy_endorsement (
    endorsement_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    policy_id           UUID NOT NULL REFERENCES policy(policy_id),
    endorsement_number  VARCHAR(50) UNIQUE NOT NULL,
    endorsement_type    VARCHAR(50) NOT NULL,          -- COVERAGE_CHANGE, SUM_INSURED_CHANGE, BENEFICIARY_CHANGE
    effective_date      DATE NOT NULL,
    description         TEXT,
    premium_adjustment  DECIMAL(18,2) DEFAULT 0,
    status              VARCHAR(20) DEFAULT 'PENDING',
    approved_by         VARCHAR(100),
    approved_at         TIMESTAMP,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100)
);

-- Indexes
CREATE INDEX idx_policy_customer ON policy(customer_id);
CREATE INDEX idx_policy_product ON policy(product_id);
CREATE INDEX idx_policy_status ON policy(status);
CREATE INDEX idx_policy_dates ON policy(effective_date, expiry_date);
CREATE INDEX idx_policy_agent ON policy(agent_id);
```

### Claim Schema

```sql
-- Claim Table
CREATE TABLE claim (
    claim_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    claim_number        VARCHAR(50) UNIQUE NOT NULL,
    policy_id           UUID NOT NULL REFERENCES policy(policy_id),
    customer_id         UUID NOT NULL REFERENCES customer(customer_id),
    coverage_id         UUID REFERENCES policy_coverage(coverage_id),

    -- Loss Details
    loss_date           DATE NOT NULL,
    loss_time           TIME,
    notification_date   DATE NOT NULL,
    loss_location       TEXT,
    loss_description    TEXT NOT NULL,
    loss_cause          VARCHAR(100),

    -- Claim Classification
    claim_type          VARCHAR(50) NOT NULL,
    claim_category      VARCHAR(50),
    peril_code          VARCHAR(50),

    -- Status
    status              VARCHAR(20) NOT NULL DEFAULT 'OPEN',
    sub_status          VARCHAR(50),
    priority            VARCHAR(10) DEFAULT 'MEDIUM',  -- LOW, MEDIUM, HIGH, CRITICAL

    -- Amounts
    claimed_amount      DECIMAL(18,2) NOT NULL,
    assessed_amount     DECIMAL(18,2),
    approved_amount     DECIMAL(18,2),
    deductible_amount   DECIMAL(18,2) DEFAULT 0,
    settled_amount      DECIMAL(18,2),
    currency            VARCHAR(3) DEFAULT 'USD',

    -- Reserve
    reserve_amount      DECIMAL(18,2),
    reserve_updated_at  TIMESTAMP,

    -- Assignment
    adjuster_id         UUID,
    adjuster_name       VARCHAR(255),
    assigned_at         TIMESTAMP,

    -- Fraud Detection
    fraud_score         DECIMAL(5,2),
    fraud_flag          BOOLEAN DEFAULT FALSE,
    fraud_indicators    JSONB,
    siu_referral        BOOLEAN DEFAULT FALSE,

    -- Settlement
    settlement_date     DATE,
    settlement_type     VARCHAR(50),                   -- CASH, REPAIR, REPLACEMENT
    settlement_notes    TEXT,

    -- Dates
    first_contact_date  DATE,
    target_closure_date DATE,
    actual_closure_date DATE,

    -- External References
    police_report_number VARCHAR(100),
    hospital_name       VARCHAR(255),

    -- Audit Fields
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100),
    updated_by          VARCHAR(100),

    -- Constraints
    CONSTRAINT chk_claim_status CHECK (status IN ('DRAFT', 'OPEN', 'ASSIGNED', 'UNDER_REVIEW', 'ON_HOLD', 'APPROVED', 'PARTIALLY_APPROVED', 'REJECTED', 'SETTLED', 'CLOSED', 'REOPENED')),
    CONSTRAINT chk_claim_priority CHECK (priority IN ('LOW', 'MEDIUM', 'HIGH', 'CRITICAL'))
);

-- Claim Document Table
CREATE TABLE claim_document (
    document_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    claim_id            UUID NOT NULL REFERENCES claim(claim_id),
    document_type       VARCHAR(50) NOT NULL,          -- CLAIM_FORM, PHOTO, INVOICE, REPORT, ID_PROOF
    document_name       VARCHAR(255) NOT NULL,
    file_path           VARCHAR(500) NOT NULL,
    file_size           BIGINT,
    mime_type           VARCHAR(100),
    uploaded_by         VARCHAR(100),
    uploaded_at         TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    verified            BOOLEAN DEFAULT FALSE,
    verified_by         VARCHAR(100),
    verified_at         TIMESTAMP
);

-- Claim Payment Table
CREATE TABLE claim_payment (
    payment_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    claim_id            UUID NOT NULL REFERENCES claim(claim_id),
    payment_number      VARCHAR(50) UNIQUE NOT NULL,
    payment_type        VARCHAR(50) NOT NULL,          -- INTERIM, FINAL, EX_GRATIA
    payee_name          VARCHAR(255) NOT NULL,
    payee_type          VARCHAR(50),                   -- INSURED, PROVIDER, THIRD_PARTY
    amount              DECIMAL(18,2) NOT NULL,
    currency            VARCHAR(3) DEFAULT 'USD',
    payment_method      VARCHAR(50),                   -- BANK_TRANSFER, CHECK, CASH
    bank_account        VARCHAR(100),
    payment_date        DATE,
    status              VARCHAR(20) DEFAULT 'PENDING', -- PENDING, APPROVED, PROCESSED, FAILED
    reference_number    VARCHAR(100),
    notes               TEXT,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100)
);

-- Indexes
CREATE INDEX idx_claim_policy ON claim(policy_id);
CREATE INDEX idx_claim_customer ON claim(customer_id);
CREATE INDEX idx_claim_status ON claim(status);
CREATE INDEX idx_claim_loss_date ON claim(loss_date);
CREATE INDEX idx_claim_adjuster ON claim(adjuster_id);
CREATE INDEX idx_claim_fraud ON claim(fraud_flag, fraud_score);
```

### Product Schema

```sql
-- Product Table
CREATE TABLE product (
    product_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_code        VARCHAR(50) UNIQUE NOT NULL,
    product_name        VARCHAR(255) NOT NULL,
    product_type        VARCHAR(50) NOT NULL,          -- LIFE, HEALTH, MOTOR, PROPERTY, LIABILITY
    line_of_business    VARCHAR(100),
    description         TEXT,

    -- Status
    status              VARCHAR(20) DEFAULT 'DRAFT',   -- DRAFT, ACTIVE, INACTIVE, DISCONTINUED
    effective_from      DATE,
    effective_to        DATE,

    -- Eligibility
    min_entry_age       INT,
    max_entry_age       INT,
    min_sum_insured     DECIMAL(18,2),
    max_sum_insured     DECIMAL(18,2),

    -- Policy Terms
    min_policy_term     INT,                           -- In months
    max_policy_term     INT,
    default_policy_term INT,

    -- Waiting Periods
    waiting_period_days INT DEFAULT 0,

    -- Underwriting
    requires_medical    BOOLEAN DEFAULT FALSE,
    auto_uw_limit       DECIMAL(18,2),

    -- Commission
    first_year_commission_rate  DECIMAL(5,2),
    renewal_commission_rate     DECIMAL(5,2),

    -- Audit Fields
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          VARCHAR(100),
    version             INT DEFAULT 1
);

-- Product Plan Table
CREATE TABLE plan (
    plan_id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id          UUID NOT NULL REFERENCES product(product_id),
    plan_code           VARCHAR(50) NOT NULL,
    plan_name           VARCHAR(255) NOT NULL,
    description         TEXT,
    base_premium        DECIMAL(18,2),
    status              VARCHAR(20) DEFAULT 'ACTIVE',
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(product_id, plan_code)
);

-- Coverage Master Table
CREATE TABLE coverage_master (
    coverage_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id          UUID NOT NULL REFERENCES product(product_id),
    coverage_code       VARCHAR(50) NOT NULL,
    coverage_name       VARCHAR(255) NOT NULL,
    coverage_type       VARCHAR(20) DEFAULT 'STANDARD', -- STANDARD, OPTIONAL
    description         TEXT,
    default_limit       DECIMAL(18,2),
    max_limit           DECIMAL(18,2),
    default_deductible  DECIMAL(18,2) DEFAULT 0,
    waiting_period_days INT DEFAULT 0,
    is_mandatory        BOOLEAN DEFAULT FALSE,
    premium_rate        DECIMAL(8,4),
    status              VARCHAR(20) DEFAULT 'ACTIVE',
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(product_id, coverage_code)
);

-- Rate Table
CREATE TABLE rate_table (
    rate_id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id          UUID NOT NULL REFERENCES product(product_id),
    rate_type           VARCHAR(50) NOT NULL,          -- BASE, AGE, OCCUPATION, AREA
    factor_name         VARCHAR(100) NOT NULL,
    factor_value        VARCHAR(100),
    min_value           DECIMAL(18,2),
    max_value           DECIMAL(18,2),
    rate                DECIMAL(10,6) NOT NULL,
    effective_from      DATE NOT NULL,
    effective_to        DATE,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_product_code ON product(product_code);
CREATE INDEX idx_product_status ON product(status);
CREATE INDEX idx_rate_product ON rate_table(product_id, rate_type);
```

---

## 5.3 State Machine Diagrams

### Policy Status State Machine

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       POLICY STATUS STATE MACHINE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌──────────┐                                        │
│                         │  DRAFT   │                                        │
│                         └────┬─────┘                                        │
│                              │ submit_quote                                 │
│                              ↓                                              │
│                         ┌──────────┐                                        │
│                         │  QUOTED  │ ←─────────────────┐                    │
│                         └────┬─────┘                   │                    │
│                              │ convert_proposal        │ requote            │
│                              ↓                         │                    │
│                         ┌──────────┐                   │                    │
│                         │ PROPOSED │ ──────────────────┤                    │
│                         └────┬─────┘                   │                    │
│                              │                         │                    │
│              ┌───────────────┴───────────────┐        │                    │
│              ↓                               ↓        │                    │
│        ┌──────────┐                    ┌──────────┐   │                    │
│        │ DECLINED │                    │  ISSUED  │   │                    │
│        └──────────┘                    └────┬─────┘   │                    │
│                                             │ activate│                    │
│                                             ↓         │                    │
│                                        ┌──────────┐   │                    │
│              ┌─────────────────────────│  ACTIVE  │───┘                    │
│              │                         └────┬─────┘                        │
│              │                              │                              │
│              │         ┌────────────────────┼────────────────────┐         │
│              │         ↓                    ↓                    ↓         │
│              │   ┌──────────┐        ┌──────────┐         ┌──────────┐    │
│              │   │  LAPSED  │        │ EXPIRED  │         │CANCELLED │    │
│              │   └────┬─────┘        └──────────┘         └──────────┘    │
│              │        │                                                    │
│              │        │ reinstate                                          │
│              │        ↓                                                    │
│              │   ┌──────────┐                                              │
│              └──→│  ACTIVE  │                                              │
│                  └──────────┘                                              │
│                                                                              │
│  TRANSITIONS:                                                               │
│  ─────────────                                                              │
│  DRAFT → QUOTED: submit_quote (premium calculated)                         │
│  QUOTED → PROPOSED: convert_proposal (documents submitted)                 │
│  PROPOSED → ISSUED: approve (underwriting passed, payment received)        │
│  PROPOSED → DECLINED: reject (underwriting failed)                         │
│  ISSUED → ACTIVE: activate (effective date reached)                        │
│  ACTIVE → LAPSED: lapse (premium not paid within grace period)            │
│  ACTIVE → CANCELLED: cancel (customer/insurer initiated)                   │
│  ACTIVE → EXPIRED: expire (expiry date reached, not renewed)              │
│  LAPSED → ACTIVE: reinstate (arrears paid within reinstatement period)    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Claim Status State Machine

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CLAIM STATUS STATE MACHINE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              ┌──────────┐                                   │
│                              │  DRAFT   │                                   │
│                              └────┬─────┘                                   │
│                                   │ submit                                  │
│                                   ↓                                         │
│                              ┌──────────┐                                   │
│                              │   OPEN   │                                   │
│                              └────┬─────┘                                   │
│                                   │ assign                                  │
│                                   ↓                                         │
│                              ┌──────────┐                                   │
│                              │ ASSIGNED │                                   │
│                              └────┬─────┘                                   │
│                                   │ investigate                             │
│                                   ↓                                         │
│                              ┌──────────┐       ┌──────────┐               │
│                     ┌───────│  UNDER   │←──────│ ON_HOLD  │               │
│                     │       │  REVIEW  │───────→│          │               │
│                     │       └────┬─────┘ hold   └──────────┘               │
│                     │            │       resume                             │
│                     │            │ decide                                   │
│        ┌────────────┼────────────┼────────────────────────────┐            │
│        ↓            ↓            ↓                            ↓            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐              ┌──────────┐         │
│  │ APPROVED │ │ PARTIAL  │ │ REJECTED │              │  FRAUD   │         │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘              │ REFERRED │         │
│       │            │            │                     └──────────┘         │
│       │ process    │ process    │                                          │
│       ↓            ↓            │                                          │
│  ┌──────────┐ ┌──────────┐     │                                          │
│  │ SETTLED  │ │ SETTLED  │     │                                          │
│  └────┬─────┘ └────┬─────┘     │                                          │
│       │            │            │                                          │
│       │ close      │ close      │ close                                    │
│       ↓            ↓            ↓                                          │
│  ┌─────────────────────────────────────┐                                   │
│  │              CLOSED                  │                                   │
│  └─────────────────────────────────────┘                                   │
│                    │                                                        │
│                    │ reopen                                                 │
│                    ↓                                                        │
│              ┌──────────┐                                                   │
│              │ REOPENED │ → (returns to UNDER_REVIEW)                       │
│              └──────────┘                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5.4 Audit Trail

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
    timestamp           TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- Indexes
    CONSTRAINT pk_audit_log PRIMARY KEY (audit_id)
);

-- Indexes for efficient querying
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_user ON audit_log(user_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp);
CREATE INDEX idx_audit_action ON audit_log(action);
CREATE INDEX idx_audit_correlation ON audit_log(correlation_id);
```

---

## 5.5 Data Modeling Best Practices

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
┌─────────────────────────────────────────────────────────────────┐
│                    DATA MODELING PATTERNS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. TEMPORAL DATA                                                │
│     - Use effective_from/effective_to for validity periods      │
│     - Never delete, only soft-delete with is_active flag        │
│     - Track created_at/updated_at for all records               │
│                                                                  │
│  2. VERSIONING                                                   │
│     - Version column for optimistic locking                     │
│     - Parent reference for version history                      │
│     - Snapshot tables for point-in-time queries                 │
│                                                                  │
│  3. STATUS TRACKING                                              │
│     - Status column with enum constraints                       │
│     - Sub_status for detailed state                             │
│     - Status history table for transitions                      │
│                                                                  │
│  4. AMOUNTS                                                      │
│     - Use DECIMAL(18,2) for money                               │
│     - Always store currency code                                │
│     - Separate gross/net/tax amounts                            │
│                                                                  │
│  5. REFERENCES                                                   │
│     - UUID for primary keys                                     │
│     - Business keys (policy_number) as UNIQUE                   │
│     - Soft references for external systems                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
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

[← Previous Lesson](../04-system-architecture/README.md) | [Next Lesson: API Design →](../06-api-design/README.md)
