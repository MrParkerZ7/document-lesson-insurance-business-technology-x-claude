# 6.2 Request/Response Patterns

## Overview

This sub-lesson explores detailed examples of request and response patterns for common insurance operations, demonstrating best practices for API payload design.

---

## Create Quote

### Request

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

### Response

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

---

## Register Claim (FNOL)

### Request

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

### Response

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

## Response Design Best Practices

### HATEOAS Links

Include `_links` object for related resources and actions:

```json
"_links": {
  "self": "/api/v1/resource/{id}",
  "related": "/api/v1/related/{related_id}",
  "action": "/api/v1/resource/{id}/action"
}
```

### Consistent Response Structure

All responses should follow a consistent structure:
- Resource data at the top level
- Timestamps in ISO 8601 format
- Currency amounts as decimals with currency code
- HATEOAS links for navigation

---

## Key Takeaways

1. Use descriptive field names that match business terminology
2. Include all relevant information in responses to reduce additional API calls
3. Provide actionable next steps in responses where appropriate
4. Use HATEOAS links to enable API discoverability

---

[Previous: RESTful API Design](../01-restful-api-design/README.md) | [Back to Lesson 06](../README.md) | [Next: Error Handling](../03-error-handling/README.md)
