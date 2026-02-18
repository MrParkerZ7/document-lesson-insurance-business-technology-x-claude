# 6.5 Event Schema Design

## Overview

This sub-lesson covers event-driven architecture patterns, event envelope structures, and key events in insurance systems.

---

## Event Envelope

```json
{
  "event_id": "evt_abc123def456",
  "event_type": "policy.issued",
  "event_version": "1.0",
  "source": "policy-service",
  "timestamp": "2024-01-15T10:30:00Z",
  "correlation_id": "corr_xyz789",
  "causation_id": "evt_previous123",
  "data": {
    // Event-specific payload
  },
  "metadata": {
    "user_id": "user_001",
    "channel": "WEB_PORTAL",
    "trace_id": "trace_abc123"
  }
}
```

### Envelope Fields

| Field | Type | Description |
|-------|------|-------------|
| `event_id` | string | Unique identifier for the event |
| `event_type` | string | Type of event (domain.action format) |
| `event_version` | string | Schema version for backward compatibility |
| `source` | string | Service that produced the event |
| `timestamp` | datetime | When the event occurred (ISO 8601) |
| `correlation_id` | string | ID to correlate related events |
| `causation_id` | string | ID of the event that caused this event |
| `data` | object | Event-specific payload |
| `metadata` | object | Additional context (user, channel, trace) |

---

## Key Events

| Event Type | Trigger | Subscribers |
|------------|---------|-------------|
| `quote.created` | New quote generated | Analytics |
| `quote.converted` | Quote to proposal | Notification |
| `policy.issued` | Policy issued | Notification, Billing, Analytics |
| `policy.renewed` | Policy renewed | Notification, Commission |
| `policy.cancelled` | Policy cancelled | Notification, Refund |
| `policy.lapsed` | Premium not paid | Notification, Collections |
| `claim.registered` | FNOL submitted | Notification, Assignment |
| `claim.assessed` | Assessment complete | Notification |
| `claim.approved` | Claim approved | Payment, Notification |
| `claim.settled` | Payment processed | Notification, Analytics |
| `payment.received` | Payment successful | Policy, Notification |
| `payment.failed` | Payment failed | Notification, Collections |

---

## Event Examples

### Policy Issued Event

```json
{
  "event_id": "evt_pol_issued_001",
  "event_type": "policy.issued",
  "event_version": "1.0",
  "source": "policy-service",
  "timestamp": "2024-01-15T10:30:00Z",
  "correlation_id": "corr_xyz789",
  "data": {
    "policy_id": "POL-2024-001234",
    "policy_number": "INS-2024-001234",
    "customer_id": "cust_123",
    "product_code": "MOTOR_COMP",
    "effective_date": "2024-02-01",
    "expiry_date": "2025-01-31",
    "total_premium": 1185.50,
    "currency": "USD",
    "payment_frequency": "ANNUAL"
  },
  "metadata": {
    "user_id": "agent_001",
    "channel": "AGENT_PORTAL",
    "trace_id": "trace_abc123"
  }
}
```

### Claim Registered Event

```json
{
  "event_id": "evt_clm_registered_001",
  "event_type": "claim.registered",
  "event_version": "1.0",
  "source": "claims-service",
  "timestamp": "2024-01-15T11:00:00Z",
  "correlation_id": "corr_def456",
  "data": {
    "claim_id": "CLM-2024-005678",
    "policy_id": "POL-2024-001234",
    "customer_id": "cust_123",
    "loss_date": "2024-01-14",
    "loss_cause": "COLLISION",
    "claimed_amount": 5000.00,
    "priority": "MEDIUM"
  },
  "metadata": {
    "user_id": "cust_123",
    "channel": "MOBILE_APP",
    "trace_id": "trace_def456"
  }
}
```

### Payment Received Event

```json
{
  "event_id": "evt_pmt_received_001",
  "event_type": "payment.received",
  "event_version": "1.0",
  "source": "payment-service",
  "timestamp": "2024-01-15T12:00:00Z",
  "correlation_id": "corr_ghi789",
  "data": {
    "payment_id": "PMT-2024-001234",
    "policy_id": "POL-2024-001234",
    "customer_id": "cust_123",
    "amount": 1185.50,
    "currency": "USD",
    "payment_method": "CREDIT_CARD",
    "reference_number": "REF-ABC123"
  },
  "metadata": {
    "channel": "WEB_PORTAL",
    "trace_id": "trace_ghi789"
  }
}
```

---

## Event Design Best Practices

### 1. Event Naming Convention

Use `domain.action` format:
- `policy.issued`
- `claim.approved`
- `payment.failed`

### 2. Versioning

Include version in event schema for backward compatibility:

```json
{
  "event_type": "policy.issued",
  "event_version": "2.0",
  "data": {
    // New schema with additional fields
  }
}
```

### 3. Idempotency

Events should be idempotent - processing the same event twice should have the same result as processing it once.

### 4. Event Ordering

Include `sequence_number` or use `timestamp` for ordering when needed:

```json
{
  "event_id": "evt_001",
  "timestamp": "2024-01-15T10:30:00.123Z",
  "sequence_number": 1234
}
```

### 5. Data Completeness

Include all necessary data in the event to avoid additional API calls:

```json
{
  "event_type": "policy.cancelled",
  "data": {
    "policy_id": "POL-2024-001234",
    "customer_id": "cust_123",
    "customer_email": "john.doe@email.com",
    "cancellation_reason": "CUSTOMER_REQUEST",
    "effective_date": "2024-02-15",
    "refund_amount": 500.00
  }
}
```

---

## Event Flow Diagram

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Producer   │───>│  Event Bus   │───>│  Consumer    │
│   Service    │    │  (Kafka/     │    │  Service(s)  │
│              │    │   RabbitMQ)  │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
       │                   │                   │
       │                   │                   │
       ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Policy       │    │ Topic:       │    │ Notification │
│ Service      │    │ insurance.   │    │ Service      │
│              │    │ events       │    │              │
│ Events:      │    │              │    │ Listens:     │
│ - issued     │    │ Partitions:  │    │ - policy.*   │
│ - renewed    │    │ policy_id    │    │ - claim.*    │
│ - cancelled  │    │              │    │ - payment.*  │
└──────────────┘    └──────────────┘    └──────────────┘
```

---

## Key Takeaways

1. Use a consistent event envelope structure across all services
2. Version events for backward compatibility
3. Include correlation IDs for distributed tracing
4. Design events to be self-contained with necessary data
5. Ensure events are idempotent for reliable processing

---

[Previous: Pagination & Filtering](../04-pagination-filtering/README.md) | [Back to Lesson 06](../README.md) | [Next: External Integrations](../06-external-integrations/README.md)
