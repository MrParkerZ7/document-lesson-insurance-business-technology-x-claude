# 10.3 Environment Strategy

## Overview

A well-defined environment strategy is crucial for maintaining quality and reducing risk in software delivery. This sub-lesson covers the different environments used in the software development lifecycle, their purposes, data handling, and access controls.

---

## Environment Overview

```
+-----------------------------------------------------------------------------+
|                          ENVIRONMENT STRATEGY                                |
+-----------------------------------------------------------------------------+
|                                                                              |
|  ENVIRONMENT     PURPOSE              DATA           ACCESS                  |
|  ---------------------------------------------------------------------------|
|                                                                              |
|  +-----------------------------------------------------------------------+  |
|  |  DEVELOPMENT                                                          |  |
|  |  - Feature development                                                |  |
|  |  - Synthetic test data                                                |  |
|  |  - Developer access only                                              |  |
|  |  - Auto-deployed on commit                                            |  |
|  +-----------------------------------------------------------------------+  |
|                               |                                              |
|                               v                                              |
|  +-----------------------------------------------------------------------+  |
|  |  INTEGRATION                                                          |  |
|  |  - Service integration testing                                        |  |
|  |  - Synthetic data                                                     |  |
|  |  - Dev team access                                                    |  |
|  |  - Auto-deployed on PR merge                                          |  |
|  +-----------------------------------------------------------------------+  |
|                               |                                              |
|                               v                                              |
|  +-----------------------------------------------------------------------+  |
|  |  QA / TEST                                                            |  |
|  |  - Functional testing                                                 |  |
|  |  - Masked production data                                             |  |
|  |  - QA team access                                                     |  |
|  |  - Manual deployment trigger                                          |  |
|  +-----------------------------------------------------------------------+  |
|                               |                                              |
|                               v                                              |
|  +-----------------------------------------------------------------------+  |
|  |  STAGING / UAT                                                        |  |
|  |  - Pre-production validation                                          |  |
|  |  - Masked production data                                             |  |
|  |  - Production-like infrastructure                                     |  |
|  |  - Business user access                                               |  |
|  |  - Approval required for deployment                                   |  |
|  +-----------------------------------------------------------------------+  |
|                               |                                              |
|                               v                                              |
|  +-----------------------------------------------------------------------+  |
|  |  PRODUCTION                                                           |  |
|  |  - Live system                                                        |  |
|  |  - Real customer data                                                 |  |
|  |  - Restricted access                                                  |  |
|  |  - Blue/Green or Canary deployment                                   |  |
|  |  - Change approval required                                           |  |
|  +-----------------------------------------------------------------------+  |
|                                                                              |
+-----------------------------------------------------------------------------+
```

---

## Environment Details

### Development Environment

| Attribute | Description |
|-----------|-------------|
| **Purpose** | Individual feature development and initial testing |
| **Data** | Synthetic test data, fixtures |
| **Infrastructure** | Minimal resources, shared or local |
| **Deployment** | Automatic on every commit |
| **Access** | Developers only |
| **Testing** | Unit tests, local integration tests |

### Integration Environment

| Attribute | Description |
|-----------|-------------|
| **Purpose** | Testing service interactions and API contracts |
| **Data** | Synthetic data, test fixtures |
| **Infrastructure** | Shared development cluster |
| **Deployment** | Automatic on PR merge to develop |
| **Access** | Development team |
| **Testing** | Integration tests, contract tests |

### QA / Test Environment

| Attribute | Description |
|-----------|-------------|
| **Purpose** | Functional testing, regression testing |
| **Data** | Masked/anonymized production data |
| **Infrastructure** | Dedicated QA cluster |
| **Deployment** | Manual trigger, scheduled releases |
| **Access** | QA team, test automation |
| **Testing** | Functional tests, regression tests, exploratory testing |

### Staging / UAT Environment

| Attribute | Description |
|-----------|-------------|
| **Purpose** | Pre-production validation, user acceptance testing |
| **Data** | Masked production data, production-like volume |
| **Infrastructure** | Production-equivalent infrastructure |
| **Deployment** | Approval required, release candidate validation |
| **Access** | Business users, product owners, selected stakeholders |
| **Testing** | UAT, performance testing, security testing |

### Production Environment

| Attribute | Description |
|-----------|-------------|
| **Purpose** | Live customer-facing system |
| **Data** | Real customer data |
| **Infrastructure** | Full production scale, high availability |
| **Deployment** | Change approval, deployment windows, Blue/Green or Canary |
| **Access** | Highly restricted, audited access |
| **Testing** | Smoke tests, synthetic monitoring |

---

## Data Management Strategy

### Data Categories

| Category | Development | Integration | QA | Staging | Production |
|----------|-------------|-------------|-----|---------|------------|
| Customer PII | Synthetic | Synthetic | Masked | Masked | Real |
| Financial Data | Synthetic | Synthetic | Masked | Masked | Real |
| System Config | Test | Test | Test | Production-like | Production |
| Reference Data | Subset | Subset | Full | Full | Full |

### Data Masking Rules

```
Original Data          ->    Masked Data
John Smith             ->    Person_12345
john.smith@email.com   ->    user_12345@masked.local
123-45-6789            ->    XXX-XX-1234
4111-1111-1111-1111   ->    XXXX-XXXX-XXXX-1111
$50,000.00            ->    $XX,XXX.00 (or $50,000.00 if non-sensitive)
```

---

## Access Control Matrix

| Role | Development | Integration | QA | Staging | Production |
|------|-------------|-------------|-----|---------|------------|
| Developer | Full | Full | Read | Read | None |
| QA Engineer | Read | Read | Full | Full | Read (logs) |
| DevOps | Full | Full | Full | Full | Full |
| Product Owner | None | Read | Read | Full | Read (metrics) |
| Business User | None | None | None | Read | Read |
| Support | None | None | Read | Read | Read |

---

## Deployment Gates

### Development to Integration
- All unit tests pass
- Code coverage threshold met
- Code review approved
- No critical security vulnerabilities

### Integration to QA
- All integration tests pass
- Contract tests pass
- API documentation updated
- Change request documented

### QA to Staging
- All functional tests pass
- No critical or high bugs open
- Performance baseline established
- Security scan completed

### Staging to Production
- UAT sign-off obtained
- Performance tests pass SLA requirements
- Security approval
- Change approval board sign-off
- Rollback plan documented
- Monitoring alerts configured

---

## Best Practices

1. **Environment Parity**: Keep environments as similar as possible to production
2. **Data Protection**: Never use real customer data in non-production environments
3. **Access Control**: Apply least privilege principle across all environments
4. **Audit Logging**: Log all access and changes in staging and production
5. **Automated Provisioning**: Use IaC to create consistent environments
6. **Regular Refresh**: Periodically refresh non-production data from masked production data

---

## Navigation

[Previous: 10.2 Infrastructure as Code](../02-infrastructure-as-code/README.md) | [Back to Lesson 10](../README.md) | [Next: 10.4 Monitoring & Observability](../04-monitoring-observability/README.md)
