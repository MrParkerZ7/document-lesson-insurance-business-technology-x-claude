# 01 - Enterprise Architecture Overview

## Learning Objectives

- Understand the layered architecture approach for insurance systems
- Learn API Gateway patterns and responsibilities
- Understand how different layers interact

---

## High-Level Architecture

### Enterprise Architecture Layers

```
+-----------------------------------------------------------------------------+
|                              PRESENTATION LAYER                              |
+-----------------------------------------------------------------------------+
|  +----------+  +----------+  +----------+  +----------+  +----------+      |
|  |  Mobile  |  |   Web    |  |  Agent   |  |  Partner |  |  Admin   |      |
|  |   App    |  |  Portal  |  |  Portal  |  |   API    |  |  Console |      |
|  +----------+  +----------+  +----------+  +----------+  +----------+      |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                               API GATEWAY                                    |
+-----------------------------------------------------------------------------+
|  Authentication | Rate Limiting | Routing | Load Balancing | Logging       |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                              SERVICE LAYER                                   |
+-----------------------------------------------------------------------------+
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
|  |  Product  | |   Quote   | |  Policy   | |  Claims   | |  Payment  |     |
|  |  Service  | |  Service  | |  Service  | |  Service  | |  Service  |     |
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
|  | Customer  | |Underwriting| | Document | |Notification| | Reporting |     |
|  |  Service  | |  Service  | |  Service | |  Service  | |  Service  |     |
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                           INTEGRATION LAYER                                  |
+-----------------------------------------------------------------------------+
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
|  |  Message  | |   Event   | |   File    | |    ETL    | |   Batch   |     |
|  |   Queue   | |   Bus     | |  Transfer | |           | |  Process  |     |
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                               DATA LAYER                                     |
+-----------------------------------------------------------------------------+
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
|  | Relational| |  Document | |   Cache   | |   Data    | |   Data    |     |
|  |    DB     | |   Store   | |   Layer   | |   Lake    | | Warehouse |     |
|  +-----------+ +-----------+ +-----------+ +-----------+ +-----------+     |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                           EXTERNAL INTEGRATIONS                              |
+-----------------------------------------------------------------------------+
|  Payment Gateway | KYC/AML | Reinsurance | Regulatory | Third-Party APIs   |
+-----------------------------------------------------------------------------+
```

---

## API Gateway Pattern

### Gateway Responsibilities

```
+-----------------------------------------------------------------------------+
|                           API GATEWAY                                        |
+-----------------------------------------------------------------------------+
|                                                                              |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|  |   Request   |  |   Auth &    |  |    Rate     |  |   Request   |        |
|  |   Routing   |  |   AuthZ     |  |   Limiting  |  |   Transform |        |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|                                                                              |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|  |    Load     |  |   Circuit   |  |   Request   |  |   Response  |        |
|  |   Balancing |  |   Breaker   |  |   Logging   |  |   Caching   |        |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|                                                                              |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|  |    SSL      |  |   Request   |  |    API      |  |   Health    |        |
|  | Termination |  |  Validation |  |  Versioning |  |   Checks    |        |
|  +-------------+  +-------------+  +-------------+  +-------------+        |
|                                                                              |
+-----------------------------------------------------------------------------+
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

## Layer Responsibilities

| Layer | Purpose | Components |
|-------|---------|------------|
| **Presentation** | User interfaces | Mobile apps, Web portals, Admin consoles |
| **API Gateway** | Traffic management | Authentication, Rate limiting, Routing |
| **Service** | Business logic | Core microservices |
| **Integration** | Communication | Message queues, Event bus, ETL |
| **Data** | Persistence | Databases, Caches, Data warehouses |
| **External** | Third-party | Payment gateways, KYC providers |

---

## Navigation

[Back to Lesson 04](../README.md) | [Next: Microservices](../02-microservices/README.md)
