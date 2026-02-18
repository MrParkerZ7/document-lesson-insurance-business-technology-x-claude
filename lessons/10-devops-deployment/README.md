# Lesson 10: DevOps & Deployment

## Learning Objectives

By the end of this lesson, you will be able to:
- Design CI/CD pipelines for insurance systems
- Implement infrastructure as code
- Configure monitoring and observability
- Apply deployment strategies for zero-downtime releases

---

## 10.1 CI/CD Pipeline

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CI/CD PIPELINE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │   Code   │ → │  Build   │ → │   Test   │ → │ Security │ → │ Artifact │  │
│  │  Commit  │   │          │   │          │   │   Scan   │   │  Publish │  │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
│       │              │              │              │              │         │
│       ↓              ↓              ↓              ↓              ↓         │
│    Git Push      Compile        Unit Tests    SAST/DAST      Push to       │
│    Lint          Package        Integration   Dependency     Registry      │
│    Format        Docker Build   Contract      Secrets Scan                  │
│                                                                              │
│  ════════════════════════════════════════════════════════════════════════  │
│                                DEPLOYMENT                                    │
│  ════════════════════════════════════════════════════════════════════════  │
│                                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │  Deploy  │ → │  Smoke   │ → │   QA     │ → │ Staging  │ → │   Prod   │  │
│  │   DEV    │   │  Tests   │   │          │   │          │   │  Deploy  │  │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘  │
│       │              │              │              │              │         │
│       ↓              ↓              ↓              ↓              ↓         │
│    Auto          Health         Manual         Performance   Blue/Green   │
│    Deploy        Checks         Testing        Testing       Canary        │
│                  API Tests                     UAT           Rollback      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Build with Maven
        run: mvn clean compile -B

      - name: Run Unit Tests
        run: mvn test -B

      - name: Run Integration Tests
        run: mvn verify -P integration-tests -B

      - name: Code Coverage
        run: mvn jacoco:report

      - name: Upload Coverage to Codecov
        uses: codecov/codecov-action@v3

  security-scan:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run SAST with SonarQube
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

      - name: Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'insurance-platform'
          path: '.'
          format: 'HTML'

      - name: Container Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'

  publish:
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

  deploy-staging:
    runs-on: ubuntu-latest
    needs: publish
    environment: staging
    steps:
      - name: Deploy to Staging
        uses: azure/k8s-deploy@v4
        with:
          namespace: insurance-staging
          manifests: |
            k8s/staging/deployment.yaml
            k8s/staging/service.yaml
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

      - name: Run Smoke Tests
        run: |
          ./scripts/smoke-tests.sh $STAGING_URL

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production
    steps:
      - name: Deploy to Production (Blue/Green)
        uses: azure/k8s-deploy@v4
        with:
          namespace: insurance-prod
          strategy: blue-green
          manifests: |
            k8s/prod/deployment.yaml
            k8s/prod/service.yaml
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

      - name: Verify Production Health
        run: |
          ./scripts/health-check.sh $PROD_URL

      - name: Promote to Production
        if: success()
        run: |
          kubectl patch service policy-service -n insurance-prod \
            -p '{"spec":{"selector":{"version":"${{ github.sha }}"}}}'
```

---

## 10.2 Infrastructure as Code

### Kubernetes Deployment

```yaml
# k8s/policy-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: policy-service
  namespace: insurance-platform
  labels:
    app: policy-service
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: policy-service
  template:
    metadata:
      labels:
        app: policy-service
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      serviceAccountName: policy-service
      containers:
      - name: policy-service
        image: ghcr.io/insurance/policy-service:latest
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 8080
        - name: management
          containerPort: 8081
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        - name: KAFKA_BROKERS
          valueFrom:
            configMapKeyRef:
              name: kafka-config
              key: brokers
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: management
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: management
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: policy-service-config
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: policy-service
              topologyKey: kubernetes.io/hostname
---
apiVersion: v1
kind: Service
metadata:
  name: policy-service
  namespace: insurance-platform
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 8080
  selector:
    app: policy-service
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: policy-service-hpa
  namespace: insurance-platform
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: policy-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Terraform Infrastructure

```hcl
# terraform/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }

  backend "s3" {
    bucket = "insurance-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}

# EKS Cluster
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "insurance-platform"
  cluster_version = "1.28"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      desired_size = 3
      min_size     = 2
      max_size     = 10

      instance_types = ["m5.large"]
      capacity_type  = "ON_DEMAND"

      labels = {
        Environment = var.environment
      }
    }
  }
}

# RDS PostgreSQL
module "rds" {
  source  = "terraform-aws-modules/rds/aws"
  version = "~> 6.0"

  identifier = "insurance-db"

  engine               = "postgres"
  engine_version       = "15.4"
  family               = "postgres15"
  major_engine_version = "15"
  instance_class       = "db.r5.large"

  allocated_storage     = 100
  max_allocated_storage = 500

  db_name  = "insurance"
  username = "admin"
  port     = 5432

  multi_az               = true
  db_subnet_group_name   = module.vpc.database_subnet_group_name
  vpc_security_group_ids = [module.security_group.security_group_id]

  backup_retention_period = 30
  deletion_protection     = true

  performance_insights_enabled = true
  monitoring_interval          = 60
}

# ElastiCache Redis
module "elasticache" {
  source  = "terraform-aws-modules/elasticache/aws"
  version = "~> 1.0"

  cluster_id           = "insurance-cache"
  engine               = "redis"
  node_type            = "cache.r5.large"
  num_cache_nodes      = 2
  parameter_group_name = "default.redis7"
  port                 = 6379
  subnet_group_name    = module.vpc.elasticache_subnet_group_name
}

# MSK Kafka
module "msk" {
  source  = "terraform-aws-modules/msk-kafka-cluster/aws"
  version = "~> 2.0"

  name           = "insurance-kafka"
  kafka_version  = "3.5.1"
  number_of_broker_nodes = 3

  broker_node_instance_type   = "kafka.m5.large"
  broker_node_ebs_volume_size = 100

  broker_node_client_subnets  = module.vpc.private_subnets
  broker_node_security_groups = [module.security_group.security_group_id]

  encryption_in_transit_client_broker = "TLS"
  encryption_in_transit_in_cluster    = true
}
```

---

## 10.3 Environment Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ENVIRONMENT STRATEGY                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ENVIRONMENT     PURPOSE              DATA           ACCESS                  │
│  ───────────────────────────────────────────────────────────────────────    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  DEVELOPMENT                                                         │   │
│  │  - Feature development                                               │   │
│  │  - Synthetic test data                                               │   │
│  │  - Developer access only                                             │   │
│  │  - Auto-deployed on commit                                           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                              │
│                               ↓                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  INTEGRATION                                                         │   │
│  │  - Service integration testing                                       │   │
│  │  - Synthetic data                                                    │   │
│  │  - Dev team access                                                   │   │
│  │  - Auto-deployed on PR merge                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                              │
│                               ↓                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  QA / TEST                                                           │   │
│  │  - Functional testing                                                │   │
│  │  - Masked production data                                            │   │
│  │  - QA team access                                                    │   │
│  │  - Manual deployment trigger                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                              │
│                               ↓                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  STAGING / UAT                                                       │   │
│  │  - Pre-production validation                                         │   │
│  │  - Masked production data                                            │   │
│  │  - Production-like infrastructure                                    │   │
│  │  - Business user access                                              │   │
│  │  - Approval required for deployment                                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                               │                                              │
│                               ↓                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  PRODUCTION                                                          │   │
│  │  - Live system                                                       │   │
│  │  - Real customer data                                                │   │
│  │  - Restricted access                                                 │   │
│  │  - Blue/Green or Canary deployment                                  │   │
│  │  - Change approval required                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10.4 Monitoring & Observability

### Observability Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OBSERVABILITY STACK                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  METRICS (Prometheus + Grafana)                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Request rate, latency, errors (RED metrics)                       │    │
│  │  - JVM metrics (heap, GC, threads)                                   │    │
│  │  - Business metrics (quotes/day, claims/day)                         │    │
│  │  - Infrastructure metrics (CPU, memory, disk)                        │    │
│  │  - Custom dashboards per service                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  LOGGING (ELK Stack / Loki)                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Structured JSON logging                                           │    │
│  │  - Correlation IDs for request tracing                               │    │
│  │  - Log aggregation and search                                        │    │
│  │  - Log retention policies (30 days hot, 1 year cold)                │    │
│  │  - Alert on error patterns                                           │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  TRACING (Jaeger / Zipkin)                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - Distributed tracing across services                               │    │
│  │  - Service dependency mapping                                        │    │
│  │  - Latency analysis and bottleneck detection                        │    │
│  │  - Error tracking through request flow                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ALERTING (PagerDuty / OpsGenie)                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  - SLA breach alerts (response time > 500ms)                        │    │
│  │  - Error rate thresholds (> 1% errors)                              │    │
│  │  - Infrastructure alerts (disk > 80%)                               │    │
│  │  - On-call rotation                                                 │    │
│  │  - Escalation policies                                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Grafana Dashboard

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

### Alert Rules

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

---

## 10.5 Deployment Strategies

### Blue-Green Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BLUE-GREEN DEPLOYMENT                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STEP 1: Initial State                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │     Users ──→ Load Balancer ──→ BLUE (v1.0) ✓ Active               │    │
│  │                                                                      │    │
│  │                                  GREEN (v1.1) Idle                   │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  STEP 2: Deploy to Green                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │     Users ──→ Load Balancer ──→ BLUE (v1.0) ✓ Active               │    │
│  │                                                                      │    │
│  │                                  GREEN (v1.1) Deploying...          │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  STEP 3: Test Green                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │     Users ──→ Load Balancer ──→ BLUE (v1.0) ✓ Active               │    │
│  │                                                                      │    │
│  │     QA ───→                     GREEN (v1.1) Testing ✓              │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  STEP 4: Switch Traffic                                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │                                  BLUE (v1.0) Standby                │    │
│  │                                                                      │    │
│  │     Users ──→ Load Balancer ──→ GREEN (v1.1) ✓ Active              │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ROLLBACK: Instant switch back to Blue if issues detected                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Canary Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CANARY DEPLOYMENT                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: 5% Traffic to Canary                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │                         ┌──→ Stable (v1.0) [95%]                    │    │
│  │     Users ──→ LB ───────┤                                           │    │
│  │                         └──→ Canary (v1.1) [5%]                     │    │
│  │                                                                      │    │
│  │     Monitor: Errors, Latency, Business Metrics                      │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  PHASE 2: 25% Traffic (if healthy)                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │                         ┌──→ Stable (v1.0) [75%]                    │    │
│  │     Users ──→ LB ───────┤                                           │    │
│  │                         └──→ Canary (v1.1) [25%]                    │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  PHASE 3: 50% Traffic                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │                         ┌──→ Stable (v1.0) [50%]                    │    │
│  │     Users ──→ LB ───────┤                                           │    │
│  │                         └──→ Canary (v1.1) [50%]                    │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  PHASE 4: 100% - Full Rollout                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │     Users ──→ LB ──────────→ New Stable (v1.1) [100%]               │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

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

[← Previous Lesson](../09-testing-strategies/README.md) | [Next Lesson: Product Owner Guide →](../11-product-owner-guide/README.md)
