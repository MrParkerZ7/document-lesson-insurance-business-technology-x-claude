# 10.4 Monitoring & Observability

## Overview

Observability is essential for understanding the behavior of distributed systems. This sub-lesson covers the three pillars of observability (metrics, logs, traces) along with alerting strategies for insurance platform operations.

---

## Observability Stack

```
+-----------------------------------------------------------------------------+
|                          OBSERVABILITY STACK                                 |
+-----------------------------------------------------------------------------+
|                                                                              |
|  METRICS (Prometheus + Grafana)                                              |
|  +-----------------------------------------------------------------------+  |
|  |  - Request rate, latency, errors (RED metrics)                        |  |
|  |  - JVM metrics (heap, GC, threads)                                    |  |
|  |  - Business metrics (quotes/day, claims/day)                          |  |
|  |  - Infrastructure metrics (CPU, memory, disk)                         |  |
|  |  - Custom dashboards per service                                      |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  LOGGING (ELK Stack / Loki)                                                  |
|  +-----------------------------------------------------------------------+  |
|  |  - Structured JSON logging                                            |  |
|  |  - Correlation IDs for request tracing                                |  |
|  |  - Log aggregation and search                                         |  |
|  |  - Log retention policies (30 days hot, 1 year cold)                 |  |
|  |  - Alert on error patterns                                            |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  TRACING (Jaeger / Zipkin)                                                   |
|  +-----------------------------------------------------------------------+  |
|  |  - Distributed tracing across services                                |  |
|  |  - Service dependency mapping                                         |  |
|  |  - Latency analysis and bottleneck detection                         |  |
|  |  - Error tracking through request flow                                |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  ALERTING (PagerDuty / OpsGenie)                                             |
|  +-----------------------------------------------------------------------+  |
|  |  - SLA breach alerts (response time > 500ms)                         |  |
|  |  - Error rate thresholds (> 1% errors)                               |  |
|  |  - Infrastructure alerts (disk > 80%)                                |  |
|  |  - On-call rotation                                                  |  |
|  |  - Escalation policies                                                |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
+-----------------------------------------------------------------------------+
```

---

## Metrics: Prometheus & Grafana

### Grafana Dashboard Configuration

```yaml
# grafana/dashboards/policy-service.json
{
  "dashboard": {
    "title": "Policy Service",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_server_requests_seconds_count{service=\"policy-service\"}[5m]))",
            "legendFormat": "Requests/sec"
          }
        ]
      },
      {
        "title": "Response Time (p95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket{service=\"policy-service\"}[5m])) by (le))",
            "legendFormat": "p95 Latency"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_server_requests_seconds_count{service=\"policy-service\",status=~\"5..\"}[5m])) / sum(rate(http_server_requests_seconds_count{service=\"policy-service\"}[5m])) * 100",
            "legendFormat": "Error %"
          }
        ]
      },
      {
        "title": "Business Metrics",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(increase(policies_issued_total[24h]))",
            "legendFormat": "Policies Issued (24h)"
          },
          {
            "expr": "sum(increase(claims_registered_total[24h]))",
            "legendFormat": "Claims Registered (24h)"
          }
        ]
      }
    ]
  }
}
```

### Key Metrics Categories

| Category | Metrics | Purpose |
|----------|---------|---------|
| **RED Metrics** | Rate, Errors, Duration | Service health indicators |
| **USE Metrics** | Utilization, Saturation, Errors | Resource health |
| **Business Metrics** | Quotes, Policies, Claims | Business KPIs |
| **SLA Metrics** | Availability, Response Time | SLA compliance |

---

## Alerting Rules

### Prometheus Alert Configuration

```yaml
# prometheus/alerts/policy-service.yml
groups:
  - name: policy-service
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_server_requests_seconds_count{service="policy-service",status=~"5.."}[5m]))
          / sum(rate(http_server_requests_seconds_count{service="policy-service"}[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: High error rate on Policy Service
          description: Error rate is {{ $value | humanizePercentage }}

      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket{service="policy-service"}[5m])) by (le)) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: High latency on Policy Service
          description: p95 latency is {{ $value | humanizeDuration }}

      - alert: PodNotReady
        expr: |
          kube_pod_status_ready{namespace="insurance-platform",pod=~"policy-service.*"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Policy Service pod not ready
          description: Pod {{ $labels.pod }} is not ready
```

### Alert Severity Levels

| Severity | Response Time | Examples | Notification |
|----------|--------------|----------|--------------|
| **Critical** | Immediate | Service down, data loss risk | Page on-call, SMS |
| **High** | < 30 minutes | High error rate, SLA breach | Slack + Email |
| **Warning** | < 4 hours | Elevated latency, disk space | Slack |
| **Info** | Next business day | Deployment notifications | Email |

---

## Logging Best Practices

### Structured Logging Format

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "service": "policy-service",
  "traceId": "abc123def456",
  "spanId": "789ghi",
  "userId": "user-12345",
  "policyId": "POL-2024-001",
  "message": "Policy created successfully",
  "duration_ms": 150,
  "environment": "production"
}
```

### Log Levels Usage

| Level | Usage | Examples |
|-------|-------|----------|
| **ERROR** | Failures requiring attention | Exceptions, failed transactions |
| **WARN** | Potential issues | Retry attempts, deprecation warnings |
| **INFO** | Business events | Policy created, claim submitted |
| **DEBUG** | Development details | Method entry/exit, variable values |
| **TRACE** | Detailed debugging | Full request/response bodies |

---

## Distributed Tracing

### Trace Propagation

```
[API Gateway] ---> [Policy Service] ---> [Database]
     |                    |
     |                    +---> [Kafka] ---> [Notification Service]
     |
     +---> [Claims Service] ---> [Document Service]
```

### Key Tracing Concepts

- **Trace**: End-to-end journey of a request
- **Span**: Single operation within a trace
- **Context Propagation**: Passing trace IDs across service boundaries
- **Sampling**: Percentage of traces to capture (typically 1-10%)

---

## On-Call Best Practices

### Runbook Template

```markdown
## Alert: HighErrorRate on Policy Service

### Impact
- Customer impact: Unable to create new policies
- Business impact: Revenue loss, customer dissatisfaction

### Investigation Steps
1. Check Grafana dashboard for error patterns
2. Review recent deployments
3. Check dependent service health
4. Review application logs for error details

### Mitigation Steps
1. If recent deployment: Initiate rollback
2. If database issue: Check connection pool, failover status
3. If traffic spike: Scale up pods manually

### Escalation
- Level 1: DevOps on-call (15 min)
- Level 2: Service owner (30 min)
- Level 3: Engineering manager (1 hour)
```

---

## Best Practices

1. **Correlation IDs**: Use trace IDs across all logs and metrics
2. **SLO-Based Alerting**: Alert on SLO breaches, not individual metrics
3. **Alert Fatigue Prevention**: Tune thresholds, reduce noise
4. **Dashboard Hierarchy**: Overview -> Service -> Component
5. **Log Retention**: Balance cost with debugging needs
6. **Sampling Strategy**: Sample traces at appropriate rates

---

## Navigation

[Previous: 10.3 Environment Strategy](../03-environment-strategy/README.md) | [Back to Lesson 10](../README.md) | [Next: 10.5 Deployment Strategies](../05-deployment-strategies/README.md)
