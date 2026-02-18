# 8.1 Code Organization

## Overview

Proper code organization is fundamental to building maintainable insurance software systems. This sub-lesson covers project structure patterns that promote separation of concerns, scalability, and team collaboration.

---

## Project Structure

A well-organized insurance platform follows a clear hierarchical structure:

```
insurance-platform/
├── services/
│   ├── quote-service/
│   │   ├── src/
│   │   │   ├── api/
│   │   │   │   ├── controllers/
│   │   │   │   │   ├── QuoteController.java
│   │   │   │   │   └── QuoteComparisonController.java
│   │   │   │   ├── dto/
│   │   │   │   │   ├── request/
│   │   │   │   │   │   └── CreateQuoteRequest.java
│   │   │   │   │   └── response/
│   │   │   │   │       └── QuoteResponse.java
│   │   │   │   ├── mappers/
│   │   │   │   │   └── QuoteMapper.java
│   │   │   │   └── validators/
│   │   │   │       └── QuoteRequestValidator.java
│   │   │   │
│   │   │   ├── domain/
│   │   │   │   ├── models/
│   │   │   │   │   ├── Quote.java
│   │   │   │   │   ├── QuoteItem.java
│   │   │   │   │   └── PremiumBreakdown.java
│   │   │   │   ├── services/
│   │   │   │   │   ├── QuoteService.java
│   │   │   │   │   ├── PremiumCalculationService.java
│   │   │   │   │   └── QuoteConversionService.java
│   │   │   │   ├── repositories/
│   │   │   │   │   └── QuoteRepository.java
│   │   │   │   └── events/
│   │   │   │       ├── QuoteCreatedEvent.java
│   │   │   │       └── QuoteExpiredEvent.java
│   │   │   │
│   │   │   ├── infrastructure/
│   │   │   │   ├── persistence/
│   │   │   │   │   ├── entities/
│   │   │   │   │   │   └── QuoteEntity.java
│   │   │   │   │   └── repositories/
│   │   │   │   │       └── JpaQuoteRepository.java
│   │   │   │   ├── messaging/
│   │   │   │   │   ├── publishers/
│   │   │   │   │   │   └── QuoteEventPublisher.java
│   │   │   │   │   └── consumers/
│   │   │   │   │       └── ProductUpdateConsumer.java
│   │   │   │   └── external/
│   │   │   │       ├── ProductServiceClient.java
│   │   │   │       └── CustomerServiceClient.java
│   │   │   │
│   │   │   └── config/
│   │   │       ├── ApplicationConfig.java
│   │   │       ├── SecurityConfig.java
│   │   │       └── KafkaConfig.java
│   │   │
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   ├── integration/
│   │   │   └── e2e/
│   │   │
│   │   ├── resources/
│   │   │   ├── application.yml
│   │   │   └── db/migration/
│   │   │
│   │   ├── Dockerfile
│   │   └── pom.xml
│   │
│   ├── policy-service/
│   ├── claims-service/
│   └── customer-service/
│
├── shared/
│   ├── common-lib/
│   │   ├── exceptions/
│   │   ├── utils/
│   │   └── constants/
│   ├── event-schemas/
│   └── api-contracts/
│
├── infrastructure/
│   ├── terraform/
│   ├── kubernetes/
│   └── scripts/
│
└── docs/
    ├── api/
    ├── architecture/
    └── runbooks/
```

---

## Layer Descriptions

### API Layer (`src/api/`)
- **Controllers**: Handle HTTP requests and responses
- **DTOs**: Data Transfer Objects for request/response mapping
- **Mappers**: Transform between DTOs and domain models
- **Validators**: Input validation logic

### Domain Layer (`src/domain/`)
- **Models**: Core business entities and value objects
- **Services**: Business logic and domain operations
- **Repositories**: Interface definitions for data access
- **Events**: Domain events for inter-service communication

### Infrastructure Layer (`src/infrastructure/`)
- **Persistence**: Database entities and repository implementations
- **Messaging**: Event publishers and consumers
- **External**: Clients for external service integration

### Configuration (`src/config/`)
- Application configuration classes
- Security settings
- External service configurations

---

## Shared Components

The `shared/` directory contains reusable components across services:

| Component | Purpose |
|-----------|---------|
| `common-lib` | Shared utilities, exceptions, and constants |
| `event-schemas` | Message contract definitions |
| `api-contracts` | OpenAPI specifications and client interfaces |

---

## Key Principles

1. **Separation of Concerns**: Each layer has a specific responsibility
2. **Dependency Direction**: Dependencies flow inward (API -> Domain <- Infrastructure)
3. **Domain Independence**: Domain layer has no external dependencies
4. **Testability**: Each layer can be tested independently

---

## Navigation

[Back to Lesson 08](../README.md) | [Next: 8.2 Design Patterns](../02-design-patterns/README.md)
