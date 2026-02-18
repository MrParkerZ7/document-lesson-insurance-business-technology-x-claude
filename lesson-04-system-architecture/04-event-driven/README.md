# 04 - Event-Driven Architecture

## Learning Objectives

- Understand event-driven architecture patterns
- Learn Apache Kafka integration for insurance systems
- Design event schemas and topic structures

---

## Event Flow

```
+-----------------------------------------------------------------------------+
|                       EVENT-DRIVEN ARCHITECTURE                              |
+-----------------------------------------------------------------------------+
|                                                                              |
|  PUBLISHERS                      MESSAGE BROKER            SUBSCRIBERS       |
|                                                                              |
|  +-------------+                +-------------+                              |
|  |   Quote     | ---------------|             |                              |
|  |   Service   |  QuoteCreated  |             |                              |
|  +-------------+                |             |         +-------------+      |
|                                 |             | --------|Notification |      |
|  +-------------+                |   KAFKA/    |         |   Service   |      |
|  |   Policy    | ---------------|  RabbitMQ   |         +-------------+      |
|  |   Service   | PolicyIssued   |             |                              |
|  +-------------+                |             |         +-------------+      |
|                                 |             | --------| Analytics   |      |
|  +-------------+                |             |         |   Service   |      |
|  |   Claims    | ---------------|             |         +-------------+      |
|  |   Service   | ClaimSettled   |             |                              |
|  +-------------+                |             |         +-------------+      |
|                                 |             | --------|   Audit     |      |
|  +-------------+                |             |         |   Service   |      |
|  |   Payment   | ---------------|             |         +-------------+      |
|  |   Service   | PaymentReceived|             |                              |
|  +-------------+                +-------------+                              |
|                                                                              |
+-----------------------------------------------------------------------------+
```

---

## Event Schema

### Standard Event Structure

```json
{
  "event_id": "evt_abc123def456",
  "event_type": "policy.issued",
  "event_version": "1.0",
  "timestamp": "2024-01-15T10:30:00Z",
  "source": "policy-service",
  "correlation_id": "corr_xyz789",
  "data": {
    "policy_id": "POL-2024-001234",
    "policy_number": "INS-2024-001234",
    "customer_id": "CUST-001",
    "product_code": "MOTOR_COMP",
    "effective_date": "2024-01-15",
    "expiry_date": "2025-01-14",
    "premium_amount": 1441.00,
    "currency": "USD"
  },
  "metadata": {
    "user_id": "agent_001",
    "channel": "WEB_PORTAL",
    "trace_id": "trace_123"
  }
}
```

---

## Event Topics

| Topic | Publishers | Subscribers |
|-------|------------|-------------|
| `quote.events` | Quote Service | Notification, Analytics |
| `policy.events` | Policy Service | Notification, Billing, Analytics |
| `claim.events` | Claims Service | Notification, Payment, Analytics |
| `payment.events` | Payment Service | Policy, Claims, Notification |
| `customer.events` | Customer Service | All Services |

---

## Kafka Configuration

### Topic Configuration

```yaml
# Kafka Topic Configuration
topics:
  - name: policy.events
    partitions: 12
    replication_factor: 3
    retention_ms: 604800000  # 7 days
    config:
      cleanup.policy: delete
      min.insync.replicas: 2

  - name: claim.events
    partitions: 6
    replication_factor: 3
    retention_ms: 2592000000  # 30 days
    config:
      cleanup.policy: delete
      min.insync.replicas: 2
```

### Producer Configuration

```yaml
# Spring Kafka Producer Configuration
spring:
  kafka:
    producer:
      bootstrap-servers: kafka-broker:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
      properties:
        enable.idempotence: true
        max.in.flight.requests.per.connection: 5
```

### Consumer Configuration

```yaml
# Spring Kafka Consumer Configuration
spring:
  kafka:
    consumer:
      bootstrap-servers: kafka-broker:9092
      group-id: notification-service
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: com.insurance.events
```

---

## Event Processing Patterns

### Event Sourcing

```
+-----------------------------------------------+
|              EVENT SOURCING                    |
+-----------------------------------------------+
|                                               |
|  Commands -> Events -> State                  |
|                                               |
|  +-------------+     +----------------+       |
|  | IssuPolicy  | --> | PolicyIssued   |       |
|  | Command     |     | Event          |       |
|  +-------------+     +----------------+       |
|                             |                 |
|                             v                 |
|                      +-------------+          |
|                      | Event Store |          |
|                      +-------------+          |
|                             |                 |
|                             v                 |
|                      +-------------+          |
|                      | Policy      |          |
|                      | Projection  |          |
|                      +-------------+          |
|                                               |
+-----------------------------------------------+
```

### CQRS Pattern

```
+-----------------------------------------------+
|                 CQRS PATTERN                   |
+-----------------------------------------------+
|                                               |
|  WRITE SIDE              READ SIDE            |
|                                               |
|  +----------+           +----------+          |
|  | Commands |           | Queries  |          |
|  +----------+           +----------+          |
|       |                      |                |
|       v                      v                |
|  +----------+           +----------+          |
|  | Command  |           | Query    |          |
|  | Handlers |           | Handlers |          |
|  +----------+           +----------+          |
|       |                      |                |
|       v                      v                |
|  +----------+           +----------+          |
|  | Write DB |  Events   | Read DB  |          |
|  | (Source) | --------> | (Views)  |          |
|  +----------+           +----------+          |
|                                               |
+-----------------------------------------------+
```

---

## Best Practices

1. **Idempotency**: Design consumers to handle duplicate events
2. **Schema Evolution**: Use versioned schemas with backward compatibility
3. **Dead Letter Queues**: Handle failed event processing gracefully
4. **Monitoring**: Track event lag, throughput, and error rates
5. **Partitioning**: Partition by entity ID for ordering guarantees

---

## Navigation

[Previous: Technology Stack](../03-technology-stack/README.md) | [Back to Lesson 04](../README.md)
