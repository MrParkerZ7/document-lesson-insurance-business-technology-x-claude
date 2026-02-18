# 01 - Testing Pyramid

[Back to Lesson 09](../README.md)

---

## Overview

The testing pyramid is a fundamental concept that guides how to structure your test suite for optimal coverage and efficiency. It emphasizes having many fast, cheap unit tests at the base, fewer integration tests in the middle, and minimal expensive E2E tests at the top.

---

## Testing Pyramid Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          TESTING PYRAMID                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │       E2E         │  <- Few tests                  │
│                         │      Tests        │    Slow, expensive            │
│                         │    (UI/API)       │    Real dependencies          │
│                         └───────────────────┘                               │
│                                                                              │
│                    ┌─────────────────────────────┐                          │
│                    │       Integration           │  <- Some tests            │
│                    │         Tests               │    Medium speed           │
│                    │  (Service + Dependencies)   │    Test containers        │
│                    └─────────────────────────────┘                          │
│                                                                              │
│              ┌───────────────────────────────────────────┐                  │
│              │              Unit Tests                    │  <- Many tests   │
│              │     (Single class/function in isolation)   │    Fast, cheap  │
│              │                                            │    Mocked deps  │
│              └───────────────────────────────────────────┘                  │
│                                                                              │
│  RATIO: Unit (70%) : Integration (20%) : E2E (10%)                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

> See also: [09.1-testing-pyramid.drawio](./09.1-testing-pyramid.drawio) for the visual diagram.

---

## Test Type Characteristics

| Test Type | Speed | Cost | Scope | Dependencies |
|-----------|-------|------|-------|--------------|
| **Unit Tests** | Fast (ms) | Low | Single class/function | Mocked |
| **Integration Tests** | Medium (seconds) | Medium | Service + dependencies | Test containers |
| **E2E Tests** | Slow (minutes) | High | Complete system | Real services |

---

## Recommended Test Ratio

For insurance systems, follow this distribution:

- **Unit Tests (70%)**: Business logic, calculations, validations
- **Integration Tests (20%)**: Database operations, API calls, event publishing
- **E2E Tests (10%)**: Critical user journeys, happy paths

---

## When to Use Each Test Type

### Unit Tests
- Premium calculations
- Discount/loading logic
- State transitions
- Validation rules
- Domain entity behavior

### Integration Tests
- Repository operations
- Service-to-service communication
- Event publishing/consuming
- External API integrations

### E2E Tests
- Quote-to-policy conversion
- Complete claims journey
- Renewal workflows
- Payment processing

---

## Navigation

[Back to Lesson 09](../README.md) | [Next: Unit Testing](../02-unit-testing/README.md)
