# 02 - Microservices Architecture

## Learning Objectives

- Understand microservices topology for insurance systems
- Learn service responsibilities and boundaries
- Understand inter-service communication patterns

---

## Service Topology

```
+-----------------------------------------------------------------------------+
|                       MICROSERVICES TOPOLOGY                                 |
+-----------------------------------------------------------------------------+
|                                                                              |
|  +---------------------------------------------------------------------+    |
|  |                         API GATEWAY                                  |    |
|  |           (Kong / AWS API Gateway / Azure APIM)                      |    |
|  +---------------------------------------------------------------------+    |
|                                    |                                         |
|            +------------------- ---+-------------------+                     |
|            v                       v                   v                     |
|     +-------------+         +-------------+         +-------------+          |
|     |   Quote     |         |   Policy    |         |   Claims    |          |
|     |   Service   | ------> |   Service   | <------ |   Service   |          |
|     |             |         |             |         |             |          |
|     | - Calculate |         | - Issue     |         | - Register  |          |
|     | - Compare   |         | - Endorse   |         | - Assess    |          |
|     | - Save      |         | - Renew     |         | - Settle    |          |
|     +-------------+         +-------------+         +-------------+          |
|            |                       |                       |                 |
|            +-----------------------+-------------------  --+                 |
|                                    v                                         |
|                          +-----------------+                                 |
|                          |    Customer     |                                 |
|                          |    Service      |                                 |
|                          |                 |                                 |
|                          | - Profile       |                                 |
|                          | - KYC           |                                 |
|                          | - Communication |                                 |
|                          +-----------------+                                 |
|                                    |                                         |
|            +-----------------------+-------------------+                     |
|            v                       v                   v                     |
|     +-------------+         +-------------+         +-------------+          |
|     |   Product   |         |   Payment   |         |  Document   |          |
|     |   Service   |         |   Service   |         |   Service   |          |
|     +-------------+         +-------------+         +-------------+          |
|                                                                              |
|  +---------------------------------------------------------------------+    |
|  |                       MESSAGE BROKER                                 |    |
|  |           (Apache Kafka / RabbitMQ / AWS SNS+SQS)                   |    |
|  +---------------------------------------------------------------------+    |
|                                                                              |
+-----------------------------------------------------------------------------+
```

---

## Service Responsibilities

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

## Infrastructure Patterns

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

---

### Service Mesh (Istio)

```
+---------------------------------------------------------------+
|                    SERVICE MESH (ISTIO)                        |
+---------------------------------------------------------------+
|                                                                |
|  +---------------------------------------------------------+  |
|  |                    ISTIO CONTROL PLANE                   |  |
|  |  +---------+  +---------+  +---------+  +---------+     |  |
|  |  |  Pilot  |  | Citadel |  |  Galley |  |  Mixer  |     |  |
|  |  +---------+  +---------+  +---------+  +---------+     |  |
|  +---------------------------------------------------------+  |
|                              |                                 |
|                              v                                 |
|  +---------------------------------------------------------+  |
|  |                    DATA PLANE (Envoy Sidecars)          |  |
|  |                                                          |  |
|  |  +--------------+      +--------------+                 |  |
|  |  | Policy Svc   |      | Claims Svc   |                 |  |
|  |  | +----------+ |      | +----------+ |                 |  |
|  |  | |  Envoy   | | <--> | |  Envoy   | |                 |  |
|  |  | +----------+ |      | +----------+ |                 |  |
|  |  +--------------+      +--------------+                 |  |
|  |                                                          |  |
|  +---------------------------------------------------------+  |
|                                                                |
|  Benefits:                                                     |
|  - mTLS encryption                                             |
|  - Traffic management                                          |
|  - Circuit breaking                                            |
|  - Observability                                               |
|                                                                |
+---------------------------------------------------------------+
```

---

## Architecture Diagram

See the detailed architecture diagram: [04.1-microservices-architecture.drawio](./04.1-microservices-architecture.drawio)

---

## Navigation

[Previous: Architecture Overview](../01-architecture-overview/README.md) | [Back to Lesson 04](../README.md) | [Next: Technology Stack](../03-technology-stack/README.md)
