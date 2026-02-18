# 6.4 Pagination & Filtering

## Overview

This sub-lesson covers how to implement efficient pagination and filtering mechanisms for API endpoints handling large datasets in insurance systems.

---

## Pagination

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

### Pagination Parameters

| Parameter | Description | Default | Max |
|-----------|-------------|---------|-----|
| `page` | Page number (1-indexed) | 1 | - |
| `size` | Items per page | 20 | 100 |
| `sort` | Sort field and direction | varies | - |

### Cursor-Based Pagination

For large datasets, cursor-based pagination is more efficient:

```http
GET /api/v1/claims?cursor=eyJpZCI6IkNMTS0yMDI0LTAwMTAwMCJ9&limit=50
```

```json
{
  "data": [...],
  "pagination": {
    "limit": 50,
    "has_more": true,
    "next_cursor": "eyJpZCI6IkNMTS0yMDI0LTAwMTA1MCJ9",
    "previous_cursor": "eyJpZCI6IkNMTS0yMDI0LTAwMDk1MCJ9"
  }
}
```

---

## Filtering

### Filter by Status

```http
GET /api/v1/policies?status=ACTIVE
```

### Filter by Date Range

```http
GET /api/v1/claims?loss_date_from=2024-01-01&loss_date_to=2024-01-31
```

### Filter by Multiple Values

```http
GET /api/v1/policies?status=ACTIVE,LAPSED
```

### Search

```http
GET /api/v1/customers?search=john.doe@email.com
```

### Complex Filter

```http
GET /api/v1/claims?status=OPEN&priority=HIGH&adjuster_id=ADJ-001
```

---

## Common Filter Patterns

### Policy Filters

| Filter | Example | Description |
|--------|---------|-------------|
| `status` | `?status=ACTIVE` | Filter by policy status |
| `product_code` | `?product_code=MOTOR_COMP` | Filter by product |
| `customer_id` | `?customer_id=cust_123` | Filter by customer |
| `effective_date_from` | `?effective_date_from=2024-01-01` | Policies effective after date |
| `effective_date_to` | `?effective_date_to=2024-12-31` | Policies effective before date |
| `expiry_date_from` | `?expiry_date_from=2024-01-01` | Policies expiring after date |
| `expiry_date_to` | `?expiry_date_to=2024-12-31` | Policies expiring before date |

### Claim Filters

| Filter | Example | Description |
|--------|---------|-------------|
| `status` | `?status=OPEN,UNDER_ASSESSMENT` | Filter by claim status |
| `priority` | `?priority=HIGH` | Filter by priority |
| `adjuster_id` | `?adjuster_id=ADJ-001` | Filter by assigned adjuster |
| `loss_date_from` | `?loss_date_from=2024-01-01` | Claims with loss date after |
| `loss_date_to` | `?loss_date_to=2024-01-31` | Claims with loss date before |
| `claimed_amount_min` | `?claimed_amount_min=1000` | Minimum claimed amount |
| `claimed_amount_max` | `?claimed_amount_max=50000` | Maximum claimed amount |

---

## Sorting

### Sort Parameters

```http
# Single field sort
GET /api/v1/policies?sort=created_at,desc

# Multiple field sort
GET /api/v1/claims?sort=priority,desc&sort=created_at,asc
```

### Sortable Fields

| Resource | Sortable Fields |
|----------|-----------------|
| Policies | `created_at`, `effective_date`, `expiry_date`, `premium_amount` |
| Claims | `created_at`, `loss_date`, `claimed_amount`, `priority`, `status` |
| Quotes | `created_at`, `valid_until`, `total_premium` |
| Customers | `created_at`, `last_name`, `first_name` |

---

## Field Selection

Allow clients to request only specific fields:

```http
GET /api/v1/policies?fields=policy_id,policy_number,status,premium_amount
```

```json
{
  "data": [
    {
      "policy_id": "POL-2024-001234",
      "policy_number": "INS-2024-001234",
      "status": "ACTIVE",
      "premium_amount": 1185.50
    }
  ],
  "pagination": {...}
}
```

---

## Response Headers

Include pagination metadata in response headers:

```http
HTTP/1.1 200 OK
X-Total-Count: 156
X-Total-Pages: 8
X-Page: 1
X-Page-Size: 20
Link: </api/v1/policies?page=2&size=20>; rel="next",
      </api/v1/policies?page=8&size=20>; rel="last"
```

---

## Key Takeaways

1. Use offset-based pagination for simple use cases
2. Use cursor-based pagination for large datasets
3. Provide consistent filtering parameters across similar resources
4. Support multiple sort fields with direction
5. Include HATEOAS links for navigation

---

[Previous: Error Handling](../03-error-handling/README.md) | [Back to Lesson 06](../README.md) | [Next: Event Schema Design](../05-event-schema-design/README.md)
