# Lesson 06: API Design & Integration

## Learning Objectives

By the end of this lesson, you will be able to:
- Design RESTful APIs for insurance systems
- Implement event-driven integration patterns
- Define API contracts and documentation
- Integrate with external systems (payment, KYC, reinsurance)

---

## 6.1 RESTful API Design

### API Endpoints Structure

```
/api/v1
├── /customers
│   ├── GET     /                       # List customers (paginated)
│   ├── POST    /                       # Create customer
│   ├── GET     /{id}                   # Get customer by ID
│   ├── PUT     /{id}                   # Update customer
│   ├── DELETE  /{id}                   # Soft delete customer
│   ├── GET     /{id}/policies          # Get customer's policies
│   ├── GET     /{id}/claims            # Get customer's claims
│   ├── GET     /{id}/quotes            # Get customer's quotes
│   └── POST    /{id}/kyc               # Submit KYC documents
│
├── /quotes
│   ├── POST    /                       # Create quote
│   ├── GET     /{id}                   # Get quote details
│   ├── PUT     /{id}                   # Update quote
│   ├── POST    /{id}/calculate         # Recalculate premium
│   ├── POST    /{id}/convert           # Convert to proposal
│   ├── GET     /{id}/compare           # Compare with alternatives
│   └── POST    /{id}/share             # Share quote via email
│
├── /policies
│   ├── GET     /                       # List policies (paginated)
│   ├── POST    /                       # Issue policy
│   ├── GET     /{id}                   # Get policy details
│   ├── GET     /{id}/coverages         # Get policy coverages
│   ├── GET     /{id}/documents         # Get policy documents
│   ├── GET     /{id}/premiums          # Get premium schedule
│   ├── GET     /{id}/claims            # Get policy claims
│   ├── POST    /{id}/endorse           # Create endorsement
│   ├── POST    /{id}/renew             # Renew policy
│   ├── POST    /{id}/cancel            # Cancel policy
│   └── POST    /{id}/reinstate         # Reinstate lapsed policy
│
├── /claims
│   ├── GET     /                       # List claims (paginated)
│   ├── POST    /                       # Register claim (FNOL)
│   ├── GET     /{id}                   # Get claim details
│   ├── PUT     /{id}                   # Update claim
│   ├── GET     /{id}/documents         # Get claim documents
│   ├── POST    /{id}/documents         # Upload claim document
│   ├── GET     /{id}/timeline          # Get claim timeline
│   ├── POST    /{id}/assess            # Submit assessment
│   ├── POST    /{id}/approve           # Approve claim
│   ├── POST    /{id}/reject            # Reject claim
│   ├── POST    /{id}/settle            # Settle claim
│   └── POST    /{id}/reopen            # Reopen closed claim
│
├── /payments
│   ├── POST    /                       # Process payment
│   ├── GET     /{id}                   # Get payment status
│   ├── POST    /{id}/refund            # Process refund
│   └── GET     /methods                # Get available payment methods
│
└── /products
    ├── GET     /                       # List products
    ├── GET     /{id}                   # Get product details
    ├── GET     /{id}/plans             # Get product plans
    ├── GET     /{id}/coverages         # Get product coverages
    └── GET     /{id}/rates             # Get rating tables
```

---

## 6.2 Request/Response Examples

### Create Quote

**Request:**
```http
POST /api/v1/quotes
Content-Type: application/json
Authorization: Bearer {token}
X-Correlation-ID: corr_abc123

{
  "product_code": "MOTOR_COMP",
  "customer": {
    "customer_id": "cust_existing_123",
    "first_name": "John",
    "last_name": "Doe",
    "date_of_birth": "1985-03-15",
    "email": "john.doe@email.com",
    "phone": "+1-555-123-4567"
  },
  "risk_details": {
    "vehicle": {
      "type": "CAR",
      "make": "Toyota",
      "model": "Camry",
      "year": 2022,
      "registration_number": "ABC-1234",
      "vin": "1HGBH41JXMN109186",
      "engine_capacity_cc": 2500,
      "fuel_type": "PETROL",
      "color": "Silver"
    },
    "usage": {
      "type": "PRIVATE",
      "annual_mileage": 15000,
      "parking_location": "GARAGE"
    },
    "driver": {
      "license_number": "DL-123456",
      "license_date": "2005-06-20",
      "years_driving": 18,
      "claims_history": 0
    }
  },
  "coverage_options": {
    "sum_insured": 50000,
    "deductible": 500,
    "add_ons": ["ROADSIDE_ASSIST", "WINDSCREEN", "NCB_PROTECT"]
  },
  "effective_date": "2024-02-01",
  "policy_term_months": 12
}
```

**Response:**
```json
{
  "quote_id": "QT-2024-001234",
  "status": "ACTIVE",
  "created_at": "2024-01-15T10:30:00Z",
  "valid_until": "2024-01-30T23:59:59Z",
  "product": {
    "code": "MOTOR_COMP",
    "name": "Comprehensive Motor Insurance"
  },
  "policy_term": {
    "effective_date": "2024-02-01",
    "expiry_date": "2025-01-31",
    "term_months": 12
  },
  "coverage_summary": {
    "sum_insured": 50000.00,
    "deductible": 500.00,
    "coverages": [
      {
        "code": "TPL",
        "name": "Third Party Liability",
        "limit": 1000000.00
      },
      {
        "code": "OD",
        "name": "Own Damage",
        "limit": 50000.00,
        "deductible": 500.00
      }
    ],
    "add_ons": [
      {
        "code": "ROADSIDE_ASSIST",
        "name": "24/7 Roadside Assistance",
        "premium": 50.00
      },
      {
        "code": "WINDSCREEN",
        "name": "Windscreen Cover",
        "premium": 30.00
      },
      {
        "code": "NCB_PROTECT",
        "name": "No Claim Bonus Protection",
        "premium": 75.00
      }
    ]
  },
  "premium_breakdown": {
    "base_premium": 1200.00,
    "loadings": [],
    "discounts": [
      {
        "type": "NO_CLAIMS_BONUS",
        "description": "5 years no claims",
        "amount": -240.00,
        "percentage": -20
      },
      {
        "type": "ANTI_THEFT",
        "description": "Anti-theft device installed",
        "amount": -60.00,
        "percentage": -5
      }
    ],
    "add_ons_premium": 155.00,
    "net_premium": 1055.00,
    "tax": {
      "type": "INSURANCE_TAX",
      "rate": 10,
      "amount": 105.50
    },
    "fees": {
      "policy_fee": 25.00
    },
    "total_premium": 1185.50,
    "currency": "USD"
  },
  "payment_options": [
    {
      "frequency": "ANNUAL",
      "installments": 1,
      "amount_per_installment": 1185.50,
      "total_amount": 1185.50
    },
    {
      "frequency": "SEMI_ANNUAL",
      "installments": 2,
      "amount_per_installment": 605.00,
      "total_amount": 1210.00
    },
    {
      "frequency": "QUARTERLY",
      "installments": 4,
      "amount_per_installment": 310.00,
      "total_amount": 1240.00
    },
    {
      "frequency": "MONTHLY",
      "installments": 12,
      "amount_per_installment": 105.00,
      "total_amount": 1260.00
    }
  ],
  "_links": {
    "self": "/api/v1/quotes/QT-2024-001234",
    "convert": "/api/v1/quotes/QT-2024-001234/convert",
    "compare": "/api/v1/quotes/QT-2024-001234/compare",
    "customer": "/api/v1/customers/cust_existing_123"
  }
}
```

### Register Claim (FNOL)

**Request:**
```http
POST /api/v1/claims
Content-Type: application/json
Authorization: Bearer {token}

{
  "policy_id": "POL-2024-001234",
  "loss_details": {
    "date": "2024-01-14",
    "time": "14:30:00",
    "location": {
      "address": "123 Main Street",
      "city": "Springfield",
      "state": "IL",
      "postal_code": "62701",
      "coordinates": {
        "latitude": 39.7817,
        "longitude": -89.6501
      }
    },
    "description": "Rear-end collision at traffic light. Other driver ran red light and hit my vehicle from behind.",
    "cause": "COLLISION"
  },
  "damage_details": {
    "vehicle_drivable": false,
    "airbags_deployed": false,
    "damage_areas": ["REAR_BUMPER", "TRUNK", "REAR_LIGHTS"],
    "estimated_damage": 5000.00
  },
  "third_party": {
    "involved": true,
    "details": {
      "name": "Jane Smith",
      "phone": "+1-555-987-6543",
      "vehicle_registration": "XYZ-5678",
      "insurance_company": "Other Insurance Co",
      "policy_number": "OIC-98765"
    }
  },
  "injuries": {
    "policyholder_injured": false,
    "third_party_injured": false
  },
  "witnesses": [
    {
      "name": "Bob Wilson",
      "phone": "+1-555-111-2222",
      "statement": "I saw the other car run the red light"
    }
  ],
  "police_report": {
    "filed": true,
    "report_number": "SPD-2024-00123",
    "station": "Springfield Police Department"
  },
  "claimant": {
    "relationship": "SELF",
    "contact_preference": "EMAIL"
  }
}
```

**Response:**
```json
{
  "claim_id": "CLM-2024-005678",
  "claim_number": "CLM-2024-005678",
  "status": "OPEN",
  "priority": "MEDIUM",
  "created_at": "2024-01-15T11:00:00Z",
  "policy": {
    "policy_id": "POL-2024-001234",
    "policy_number": "INS-2024-001234",
    "product": "MOTOR_COMP"
  },
  "loss_date": "2024-01-14",
  "notification_date": "2024-01-15",
  "claimed_amount": 5000.00,
  "coverage_applicable": [
    {
      "code": "OD",
      "name": "Own Damage",
      "limit": 50000.00,
      "deductible": 500.00
    }
  ],
  "assigned_adjuster": {
    "id": "ADJ-001",
    "name": "Michael Johnson",
    "email": "m.johnson@insurance.com",
    "phone": "+1-555-CLAIMS"
  },
  "next_steps": [
    "Upload photos of vehicle damage",
    "Submit repair estimate from authorized garage",
    "Provide copy of police report"
  ],
  "required_documents": [
    {
      "type": "DAMAGE_PHOTOS",
      "description": "Photos of all damaged areas",
      "status": "PENDING"
    },
    {
      "type": "REPAIR_ESTIMATE",
      "description": "Estimate from authorized garage",
      "status": "PENDING"
    },
    {
      "type": "POLICE_REPORT",
      "description": "Copy of filed police report",
      "status": "PENDING"
    }
  ],
  "sla": {
    "target_first_contact": "2024-01-16T11:00:00Z",
    "target_assessment": "2024-01-22T11:00:00Z",
    "target_settlement": "2024-01-29T11:00:00Z"
  },
  "_links": {
    "self": "/api/v1/claims/CLM-2024-005678",
    "documents": "/api/v1/claims/CLM-2024-005678/documents",
    "timeline": "/api/v1/claims/CLM-2024-005678/timeline",
    "policy": "/api/v1/policies/POL-2024-001234"
  }
}
```

---

## 6.3 Error Handling

### Standard Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "risk_details.vehicle.year",
        "code": "INVALID_VALUE",
        "message": "Vehicle year must be between 2000 and current year"
      },
      {
        "field": "coverage_options.sum_insured",
        "code": "EXCEEDS_LIMIT",
        "message": "Sum insured exceeds maximum allowed value of $100,000"
      }
    ],
    "correlation_id": "corr_abc123",
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/v1/quotes",
    "request_id": "req_xyz789"
  }
}
```

### Error Code Categories

| Code Range | Category | Examples |
|------------|----------|----------|
| 1xxx | Validation Errors | 1001: Missing required field |
| 2xxx | Authentication/Authorization | 2001: Invalid token |
| 3xxx | Business Rule Violations | 3001: Policy not active |
| 4xxx | External Service Errors | 4001: Payment gateway timeout |
| 5xxx | System Errors | 5001: Database connection failed |

### HTTP Status Codes

| Status | Usage |
|--------|-------|
| 200 | Successful GET, PUT |
| 201 | Successful POST (created) |
| 204 | Successful DELETE |
| 400 | Validation error |
| 401 | Authentication required |
| 403 | Forbidden (insufficient permissions) |
| 404 | Resource not found |
| 409 | Conflict (duplicate, state conflict) |
| 422 | Business rule violation |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
| 502 | External service error |
| 503 | Service unavailable |

---

## 6.4 API Pagination & Filtering

### Pagination Request

```http
GET /api/v1/policies?page=1&size=20&sort=created_at,desc
```

### Pagination Response

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "size": 20,
    "total_items": 156,
    "total_pages": 8,
    "has_next": true,
    "has_previous": false
  },
  "_links": {
    "self": "/api/v1/policies?page=1&size=20",
    "first": "/api/v1/policies?page=1&size=20",
    "last": "/api/v1/policies?page=8&size=20",
    "next": "/api/v1/policies?page=2&size=20"
  }
}
```

### Filtering Examples

```http
# Filter by status
GET /api/v1/policies?status=ACTIVE

# Filter by date range
GET /api/v1/claims?loss_date_from=2024-01-01&loss_date_to=2024-01-31

# Filter by multiple values
GET /api/v1/policies?status=ACTIVE,LAPSED

# Search
GET /api/v1/customers?search=john.doe@email.com

# Complex filter
GET /api/v1/claims?status=OPEN&priority=HIGH&adjuster_id=ADJ-001
```

---

## 6.5 Event Schema Design

### Event Envelope

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

### Key Events

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

## 6.6 External Integrations

### Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       EXTERNAL INTEGRATIONS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐     ┌─────────────────────────────────────────────────┐   │
│  │  Insurance  │     │              INTEGRATION LAYER                   │   │
│  │   System    │     │  ┌─────────────┐  ┌─────────────┐              │   │
│  │             │────→│  │   Adapter   │  │   Circuit   │              │   │
│  │             │     │  │   Pattern   │  │   Breaker   │              │   │
│  └─────────────┘     │  └─────────────┘  └─────────────┘              │   │
│                      │         │                │                      │   │
│                      │         ↓                ↓                      │   │
│                      │  ┌──────────────────────────────────────────┐  │   │
│                      │  │           EXTERNAL SERVICES               │  │   │
│                      │  └──────────────────────────────────────────┘  │   │
│                      └─────────────────────────────────────────────────┘   │
│                                         │                                   │
│         ┌───────────────────────────────┼───────────────────────────────┐  │
│         ↓                 ↓             ↓             ↓                 ↓  │
│  ┌───────────┐     ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐│
│  │  Payment  │     │   KYC/    │ │Reinsurance│ │ Regulatory│ │  Third    ││
│  │  Gateway  │     │   AML     │ │  Systems  │ │ Reporting │ │  Party    ││
│  └───────────┘     └───────────┘ └───────────┘ └───────────┘ └───────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Integration Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Synchronous REST** | Real-time data retrieval | KYC verification |
| **Async Webhook** | Event notification | Payment confirmation |
| **Message Queue** | Reliable delivery | Reinsurance bordereaux |
| **File Transfer** | Batch processing | Regulatory reports |
| **GraphQL** | Flexible queries | Partner portal |

---

## Key Takeaways

1. **REST API design** should follow consistent conventions
2. **Error handling** must be informative and actionable
3. **Pagination and filtering** enable efficient data access
4. **Event schemas** should be versioned and well-documented
5. **External integrations** require resilience patterns

---

## Exercises

1. Design API endpoints for policy endorsement workflow
2. Create an event schema for claim status changes
3. Implement error handling for payment gateway integration

---

[← Previous Lesson](../05-data-models/README.md) | [Next Lesson: Security & Compliance →](../07-security-compliance/README.md)
