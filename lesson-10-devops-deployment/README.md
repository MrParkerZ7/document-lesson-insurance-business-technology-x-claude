# Lesson 10: DevOps & Deployment

## Overview

This lesson covers essential DevOps practices and deployment strategies for insurance systems. You will learn how to design CI/CD pipelines, implement infrastructure as code, configure monitoring and observability, and apply deployment strategies for zero-downtime releases.

## Learning Objectives

By the end of this lesson, you will be able to:
- Design CI/CD pipelines for insurance systems
- Implement infrastructure as code
- Configure monitoring and observability
- Apply deployment strategies for zero-downtime releases

---

## Sub-Lessons

### [10.1 CI/CD Pipeline](./10.1-cicd-pipeline/README.md)
Learn how to design and implement continuous integration and continuous deployment pipelines using GitHub Actions, including build, test, security scanning, and deployment stages.

### [10.2 Infrastructure as Code](./10.2-infrastructure-as-code/README.md)
Explore Kubernetes deployments and Terraform configurations for provisioning cloud infrastructure, including EKS clusters, RDS databases, ElastiCache, and MSK Kafka.

### [10.3 Environment Strategy](./10.3-environment-strategy/README.md)
Understand environment management from development through production, including data handling, access controls, and deployment triggers for each environment.

### [10.4 Monitoring & Observability](./10.4-monitoring-observability/README.md)
Master the observability stack including metrics (Prometheus/Grafana), logging (ELK/Loki), distributed tracing (Jaeger/Zipkin), and alerting (PagerDuty/OpsGenie).

### [10.5 Deployment Strategies](./10.5-deployment-strategies/README.md)
Learn deployment strategies including Blue-Green and Canary deployments for zero-downtime releases with instant rollback capabilities.

---

## Key Takeaways

1. **CI/CD pipelines** automate build, test, and deployment
2. **Infrastructure as Code** enables reproducible environments
3. **Multiple environments** support different stages of development
4. **Observability** provides visibility into system behavior
5. **Deployment strategies** enable zero-downtime releases

---

## Exercises

1. Create a GitHub Actions pipeline for claims service
2. Write Terraform modules for RDS and ElastiCache
3. Configure Prometheus alerts for SLA monitoring

---

## Navigation

[Previous Lesson: Testing Strategies](../lesson-09-testing-strategies/README.md) | [Next Lesson: Product Owner Guide](../lesson-11-product-owner-guide/README.md)
