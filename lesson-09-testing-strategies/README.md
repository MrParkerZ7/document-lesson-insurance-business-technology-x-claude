# Lesson 09: Testing Strategies

## Learning Objectives

By the end of this lesson, you will be able to:
- Design comprehensive test strategies for insurance systems
- Implement unit, integration, and E2E tests
- Manage test data effectively
- Apply testing best practices for insurance domain

---

## Sub-Lessons

### [01 - Testing Pyramid](./01-testing-pyramid/README.md)
Understanding the testing pyramid concept, test distribution ratios, and when to use each type of test.

### [02 - Unit Testing](./02-unit-testing/README.md)
Writing effective unit tests for insurance domain logic including premium calculations and policy state machines.

### [03 - Integration Testing](./03-integration-testing/README.md)
Testing service interactions with databases, event publishers, and other dependencies using test containers.

### [04 - End-to-End Testing](./04-end-to-end-testing/README.md)
Implementing API E2E tests and complete user journey tests for insurance workflows.

### [05 - Test Data Management](./05-test-data-management/README.md)
Creating test fixtures, factories, and managing test data effectively.

### [06 - Testing Best Practices](./06-testing-best-practices/README.md)
Insurance-specific test scenarios and coverage guidelines.

---

## Key Takeaways

1. **Testing pyramid** guides test distribution (many unit, few E2E)
2. **Fixtures and factories** enable consistent test data
3. **Integration tests** verify component interactions
4. **E2E tests** validate complete user journeys
5. **Domain-specific scenarios** ensure business logic correctness

---

## Exercises

1. Write unit tests for claim reserve calculation
2. Create integration tests for policy renewal workflow
3. Design E2E test for quote-to-policy conversion with payment

---

[Previous Lesson](../lesson-08-development-practices/README.md) | [Next Lesson: DevOps & Deployment](../lesson-10-devops-deployment/README.md)
