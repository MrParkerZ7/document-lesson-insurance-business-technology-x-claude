# 10.2 Infrastructure as Code

## Overview

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than manual processes. This sub-lesson covers Kubernetes deployments and Terraform configurations for insurance platform infrastructure.

---

## Kubernetes Deployment

### Policy Service Deployment

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
```

### Service Configuration

```yaml
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
```

### Horizontal Pod Autoscaler

```yaml
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

---

## Terraform Infrastructure

### Main Configuration

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
```

### EKS Cluster

```hcl
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
```

### RDS PostgreSQL

```hcl
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
```

### ElastiCache Redis

```hcl
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
```

### MSK Kafka

```hcl
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

## Key Concepts

### Kubernetes Resources
- **Deployment**: Manages the desired state of application pods
- **Service**: Exposes the application within the cluster
- **HorizontalPodAutoscaler**: Automatically scales pods based on metrics
- **ConfigMap**: Stores non-sensitive configuration data
- **Secret**: Stores sensitive data like passwords and API keys

### Terraform Resources
- **EKS**: Managed Kubernetes service on AWS
- **RDS**: Managed relational database service
- **ElastiCache**: Managed Redis/Memcached service
- **MSK**: Managed Apache Kafka service

---

## Best Practices

1. **Version Control**: Store all IaC configurations in Git
2. **State Management**: Use remote state storage (S3, Azure Blob, GCS)
3. **Modularity**: Break configurations into reusable modules
4. **Environment Separation**: Use workspaces or separate state files per environment
5. **Secret Management**: Never store secrets in code; use secret managers
6. **Resource Tagging**: Tag all resources for cost tracking and management

---

## Navigation

[Previous: 10.1 CI/CD Pipeline](../10.1-cicd-pipeline/README.md) | [Back to Lesson 10](../README.md) | [Next: 10.3 Environment Strategy](../10.3-environment-strategy/README.md)
