# 10.1 CI/CD Pipeline

## Overview

This sub-lesson covers the design and implementation of CI/CD (Continuous Integration/Continuous Deployment) pipelines for insurance systems. CI/CD pipelines automate the process of building, testing, and deploying applications, ensuring consistent and reliable software delivery.

---

## Pipeline Architecture

```
+-----------------------------------------------------------------------------+
|                          CI/CD PIPELINE                                      |
+-----------------------------------------------------------------------------+
|                                                                              |
|  +----------+   +----------+   +----------+   +----------+   +----------+  |
|  |   Code   | > |  Build   | > |   Test   | > | Security | > | Artifact |  |
|  |  Commit  |   |          |   |          |   |   Scan   |   |  Publish |  |
|  +----------+   +----------+   +----------+   +----------+   +----------+  |
|       |              |              |              |              |         |
|       v              v              v              v              v         |
|    Git Push      Compile        Unit Tests    SAST/DAST      Push to       |
|    Lint          Package        Integration   Dependency     Registry      |
|    Format        Docker Build   Contract      Secrets Scan                  |
|                                                                              |
|  ========================================================================== |
|                                DEPLOYMENT                                    |
|  ========================================================================== |
|                                                                              |
|  +----------+   +----------+   +----------+   +----------+   +----------+  |
|  |  Deploy  | > |  Smoke   | > |   QA     | > | Staging  | > |   Prod   |  |
|  |   DEV    |   |  Tests   |   |          |   |          |   |  Deploy  |  |
|  +----------+   +----------+   +----------+   +----------+   +----------+  |
|       |              |              |              |              |         |
|       v              v              v              v              v         |
|    Auto          Health         Manual         Performance   Blue/Green   |
|    Deploy        Checks         Testing        Testing       Canary        |
|                  API Tests                     UAT           Rollback      |
|                                                                              |
+-----------------------------------------------------------------------------+
```

---

## GitHub Actions Pipeline

### Complete CI/CD Workflow

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

## Pipeline Stages Explained

### 1. Build Stage
- **Code Checkout**: Retrieves the latest code from the repository
- **Compilation**: Compiles the source code
- **Packaging**: Creates deployable artifacts (JARs, WARs, Docker images)

### 2. Test Stage
- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test component interactions
- **Contract Tests**: Validate API contracts between services

### 3. Security Scan Stage
- **SAST (Static Application Security Testing)**: Analyzes source code for vulnerabilities
- **DAST (Dynamic Application Security Testing)**: Tests running application for security issues
- **Dependency Scanning**: Checks for vulnerable dependencies
- **Container Scanning**: Scans Docker images for vulnerabilities

### 4. Artifact Publishing
- **Container Registry**: Push Docker images to registry (GHCR, ECR, Docker Hub)
- **Artifact Repository**: Store build artifacts (Nexus, Artifactory)

### 5. Deployment Stages
- **Development**: Auto-deployed on commit
- **Staging**: Deployed after passing tests
- **Production**: Deployed with approval and deployment strategy

---

## Diagram

See the accompanying diagram file: [10.1-cicd-pipeline.drawio](./10.1-cicd-pipeline.drawio)

---

## Best Practices

1. **Fail Fast**: Run quick tests first to provide rapid feedback
2. **Parallel Execution**: Run independent stages in parallel
3. **Immutable Artifacts**: Build once, deploy everywhere
4. **Security Scanning**: Integrate security checks into the pipeline
5. **Environment Parity**: Keep environments as similar as possible
6. **Automated Rollback**: Have automated rollback mechanisms in place

---

## Navigation

[Back to Lesson 10](../README.md) | [Next: 10.2 Infrastructure as Code](../02-infrastructure-as-code/README.md)
