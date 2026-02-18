# 7.3 Data Protection

## Overview

Insurance companies handle some of the most sensitive personal and financial data. This sub-lesson covers data classification, masking strategies, and encryption implementations to protect customer information.

---

## Sensitive Data Classification

Data classification determines the level of protection required for each data type:

| Classification | Examples | Protection Requirements |
|---------------|----------|------------------------|
| **Critical** | SSN, National ID, Bank Account | Encrypted + Tokenized + Access Logged |
| **Highly Sensitive** | Medical Records, Credit Score | Encrypted + Access Controlled |
| **Sensitive** | DOB, Address, Phone | Encrypted + Role-Based Access |
| **PII** | Name, Email | Encrypted in Transit |
| **Internal** | Policy Details | Standard Access Control |
| **Public** | Product Catalog | No Restrictions |

### Classification Guidelines

```
+-----------------------------------------------------------------------+
|                    DATA CLASSIFICATION MATRIX                          |
+-----------------------------------------------------------------------+
|                                                                        |
|  CRITICAL                                                              |
|  +------------------------------------------------------------------+ |
|  | SSN | National ID | Bank Account | Tax ID | Passport Number      | |
|  +------------------------------------------------------------------+ |
|  | Requirements: AES-256 encryption, tokenization, full audit       | |
|  |               logging, approval required for access              | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  HIGHLY SENSITIVE                                                      |
|  +------------------------------------------------------------------+ |
|  | Medical Records | Credit Score | Income | Employment History     | |
|  +------------------------------------------------------------------+ |
|  | Requirements: AES-256 encryption, role-based access,             | |
|  |               audit logging                                       | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  SENSITIVE                                                             |
|  +------------------------------------------------------------------+ |
|  | Date of Birth | Address | Phone | Driver's License               | |
|  +------------------------------------------------------------------+ |
|  | Requirements: Encryption at rest, RBAC                            | |
|  +------------------------------------------------------------------+ |
|                                                                        |
|  PII                                                                   |
|  +------------------------------------------------------------------+ |
|  | Name | Email | Policy Number | Claim Number                       | |
|  +------------------------------------------------------------------+ |
|  | Requirements: TLS encryption, standard access control             | |
|  +------------------------------------------------------------------+ |
|                                                                        |
+-----------------------------------------------------------------------+
```

---

## Data Masking Rules

Data masking protects sensitive information while allowing legitimate business operations:

```
+-----------------------------------------------------------------+
|                       DATA MASKING                               |
+-----------------------------------------------------------------+
|                                                                  |
|  FIELD              ORIGINAL              MASKED                |
|  ------------------------------------------------------------   |
|  SSN                123-45-6789          XXX-XX-6789            |
|  Credit Card        4532-1234-5678-9012  XXXX-XXXX-XXXX-9012   |
|  Bank Account       1234567890           XXXXXX7890             |
|  Email              john.doe@email.com   j***e@email.com        |
|  Phone              +1-555-123-4567      +1-555-XXX-4567        |
|  Address            123 Main Street      123 M*** S*****        |
|  Date of Birth      1985-03-15           XXXX-XX-15             |
|                                                                  |
|  MASKING BY ROLE:                                               |
|  ---------------                                                |
|  Customer Support:  Partial masking (last 4 digits visible)     |
|  Agent:             Partial masking (with consent)              |
|  Underwriter:       Full access (logged)                        |
|  System Admin:      Full access (logged + approved)             |
|                                                                  |
+-----------------------------------------------------------------+
```

### Masking Implementation

```python
class DataMasker:
    """Utility class for masking sensitive data"""

    @staticmethod
    def mask_ssn(ssn: str, show_last: int = 4) -> str:
        """Mask SSN showing only last N digits"""
        if not ssn:
            return ""
        clean = ssn.replace("-", "")
        masked = "X" * (len(clean) - show_last) + clean[-show_last:]
        return f"{masked[:3]}-{masked[3:5]}-{masked[5:]}"

    @staticmethod
    def mask_credit_card(number: str, show_last: int = 4) -> str:
        """Mask credit card showing only last N digits"""
        clean = number.replace("-", "").replace(" ", "")
        masked = "X" * (len(clean) - show_last) + clean[-show_last:]
        return "-".join([masked[i:i+4] for i in range(0, len(masked), 4)])

    @staticmethod
    def mask_email(email: str) -> str:
        """Mask email address"""
        if "@" not in email:
            return "***"
        local, domain = email.split("@")
        if len(local) <= 2:
            return f"*@{domain}"
        return f"{local[0]}***{local[-1]}@{domain}"

    @staticmethod
    def mask_phone(phone: str) -> str:
        """Mask phone number showing first and last parts"""
        digits = ''.join(filter(str.isdigit, phone))
        if len(digits) < 7:
            return "XXX-XXXX"
        return f"{phone[:6]}XXX-{digits[-4:]}"
```

---

## Encryption Strategy

### Encryption Configuration

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

### Encryption Layers

| Layer | Method | Use Case |
|-------|--------|----------|
| **At Rest** | AES-256-GCM | Database storage |
| **In Transit** | TLS 1.3 | API communication |
| **Field Level** | AES-256-GCM | Sensitive columns |
| **Application** | AES-256-CBC | Configuration secrets |
| **Tokenization** | Token vault | Payment card data |

### Key Management

```
+-----------------------------------------------------------------------+
|                       KEY MANAGEMENT                                   |
+-----------------------------------------------------------------------+
|                                                                        |
|  +-----------------+      +-------------------+     +--------------+   |
|  |  Application    |      |   Key Management  |     |    HSM       |   |
|  |    Server       |----->|      Service      |---->|  (Hardware)  |   |
|  +-----------------+      +-------------------+     +--------------+   |
|                                  |                                     |
|                                  v                                     |
|  KEY ROTATION POLICY:                                                  |
|  - Master Key: Annual rotation with approval                           |
|  - Data Encryption Keys: 90-day rotation                              |
|  - Session Keys: Per-session or hourly                                |
|  - API Keys: Quarterly rotation                                        |
|                                                                        |
|  KEY HIERARCHY:                                                        |
|  +------------------------------------------------------------------+ |
|  |  Master Key (KMS) --> Key Encryption Key --> Data Encryption Key | |
|  +------------------------------------------------------------------+ |
|                                                                        |
+-----------------------------------------------------------------------+
```

---

## Data Loss Prevention (DLP)

### DLP Rules

| Rule | Pattern | Action |
|------|---------|--------|
| **SSN Detection** | `\d{3}-\d{2}-\d{4}` | Block/Alert |
| **Credit Card** | `\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}` | Block/Mask |
| **Email PII** | Bulk email addresses | Alert |
| **Medical Terms** | ICD codes, diagnosis | Encrypt/Alert |

### DLP Implementation Points

1. **Email Gateways**: Scan outbound emails for sensitive data
2. **API Responses**: Mask sensitive fields based on user role
3. **File Exports**: Scan and redact before download
4. **Database Queries**: Monitor for bulk data extraction

---

## Navigation

[Back to Lesson 07](../README.md) | [Previous: Access Control](../02-access-control/README.md) | [Next: Regulatory Compliance](../04-regulatory-compliance/README.md)
