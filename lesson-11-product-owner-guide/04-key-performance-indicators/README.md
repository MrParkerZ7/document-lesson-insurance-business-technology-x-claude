# 11.4 Key Performance Indicators (KPIs)

## Overview

KPIs are essential for measuring the success of insurance technology products. This sub-lesson covers comprehensive metrics across sales, policy operations, claims, and technical performance, enabling Product Owners to track outcomes and make data-driven decisions.

---

## Sales and Distribution KPIs

Track the effectiveness of your sales and distribution channels:

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Quote to Policy Conversion** | Policies / Quotes x 100 | > 25% | Daily |
| **Average Quote Time** | Sum(Quote Duration) / Count | < 2 min | Daily |
| **Digital Sales %** | Digital Policies / Total x 100 | > 40% | Monthly |
| **Agent Productivity** | Policies per Agent | > 20/month | Monthly |
| **Quote Abandonment Rate** | Abandoned / Started x 100 | < 30% | Weekly |

### Understanding Sales KPIs

**Quote to Policy Conversion**
- Measures effectiveness of the quote-to-purchase journey
- Low conversion may indicate pricing issues, UX problems, or competitive disadvantage
- Track by channel, product, and customer segment

**Quote Abandonment Rate**
- Identifies friction points in the quote process
- Analyze at which step users abandon
- Compare across devices and user types

---

## Policy Operations KPIs

Measure operational efficiency in policy administration:

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Policy Issuance SLA** | Policies issued within SLA / Total | > 95% | Daily |
| **Endorsement Turnaround** | Avg time to process endorsement | < 4 hours | Daily |
| **Renewal Rate** | Renewed / Expiring x 100 | > 85% | Monthly |
| **Auto-Renewal Success** | Successful / Attempted x 100 | > 90% | Monthly |
| **Policy Error Rate** | Policies with errors / Total | < 0.5% | Weekly |

### Understanding Operations KPIs

**Policy Issuance SLA**
- Critical for customer satisfaction
- Measures system reliability and process efficiency
- Break down by product type and channel

**Renewal Rate**
- Key indicator of customer satisfaction and loyalty
- Analyze reasons for non-renewal
- Track early vs. late renewal patterns

---

## Claims KPIs

Claims are a moment of truth for insurance customers:

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **Claims Cycle Time** | FNOL to Settlement Duration | < 5 days | Weekly |
| **First Contact Resolution** | Resolved first contact / Total | > 70% | Weekly |
| **Claims Approval Rate** | Approved / Total x 100 | > 85% | Monthly |
| **STP Rate** | Auto-processed / Total x 100 | > 50% | Weekly |
| **Fraud Detection Rate** | Fraud caught / Total fraud | > 80% | Monthly |
| **Customer Satisfaction (Claims)** | CSAT Score | > 4.2/5 | Monthly |

### Understanding Claims KPIs

**Straight-Through Processing (STP) Rate**
- Measures automation effectiveness
- Higher STP reduces operational costs
- Balance with fraud detection accuracy

**Claims Cycle Time**
- Break down into sub-stages:
  - FNOL to Assignment
  - Assignment to Assessment
  - Assessment to Decision
  - Decision to Payment

```
Claims Cycle Time Breakdown:
+--------+  +----------+  +----------+  +--------+  +---------+
|  FNOL  |->|Assignment|->|Assessment|->|Decision|->| Payment |
+--------+  +----------+  +----------+  +--------+  +---------+
   1 day      0.5 day       2 days       1 day       0.5 day
```

---

## Technical KPIs

Ensure your systems perform reliably:

| Metric | Formula | Target | Frequency |
|--------|---------|--------|-----------|
| **System Uptime** | Uptime / Total Time x 100 | > 99.9% | Daily |
| **API Response Time** | p95 latency | < 200ms | Hourly |
| **Error Rate** | Errors / Requests x 100 | < 0.1% | Hourly |
| **Deployment Frequency** | Deployments per week | > 2 | Weekly |
| **Lead Time for Changes** | Commit to production | < 1 day | Weekly |

### Understanding Technical KPIs

**System Uptime**
- 99.9% = 8.76 hours downtime per year
- 99.95% = 4.38 hours downtime per year
- 99.99% = 52.6 minutes downtime per year

**API Response Time**
- p95 = 95th percentile (95% of requests are faster)
- More meaningful than average
- Track by endpoint and operation type

---

## KPI Dashboard Design

### Executive Dashboard

```
+-----------------------------------------------------------------------+
|                        EXECUTIVE KPI DASHBOARD                          |
+-----------------------------------------------------------------------+
|                                                                         |
|  SALES FUNNEL                    CLAIMS PERFORMANCE                     |
|  +-------------------------+     +-------------------------+            |
|  | Quotes Today:    1,234 |     | Open Claims:       456 |            |
|  | Conversion Rate:  28% |     | Avg Cycle Time:  4.2d |            |
|  | GWP MTD:      $2.5M   |     | STP Rate:         58% |            |
|  +-------------------------+     +-------------------------+            |
|                                                                         |
|  OPERATIONS                      SYSTEM HEALTH                          |
|  +-------------------------+     +-------------------------+            |
|  | Policies Issued:   892 |     | Uptime:         99.95% |            |
|  | Pending:           45  |     | API p95:         185ms |            |
|  | Error Rate:      0.3%  |     | Errors:          0.05% |            |
|  +-------------------------+     +-------------------------+            |
|                                                                         |
+-----------------------------------------------------------------------+
```

### Operational Dashboard

Include trend charts and drill-down capabilities:

| Section | Metrics | Visualization |
|---------|---------|---------------|
| **Conversion Funnel** | Quotes > Applications > Policies | Funnel chart |
| **Claims Pipeline** | By status, by age | Kanban/Pipeline |
| **SLA Compliance** | By process, by team | Heat map |
| **System Performance** | Response times, errors | Time series |

---

## Setting KPI Targets

### SMART Criteria

| Criterion | Example |
|-----------|---------|
| **Specific** | Quote conversion rate, not "sales performance" |
| **Measurable** | 25% conversion, not "improved" |
| **Achievable** | Based on industry benchmarks and current baseline |
| **Relevant** | Aligned with business objectives |
| **Time-bound** | Monthly target, quarterly review |

### Benchmark Sources

- Industry reports (e.g., insurance technology benchmarks)
- Competitor analysis
- Historical data (year-over-year comparison)
- Regulatory requirements

---

## KPI Review Cadence

| Timeframe | Focus | Attendees |
|-----------|-------|-----------|
| **Daily** | Operational metrics, exceptions | Operations team |
| **Weekly** | Trend analysis, SLA compliance | Team leads |
| **Monthly** | Strategic KPIs, business outcomes | Leadership |
| **Quarterly** | Target review, goal adjustment | Executive team |

### Monthly KPI Review Agenda

1. **Executive Summary** (5 min)
   - Overall performance vs. targets
   - Key highlights and concerns

2. **Deep Dive by Area** (30 min)
   - Sales and distribution metrics
   - Operations performance
   - Claims analysis
   - Technical health

3. **Root Cause Analysis** (15 min)
   - Investigate misses
   - Identify improvement opportunities

4. **Action Planning** (10 min)
   - Assign improvement initiatives
   - Set next period targets

---

## Key Takeaways

1. KPIs should cover sales, operations, claims, and technical dimensions
2. Set SMART targets based on benchmarks and baselines
3. Use dashboards for real-time visibility
4. Regular review cadence ensures continuous improvement
5. Focus on actionable metrics that drive decisions

---

[Previous: Feature Prioritization](../03-feature-prioritization/README.md) | [Back to Lesson 11](../README.md) | [Next: Roadmap Planning](../05-roadmap-planning/README.md)
