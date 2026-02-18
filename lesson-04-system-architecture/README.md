# Lesson 04: System Architecture

## Learning Objectives

By the end of this lesson, you will be able to:
- Design high-level architecture for insurance systems
- Understand microservices patterns for insurance
- Select appropriate technology stacks
- Implement API gateway and service mesh patterns
- Apply event-driven architecture principles

---

## Sub-Lessons

| Sub-Lesson | Topic | Description |
|------------|-------|-------------|
| [01-architecture-overview](./01-architecture-overview/README.md) | Enterprise Architecture | High-level architecture layers, API gateway patterns |
| [02-microservices](./02-microservices/README.md) | Microservices Architecture | Service topology, responsibilities, and patterns |
| [03-technology-stack](./03-technology-stack/README.md) | Technology Stack | Recommended technologies, database strategy |
| [04-event-driven](./04-event-driven/README.md) | Event-Driven Architecture | Event flow, Kafka integration, event schemas |

---

## Key Takeaways

1. **Layered architecture** separates concerns (presentation, service, data)
2. **Microservices** enable independent deployment and scaling
3. **API Gateway** centralizes cross-cutting concerns
4. **Event-driven** architecture enables loose coupling
5. **Polyglot persistence** uses the right database for each use case

---

## Exercises

1. Design a microservices architecture for a health insurance platform
2. Create a Kafka topic structure for policy events
3. Configure rate limiting rules for different API consumers

---

[Previous Lesson](../lesson-03-core-processes/README.md) | [Next Lesson](../lesson-05-data-models/README.md)
