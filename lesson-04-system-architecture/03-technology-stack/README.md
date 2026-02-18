# 03 - Technology Stack

## Learning Objectives

- Understand recommended technology choices for insurance systems
- Learn polyglot persistence strategies
- Select appropriate tools for each layer

---

## Recommended Stack

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

---

## Database Strategy

### Polyglot Persistence

```
+---------------------------------------------------------------+
|                    POLYGLOT PERSISTENCE                        |
+---------------------------------------------------------------+
|                                                                |
|  +-----------------+                                           |
|  |   PostgreSQL    | <- Transactional data                     |
|  |                 |   (Policies, Claims, Customers)           |
|  +-----------------+                                           |
|                                                                |
|  +-----------------+                                           |
|  |    MongoDB      | <- Flexible schemas                       |
|  |                 |   (Documents, Audit logs, Quotes)         |
|  +-----------------+                                           |
|                                                                |
|  +-----------------+                                           |
|  |     Redis       | <- Caching, Sessions                      |
|  |                 |   (Rate tables, User sessions)            |
|  +-----------------+                                           |
|                                                                |
|  +-----------------+                                           |
|  |  Elasticsearch  | <- Full-text search                       |
|  |                 |   (Policy search, Claims search)          |
|  +-----------------+                                           |
|                                                                |
|  +-----------------+                                           |
|  |    S3/Blob      | <- File storage                           |
|  |                 |   (Policy documents, Claim photos)        |
|  +-----------------+                                           |
|                                                                |
+---------------------------------------------------------------+
```

---

## Technology Selection Criteria

### Backend Framework Selection

| Criteria | Java/Spring Boot | Node.js | .NET Core | Go |
|----------|-----------------|---------|-----------|-----|
| **Performance** | Good | Good | Excellent | Excellent |
| **Enterprise Support** | Excellent | Good | Excellent | Good |
| **Developer Availability** | High | High | Medium | Low |
| **Insurance Domain Libs** | Excellent | Limited | Good | Limited |
| **Regulatory Compliance** | Excellent | Good | Excellent | Good |

### Database Selection

| Use Case | Recommended DB | Reason |
|----------|---------------|--------|
| **Transactional Data** | PostgreSQL | ACID compliance, complex queries |
| **Document Storage** | MongoDB | Flexible schema, JSON support |
| **Caching** | Redis | High performance, TTL support |
| **Search** | Elasticsearch | Full-text search, aggregations |
| **File Storage** | S3/Blob | Scalable, cost-effective |
| **Analytics** | Data Warehouse | Historical analysis, reporting |

---

## Infrastructure Components

### Container Orchestration

- **Kubernetes** for production workloads
- **Docker Compose** for local development
- **Helm Charts** for deployment management

### Monitoring Stack

```
+-----------------------------------------------+
|            OBSERVABILITY STACK                 |
+-----------------------------------------------+
|                                               |
|  Metrics:    Prometheus + Grafana            |
|  Logging:    ELK Stack (Elasticsearch,       |
|              Logstash, Kibana)               |
|  Tracing:    Jaeger                          |
|  Alerting:   AlertManager + PagerDuty        |
|                                               |
+-----------------------------------------------+
```

---

## Navigation

[Previous: Microservices](../02-microservices/README.md) | [Back to Lesson 04](../README.md) | [Next: Event-Driven Architecture](../04-event-driven/README.md)
