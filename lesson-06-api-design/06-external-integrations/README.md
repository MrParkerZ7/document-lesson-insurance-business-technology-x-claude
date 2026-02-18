# 6.6 External Integrations

## Overview

This sub-lesson explores integration patterns for connecting with external systems like payment gateways, KYC services, and reinsurance systems.

---

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       EXTERNAL INTEGRATIONS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐     ┌─────────────────────────────────────────────────┐   │
│  │  Insurance  │     │              INTEGRATION LAYER                   │   │
│  │   System    │     │  ┌─────────────┐  ┌─────────────┐              │   │
│  │             │────>│  │   Adapter   │  │   Circuit   │              │   │
│  │             │     │  │   Pattern   │  │   Breaker   │              │   │
│  └─────────────┘     │  └─────────────┘  └─────────────┘              │   │
│                      │         │                │                      │   │
│                      │         ▼                ▼                      │   │
│                      │  ┌──────────────────────────────────────────┐  │   │
│                      │  │           EXTERNAL SERVICES               │  │   │
│                      │  └──────────────────────────────────────────┘  │   │
│                      └─────────────────────────────────────────────────┘   │
│                                         │                                   │
│         ┌───────────────────────────────┼───────────────────────────────┐  │
│         ▼                 ▼             ▼             ▼                 ▼  │
│  ┌───────────┐     ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐│
│  │  Payment  │     │   KYC/    │ │Reinsurance│ │ Regulatory│ │  Third    ││
│  │  Gateway  │     │   AML     │ │  Systems  │ │ Reporting │ │  Party    ││
│  └───────────┘     └───────────┘ └───────────┘ └───────────┘ └───────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Integration Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Synchronous REST** | Real-time data retrieval | KYC verification |
| **Async Webhook** | Event notification | Payment confirmation |
| **Message Queue** | Reliable delivery | Reinsurance bordereaux |
| **File Transfer** | Batch processing | Regulatory reports |
| **GraphQL** | Flexible queries | Partner portal |

---

## Payment Gateway Integration

### Request

```http
POST /api/v1/payments
Content-Type: application/json
Authorization: Bearer {token}

{
  "policy_id": "POL-2024-001234",
  "amount": 1185.50,
  "currency": "USD",
  "payment_method": {
    "type": "CREDIT_CARD",
    "card_token": "tok_visa_4242",
    "billing_address": {
      "line1": "123 Main St",
      "city": "Springfield",
      "state": "IL",
      "postal_code": "62701",
      "country": "US"
    }
  },
  "idempotency_key": "idem_abc123"
}
```

### Response

```json
{
  "payment_id": "PMT-2024-001234",
  "status": "COMPLETED",
  "gateway_reference": "ch_3abc123def",
  "amount": 1185.50,
  "currency": "USD",
  "processed_at": "2024-01-15T10:30:00Z",
  "receipt_url": "https://receipts.payment.com/ch_3abc123def"
}
```

---

## KYC/AML Integration

### KYC Verification Request

```http
POST /api/v1/customers/{id}/kyc
Content-Type: application/json

{
  "verification_type": "IDENTITY",
  "documents": [
    {
      "type": "PASSPORT",
      "document_number": "AB123456",
      "issuing_country": "US",
      "expiry_date": "2028-06-15",
      "front_image_url": "https://storage.example.com/docs/passport_front.jpg",
      "back_image_url": "https://storage.example.com/docs/passport_back.jpg"
    }
  ],
  "consent": {
    "terms_accepted": true,
    "data_processing_accepted": true,
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### KYC Verification Response

```json
{
  "verification_id": "VRF-2024-001234",
  "status": "APPROVED",
  "risk_score": 15,
  "risk_level": "LOW",
  "checks": [
    {
      "type": "DOCUMENT_AUTHENTICITY",
      "status": "PASSED",
      "confidence": 0.98
    },
    {
      "type": "FACE_MATCH",
      "status": "PASSED",
      "confidence": 0.95
    },
    {
      "type": "SANCTIONS_SCREENING",
      "status": "PASSED",
      "matches": []
    },
    {
      "type": "PEP_CHECK",
      "status": "PASSED",
      "matches": []
    }
  ],
  "verified_at": "2024-01-15T10:35:00Z"
}
```

---

## Reinsurance Integration

### Bordereaux Report

```json
{
  "report_id": "BRD-2024-001",
  "report_type": "PREMIUM_BORDEREAUX",
  "treaty_number": "TRY-2024-001",
  "period": {
    "from": "2024-01-01",
    "to": "2024-01-31"
  },
  "cedant": {
    "code": "INS001",
    "name": "Insurance Company Ltd"
  },
  "reinsurer": {
    "code": "RE001",
    "name": "Reinsurance Corp"
  },
  "policies": [
    {
      "policy_number": "INS-2024-001234",
      "inception_date": "2024-01-15",
      "expiry_date": "2025-01-14",
      "gross_premium": 10000.00,
      "ceded_premium": 7000.00,
      "cession_percentage": 70,
      "sum_insured": 500000.00,
      "currency": "USD"
    }
  ],
  "summary": {
    "total_policies": 150,
    "total_gross_premium": 1500000.00,
    "total_ceded_premium": 1050000.00
  }
}
```

---

## Resilience Patterns

### Circuit Breaker

```
┌─────────────────────────────────────────────────────────────┐
│                    CIRCUIT BREAKER STATES                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│     ┌──────────┐         ┌──────────┐         ┌──────────┐ │
│     │  CLOSED  │ ──────> │   OPEN   │ ──────> │HALF-OPEN │ │
│     │          │ failure │          │ timeout │          │ │
│     │ (Normal) │ thresh- │ (Reject  │         │ (Test    │ │
│     │          │ old met │  calls)  │         │  calls)  │ │
│     └──────────┘         └──────────┘         └──────────┘ │
│          ▲                                          │      │
│          │                                          │      │
│          └──────────────────────────────────────────┘      │
│                        success                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Configuration

```json
{
  "circuit_breaker": {
    "failure_threshold": 5,
    "success_threshold": 3,
    "timeout_duration_seconds": 30,
    "half_open_requests": 3
  },
  "retry": {
    "max_attempts": 3,
    "initial_delay_ms": 100,
    "max_delay_ms": 5000,
    "multiplier": 2
  },
  "timeout": {
    "connection_timeout_ms": 5000,
    "read_timeout_ms": 30000
  }
}
```

---

## Webhook Integration

### Webhook Registration

```http
POST /api/v1/webhooks
Content-Type: application/json

{
  "url": "https://partner.example.com/webhooks/insurance",
  "events": [
    "policy.issued",
    "policy.cancelled",
    "claim.settled"
  ],
  "secret": "whsec_abc123...",
  "active": true
}
```

### Webhook Payload

```json
{
  "webhook_id": "wh_evt_001",
  "event_type": "policy.issued",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "policy_id": "POL-2024-001234",
    "policy_number": "INS-2024-001234",
    "customer_id": "cust_123"
  },
  "signature": "sha256=abc123..."
}
```

### Webhook Verification

```python
import hmac
import hashlib

def verify_webhook_signature(payload, signature, secret):
    expected = hmac.new(
        secret.encode('utf-8'),
        payload.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)
```

---

## Best Practices

### 1. Use Adapter Pattern

Abstract external service interfaces:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Service    │────>│   Adapter    │────>│   External   │
│              │     │   Interface  │     │   Service    │
└──────────────┘     └──────────────┘     └──────────────┘
```

### 2. Implement Idempotency

Use idempotency keys for payment and critical operations.

### 3. Handle Timeouts

Set appropriate timeouts for each external service.

### 4. Log All Interactions

Log requests and responses for debugging and audit.

### 5. Monitor Health

Implement health checks for all external integrations.

---

## Key Takeaways

1. Use appropriate integration patterns for each use case
2. Implement circuit breakers for resilience
3. Secure webhook integrations with signature verification
4. Abstract external services using adapter pattern
5. Monitor and log all external interactions

---

[Previous: Event Schema Design](../05-event-schema-design/README.md) | [Back to Lesson 06](../README.md)
