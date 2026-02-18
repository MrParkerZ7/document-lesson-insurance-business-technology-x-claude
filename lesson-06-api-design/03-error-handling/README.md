# 6.3 Error Handling

## Overview

This sub-lesson covers standard error response formats, error code categories, and HTTP status code usage in insurance APIs. Proper error handling is crucial for API usability and debugging.

---

## Standard Error Response

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

---

## Error Code Categories

| Code Range | Category | Examples |
|------------|----------|----------|
| 1xxx | Validation Errors | 1001: Missing required field |
| 2xxx | Authentication/Authorization | 2001: Invalid token |
| 3xxx | Business Rule Violations | 3001: Policy not active |
| 4xxx | External Service Errors | 4001: Payment gateway timeout |
| 5xxx | System Errors | 5001: Database connection failed |

### Validation Errors (1xxx)

```json
{
  "error": {
    "code": "1001",
    "message": "Missing required field",
    "details": [
      {
        "field": "customer.email",
        "code": "REQUIRED",
        "message": "Email address is required"
      }
    ]
  }
}
```

### Authentication Errors (2xxx)

```json
{
  "error": {
    "code": "2001",
    "message": "Invalid token",
    "details": [
      {
        "code": "TOKEN_EXPIRED",
        "message": "The access token has expired. Please refresh your token."
      }
    ]
  }
}
```

### Business Rule Violations (3xxx)

```json
{
  "error": {
    "code": "3001",
    "message": "Policy not active",
    "details": [
      {
        "code": "INVALID_POLICY_STATUS",
        "message": "Cannot process claim for policy with status CANCELLED",
        "current_status": "CANCELLED",
        "allowed_statuses": ["ACTIVE", "GRACE_PERIOD"]
      }
    ]
  }
}
```

### External Service Errors (4xxx)

```json
{
  "error": {
    "code": "4001",
    "message": "Payment gateway timeout",
    "details": [
      {
        "code": "GATEWAY_TIMEOUT",
        "message": "Payment provider did not respond within expected time",
        "retry_after": 30
      }
    ]
  }
}
```

### System Errors (5xxx)

```json
{
  "error": {
    "code": "5001",
    "message": "Internal server error",
    "details": [
      {
        "code": "DATABASE_ERROR",
        "message": "An unexpected error occurred. Please try again later."
      }
    ],
    "correlation_id": "corr_abc123"
  }
}
```

---

## HTTP Status Codes

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

### Status Code Decision Tree

```
Is the request authenticated?
├── No → 401 Unauthorized
└── Yes → Does user have permission?
    ├── No → 403 Forbidden
    └── Yes → Is the request valid?
        ├── No → 400 Bad Request
        └── Yes → Does the resource exist?
            ├── No → 404 Not Found
            └── Yes → Are business rules satisfied?
                ├── No → 422 Unprocessable Entity
                └── Yes → Did operation succeed?
                    ├── No → 500/502/503
                    └── Yes → 200/201/204
```

---

## Error Handling Best Practices

### 1. Provide Actionable Messages

```json
{
  "error": {
    "code": "QUOTE_EXPIRED",
    "message": "This quote has expired",
    "details": [
      {
        "code": "EXPIRED",
        "message": "Quote QT-2024-001234 expired on 2024-01-30",
        "suggested_action": "Create a new quote using POST /api/v1/quotes"
      }
    ]
  }
}
```

### 2. Include Correlation IDs

Always include correlation IDs for tracing:

```json
{
  "error": {
    "code": "SYSTEM_ERROR",
    "message": "An error occurred processing your request",
    "correlation_id": "corr_abc123",
    "request_id": "req_xyz789",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### 3. Rate Limiting Response

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1705315200

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "details": [
      {
        "limit": 100,
        "window": "1 hour",
        "retry_after": 60
      }
    ]
  }
}
```

---

## Key Takeaways

1. Use consistent error response structure across all endpoints
2. Categorize errors with meaningful code ranges
3. Provide actionable error messages that help developers resolve issues
4. Include correlation IDs for debugging and support
5. Use appropriate HTTP status codes for different error types

---

[Previous: Request/Response Patterns](../02-request-response-patterns/README.md) | [Back to Lesson 06](../README.md) | [Next: Pagination & Filtering](../04-pagination-filtering/README.md)
