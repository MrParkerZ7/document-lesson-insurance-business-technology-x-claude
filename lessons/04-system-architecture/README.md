# Lesson 04: System Architecture

## Learning Objectives

By the end of this lesson, you will be able to:
- Design high-level architecture for insurance systems
- Understand microservices patterns for insurance
- Select appropriate technology stacks
- Implement API gateway and service mesh patterns

---

## 4.1 High-Level Architecture

### Enterprise Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              PRESENTATION LAYER                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │  Mobile  │  │   Web    │  │  Agent   │  │  Partner │  │  Admin   │      │
│  │   App    │  │  Portal  │  │  Portal  │  │   API    │  │  Console │      │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                               API GATEWAY                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│  Authentication │ Rate Limiting │ Routing │ Load Balancing │ Logging       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SERVICE LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │  Product  │ │   Quote   │ │  Policy   │ │  Claims   │ │  Payment  │     │
│  │  Service  │ │  Service  │ │  Service  │ │  Service  │ │  Service  │     │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘     │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │ Customer  │ │Underwriting│ │ Document │ │Notification│ │ Reporting │     │
│  │  Service  │ │  Service  │ │  Service │ │  Service  │ │  Service  │     │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INTEGRATION LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │  Message  │ │   Event   │ │   File    │ │    ETL    │ │   Batch   │     │
│  │   Queue   │ │   Bus     │ │  Transfer │ │           │ │  Process  │     │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                               DATA LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │ Relational│ │  Document │ │   Cache   │ │   Data    │ │   Data    │     │
│  │    DB     │ │   Store   │ │   Layer   │ │   Lake    │ │ Warehouse │     │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘ └───────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                           EXTERNAL INTEGRATIONS                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  Payment Gateway │ KYC/AML │ Reinsurance │ Regulatory │ Third-Party APIs   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Microservices Architecture

### Service Topology

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MICROSERVICES TOPOLOGY                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         API GATEWAY                                  │    │
│  │           (Kong / AWS API Gateway / Azure APIM)                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│            ┌───────────────────────┼───────────────────────┐                │
│            ↓                       ↓                       ↓                │
│     ┌─────────────┐         ┌─────────────┐         ┌─────────────┐        │
│     │   Quote     │         │   Policy    │         │   Claims    │        │
│     │   Service   │ ──────→ │   Service   │ ←────── │   Service   │        │
│     │             │         │             │         │             │        │
│     │ - Calculate │         │ - Issue     │         │ - Register  │        │
│     │ - Compare   │         │ - Endorse   │         │ - Assess    │        │
│     │ - Save      │         │ - Renew     │         │ - Settle    │        │
│     └─────────────┘         └─────────────┘         └─────────────┘        │
│            │                       │                       │                │
│            └───────────────────────┼───────────────────────┘                │
│                                    ↓                                         │
│                          ┌─────────────────┐                                │
│                          │    Customer     │                                │
│                          │    Service      │                                │
│                          │                 │                                │
│                          │ - Profile       │                                │
│                          │ - KYC           │                                │
│                          │ - Communication │                                │
│                          └─────────────────┘                                │
│                                    │                                         │
│            ┌───────────────────────┼───────────────────────┐                │
│            ↓                       ↓                       ↓                │
│     ┌─────────────┐         ┌─────────────┐         ┌─────────────┐        │
│     │   Product   │         │   Payment   │         │  Document   │        │
│     │   Service   │         │   Service   │         │   Service   │        │
│     └─────────────┘         └─────────────┘         └─────────────┘        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                       MESSAGE BROKER                                 │    │
│  │           (Apache Kafka / RabbitMQ / AWS SNS+SQS)                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Service Responsibilities

| Service | Responsibilities | Database | Events Published |
|---------|-----------------|----------|------------------|
| **Quote** | Premium calculation, quote management | PostgreSQL | QuoteCreated, QuoteExpired |
| **Policy** | Policy lifecycle, endorsements | PostgreSQL | PolicyIssued, PolicyRenewed |
| **Claims** | Claim registration, processing | PostgreSQL | ClaimRegistered, ClaimSettled |
| **Customer** | Customer profiles, KYC | PostgreSQL | CustomerCreated, KYCCompleted |
| **Product** | Product catalog, rating rules | PostgreSQL | ProductUpdated |
| **Payment** | Payment processing, billing | PostgreSQL | PaymentReceived, PaymentFailed |
| **Document** | Document storage, generation | MongoDB + S3 | DocumentGenerated |
| **Notification** | Email, SMS, push notifications | Redis | - |
| **Underwriting** | Risk assessment, rules engine | PostgreSQL | RiskAssessed |

---

## 4.3 Technology Stack

### Recommended Stack

| Layer | Technology Options | Recommendation |
|-------|-------------------|----------------|
| **Frontend** | React, Angular, Vue.js, Flutter | React + React Native |
| **Backend** | Java/Spring Boot, Node.js, .NET Core, Go | Java/Spring Boot |
| **API Gateway** | Kong, AWS API Gateway, Apigee, Azure APIM | Kong |
| **Message Queue** | Apache Kafka, RabbitMQ, AWS SQS | Apache Kafka |
| **Database** | PostgreSQL, MySQL, Oracle | PostgreSQL |
| **Document Store** | MongoDB, Couchbase | MongoDB |
| **Cache** | Redis, Memcached | Redis |
| **Search** | Elasticsearch, OpenSearch | Elasticsearch |
| **Container** | Docker, Kubernetes, ECS | Kubernetes |
| **Cloud** | AWS, Azure, GCP | AWS / Azure |
| **CI/CD** | Jenkins, GitLab CI, GitHub Actions | GitHub Actions |
| **Monitoring** | Prometheus, Grafana, Datadog | Prometheus + Grafana |
| **Logging** | ELK Stack, Loki, Splunk | ELK Stack |
| **Tracing** | Jaeger, Zipkin, X-Ray | Jaeger |

### Database Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    POLYGLOT PERSISTENCE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐                                            │
│  │   PostgreSQL    │ ← Transactional data                       │
│  │                 │   (Policies, Claims, Customers)            │
│  └─────────────────┘                                            │
│                                                                  │
│  ┌─────────────────┐                                            │
│  │    MongoDB      │ ← Flexible schemas                         │
│  │                 │   (Documents, Audit logs, Quotes)          │
│  └─────────────────┘                                            │
│                                                                  │
│  ┌─────────────────┐                                            │
│  │     Redis       │ ← Caching, Sessions                        │
│  │                 │   (Rate tables, User sessions)             │
│  └─────────────────┘                                            │
│                                                                  │
│  ┌─────────────────┐                                            │
│  │  Elasticsearch  │ ← Full-text search                         │
│  │                 │   (Policy search, Claims search)           │
│  └─────────────────┘                                            │
│                                                                  │
│  ┌─────────────────┐                                            │
│  │    S3/Blob      │ ← File storage                             │
│  │                 │   (Policy documents, Claim photos)         │
│  └─────────────────┘                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.4 API Gateway Pattern

### Gateway Responsibilities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Request   │  │   Auth &    │  │    Rate     │  │   Request   │        │
│  │   Routing   │  │   AuthZ     │  │   Limiting  │  │   Transform │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │    Load     │  │   Circuit   │  │   Request   │  │   Response  │        │
│  │   Balancing │  │   Breaker   │  │   Logging   │  │   Caching   │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │    SSL      │  │   Request   │  │    API      │  │   Health    │        │
│  │ Termination │  │  Validation │  │  Versioning │  │   Checks    │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Rate Limiting Configuration

```yaml
# Kong Rate Limiting Example
plugins:
  - name: rate-limiting
    config:
      minute: 100
      hour: 1000
      policy: local

  - name: rate-limiting
    service: quote-service
    config:
      minute: 50    # Lower limit for expensive operations

  - name: rate-limiting
    consumer: partner-api
    config:
      minute: 500   # Higher limit for partners
```

---

## 4.5 Event-Driven Architecture

### Event Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       EVENT-DRIVEN ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PUBLISHERS                      MESSAGE BROKER            SUBSCRIBERS       │
│                                                                              │
│  ┌─────────────┐                ┌─────────────┐                             │
│  │   Quote     │ ───────────────│             │                             │
│  │   Service   │  QuoteCreated  │             │                             │
│  └─────────────┘                │             │         ┌─────────────┐     │
│                                 │             │ ────────│ Notification│     │
│  ┌─────────────┐                │   KAFKA/    │         │   Service   │     │
│  │   Policy    │ ───────────────│  RabbitMQ   │         └─────────────┘     │
│  │   Service   │ PolicyIssued   │             │                             │
│  └─────────────┘                │             │         ┌─────────────┐     │
│                                 │             │ ────────│  Analytics  │     │
│  ┌─────────────┐                │             │         │   Service   │     │
│  │   Claims    │ ───────────────│             │         └─────────────┘     │
│  │   Service   │ ClaimSettled   │             │                             │
│  └─────────────┘                │             │         ┌─────────────┐     │
│                                 │             │ ────────│   Audit     │     │
│  ┌─────────────┐                │             │         │   Service   │     │
│  │   Payment   │ ───────────────│             │         └─────────────┘     │
│  │   Service   │ PaymentReceived│             │                             │
│  └─────────────┘                └─────────────┘                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Event Schema

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

### Event Topics

| Topic | Publishers | Subscribers |
|-------|------------|-------------|
| `quote.events` | Quote Service | Notification, Analytics |
| `policy.events` | Policy Service | Notification, Billing, Analytics |
| `claim.events` | Claims Service | Notification, Payment, Analytics |
| `payment.events` | Payment Service | Policy, Claims, Notification |
| `customer.events` | Customer Service | All Services |

---

## 4.6 Infrastructure Patterns

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: policy-service
  namespace: insurance-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: policy-service
  template:
    metadata:
      labels:
        app: policy-service
    spec:
      containers:
      - name: policy-service
        image: insurance/policy-service:v1.2.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
        - name: KAFKA_BROKERS
          value: "kafka-broker:9092"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### Service Mesh (Istio)

```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVICE MESH (ISTIO)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    ISTIO CONTROL PLANE                   │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │    │
│  │  │  Pilot  │  │ Citadel │  │  Galley │  │  Mixer  │    │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│                              ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    DATA PLANE (Envoy Sidecars)          │    │
│  │                                                          │    │
│  │  ┌──────────────┐      ┌──────────────┐                │    │
│  │  │ Policy Svc   │      │ Claims Svc   │                │    │
│  │  │ ┌──────────┐ │      │ ┌──────────┐ │                │    │
│  │  │ │  Envoy   │ │ ←──→ │ │  Envoy   │ │                │    │
│  │  │ └──────────┘ │      │ └──────────┘ │                │    │
│  │  └──────────────┘      └──────────────┘                │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Benefits:                                                       │
│  - mTLS encryption                                               │
│  - Traffic management                                            │
│  - Circuit breaking                                              │
│  - Observability                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

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

[← Previous Lesson](../03-core-processes/README.md) | [Next Lesson: Data Models →](../05-data-models/README.md)
