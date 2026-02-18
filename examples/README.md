# Examples

This folder contains code examples and templates for insurance system implementation.

## Structure

```
examples/
├── api-contracts/          # OpenAPI specifications
├── database-schemas/       # SQL schema definitions
└── event-schemas/          # Event/message schemas
```

## API Contracts

OpenAPI 3.0 specifications for insurance APIs:

- `quote-api.yaml` - Quote generation and management
- `policy-api.yaml` - Policy lifecycle operations
- `claims-api.yaml` - Claims processing
- `customer-api.yaml` - Customer management
- `payment-api.yaml` - Payment processing

## Database Schemas

SQL scripts for core insurance tables:

- `customer-schema.sql` - Customer and address tables
- `policy-schema.sql` - Policy, coverage, endorsement tables
- `claims-schema.sql` - Claim, document, payment tables
- `product-schema.sql` - Product, plan, coverage master tables
- `audit-schema.sql` - Audit log tables

## Event Schemas

JSON schemas for event-driven architecture:

- `policy-events.json` - Policy lifecycle events
- `claim-events.json` - Claims processing events
- `payment-events.json` - Payment events
- `customer-events.json` - Customer events

## Usage

These examples are referenced throughout the lessons. Copy and adapt them for your specific implementation needs.
