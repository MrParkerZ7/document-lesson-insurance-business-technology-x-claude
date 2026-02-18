# 6.1 RESTful API Design

## Overview

This sub-lesson covers the fundamentals of designing RESTful APIs for insurance systems, including endpoint structure, resource naming conventions, and HTTP method usage.

---

## API Endpoints Structure

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

## REST Design Principles

### Resource Naming Conventions

1. **Use nouns, not verbs** - Resources represent entities
   - Good: `/policies`, `/claims`
   - Bad: `/getPolicies`, `/createClaim`

2. **Use plural nouns** - Collections should be plural
   - Good: `/customers/{id}`
   - Bad: `/customer/{id}`

3. **Use hierarchical relationships** - Sub-resources show relationships
   - `/customers/{id}/policies` - Policies belonging to a customer
   - `/policies/{id}/claims` - Claims on a specific policy

### HTTP Methods

| Method | Operation | Idempotent | Safe |
|--------|-----------|------------|------|
| GET | Read resource(s) | Yes | Yes |
| POST | Create resource or trigger action | No | No |
| PUT | Update resource (full) | Yes | No |
| PATCH | Update resource (partial) | No | No |
| DELETE | Remove resource | Yes | No |

### URL Design Best Practices

1. **Version your API** - Include version in URL
   ```
   /api/v1/policies
   /api/v2/policies
   ```

2. **Use query parameters for filtering**
   ```
   /api/v1/policies?status=ACTIVE&product_code=MOTOR
   ```

3. **Use action endpoints for operations**
   ```
   POST /api/v1/policies/{id}/cancel
   POST /api/v1/claims/{id}/approve
   ```

---

## Diagram

See the API endpoints structure diagram: [06.1-api-endpoints-structure.drawio](./06.1-api-endpoints-structure.drawio)

---

## Key Takeaways

1. Follow consistent naming conventions for all endpoints
2. Use appropriate HTTP methods for each operation
3. Design hierarchical URLs to represent resource relationships
4. Version your APIs from the start

---

[Back to Lesson 06](../README.md) | [Next: Request/Response Patterns](../02-request-response-patterns/README.md)
