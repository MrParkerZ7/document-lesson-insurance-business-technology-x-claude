# 10.5 Deployment Strategies

## Overview

Deployment strategies determine how new versions of applications are released to production. This sub-lesson covers Blue-Green and Canary deployment strategies, which enable zero-downtime releases with rollback capabilities.

---

## Blue-Green Deployment

Blue-Green deployment maintains two identical production environments. At any time, only one environment serves live traffic while the other is idle or used for testing.

### How It Works

```
+-----------------------------------------------------------------------------+
|                       BLUE-GREEN DEPLOYMENT                                  |
+-----------------------------------------------------------------------------+
|                                                                              |
|  STEP 1: Initial State                                                       |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |     Users --> Load Balancer --> BLUE (v1.0) [Active]                  |  |
|  |                                                                        |  |
|  |                                  GREEN (v1.1) [Idle]                   |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  STEP 2: Deploy to Green                                                     |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |     Users --> Load Balancer --> BLUE (v1.0) [Active]                  |  |
|  |                                                                        |  |
|  |                                  GREEN (v1.1) [Deploying...]           |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  STEP 3: Test Green                                                          |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |     Users --> Load Balancer --> BLUE (v1.0) [Active]                  |  |
|  |                                                                        |  |
|  |     QA --->                     GREEN (v1.1) [Testing]                |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  STEP 4: Switch Traffic                                                      |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                                  BLUE (v1.0) [Standby]                 |  |
|  |                                                                        |  |
|  |     Users --> Load Balancer --> GREEN (v1.1) [Active]                 |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  ROLLBACK: Instant switch back to Blue if issues detected                   |
|                                                                              |
+-----------------------------------------------------------------------------+
```

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Zero Downtime** | Traffic switch is instantaneous |
| **Instant Rollback** | Simply route traffic back to previous environment |
| **Full Testing** | New version can be fully tested before going live |
| **Reduced Risk** | Production environment unchanged until switch |

### Disadvantages

| Disadvantage | Description |
|--------------|-------------|
| **Cost** | Requires double the infrastructure |
| **Database Migrations** | Complex when schema changes are involved |
| **State Management** | Sessions and caches need careful handling |

### Implementation with Kubernetes

```yaml
# Switch traffic by updating service selector
kubectl patch service policy-service -n insurance-prod \
  -p '{"spec":{"selector":{"version":"green"}}}'
```

---

## Canary Deployment

Canary deployment gradually rolls out changes to a small subset of users before rolling out to the entire infrastructure.

### How It Works

```
+-----------------------------------------------------------------------------+
|                        CANARY DEPLOYMENT                                     |
+-----------------------------------------------------------------------------+
|                                                                              |
|  PHASE 1: 5% Traffic to Canary                                              |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                         +--> Stable (v1.0) [95%]                      |  |
|  |     Users --> LB -------+                                              |  |
|  |                         +--> Canary (v1.1) [5%]                       |  |
|  |                                                                        |  |
|  |     Monitor: Errors, Latency, Business Metrics                        |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  PHASE 2: 25% Traffic (if healthy)                                          |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                         +--> Stable (v1.0) [75%]                      |  |
|  |     Users --> LB -------+                                              |  |
|  |                         +--> Canary (v1.1) [25%]                      |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  PHASE 3: 50% Traffic                                                        |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                         +--> Stable (v1.0) [50%]                      |  |
|  |     Users --> LB -------+                                              |  |
|  |                         +--> Canary (v1.1) [50%]                      |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
|  PHASE 4: 100% - Full Rollout                                               |
|  +-----------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |     Users --> LB -----------> New Stable (v1.1) [100%]                |  |
|  |                                                                        |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
+-----------------------------------------------------------------------------+
```

### Traffic Progression

| Phase | Canary Traffic | Duration | Criteria to Proceed |
|-------|----------------|----------|---------------------|
| 1 | 5% | 30 minutes | Error rate < 0.5%, p95 latency < 200ms |
| 2 | 25% | 1 hour | Error rate < 0.5%, p95 latency < 200ms |
| 3 | 50% | 2 hours | Error rate < 0.5%, p95 latency < 200ms |
| 4 | 100% | - | Full rollout |

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Gradual Rollout** | Issues affect only a subset of users |
| **Real Traffic Testing** | New version tested with real production traffic |
| **Metrics-Based Decisions** | Promotion based on actual performance data |
| **Early Detection** | Problems discovered before full rollout |

### Disadvantages

| Disadvantage | Description |
|--------------|-------------|
| **Complexity** | More complex to implement and monitor |
| **Slower Rollout** | Takes longer to complete deployment |
| **Monitoring Required** | Needs robust monitoring to detect issues |

---

## Comparison: Blue-Green vs Canary

| Aspect | Blue-Green | Canary |
|--------|------------|--------|
| **Traffic Switch** | All at once | Gradual |
| **Rollback Speed** | Instant | Requires scale down |
| **Infrastructure Cost** | 2x resources | 1x + canary pods |
| **Risk Exposure** | Binary (all or nothing) | Gradual |
| **Monitoring Needs** | Basic | Advanced |
| **Suitable For** | Simple applications | Complex, high-traffic systems |

---

## Rollback Strategies

### Automatic Rollback Triggers

```yaml
# Argo Rollouts configuration example
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: policy-service
spec:
  strategy:
    canary:
      steps:
      - setWeight: 5
      - pause: {duration: 30m}
      - setWeight: 25
      - pause: {duration: 1h}
      - setWeight: 50
      - pause: {duration: 2h}
      - setWeight: 100
      analysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: policy-service
```

### Rollback Criteria

| Metric | Threshold | Action |
|--------|-----------|--------|
| Error Rate | > 1% | Automatic rollback |
| p95 Latency | > 500ms | Automatic rollback |
| Health Check Failures | > 3 consecutive | Automatic rollback |
| Business Metric Drop | > 10% | Manual review |

---

## Best Practices

1. **Feature Flags**: Combine with feature flags for safer releases
2. **Automated Analysis**: Use automated canary analysis tools
3. **Database Compatibility**: Ensure backward-compatible schema changes
4. **Session Management**: Handle user sessions during deployments
5. **Monitoring Integration**: Integrate deployment events with monitoring
6. **Documentation**: Document rollback procedures and runbooks

---

## Navigation

[Previous: 10.4 Monitoring & Observability](../04-monitoring-observability/README.md) | [Back to Lesson 10](../README.md)
