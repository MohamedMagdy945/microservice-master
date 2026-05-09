# CI/CD (Continuous Integration & Continuous Deployment)

## Overview

CI/CD is a modern software development practice that automates the process of:
- Building applications
- Testing code
- Deploying software

CI/CD helps teams deliver software faster, safer, and more reliably.

---

# What Does CI/CD Mean?

| Term | Meaning |
|---|---|
| CI | Continuous Integration |
| CD | Continuous Deployment / Continuous Delivery |

---

# 1. Continuous Integration (CI)

Continuous Integration is the practice of frequently merging code changes into a shared repository.

Every change triggers automated:
- Builds
- Tests
- Validation checks

---

## Goals of CI

- Detect bugs early
- Prevent integration problems
- Improve code quality
- Automate testing

---

## CI Workflow

```text
Developer Pushes Code
        ↓
Build Starts
        ↓
Automated Tests Run
        ↓
Validation Checks
        ↓
Code Merged Safely
```

---

## Common CI Tasks

- Restore dependencies
- Build application
- Run unit tests
- Run integration tests
- Run linting
- Generate reports

---

## Benefits of CI

- Faster feedback
- Better collaboration
- Reduced bugs
- Stable codebase
- Easier integration

---

# 2. Continuous Delivery (CD)

Continuous Delivery ensures that software is always ready for deployment.

The deployment process is automated, but production release may require manual approval.

---

## Workflow

```text
Code Passed CI
        ↓
Package Application
        ↓
Deploy to Staging
        ↓
Manual Approval
        ↓
Deploy to Production
```

---

## Benefits

- Reliable releases
- Faster deployments
- Reduced deployment risks
- Easier rollback

---

# 3. Continuous Deployment

Continuous Deployment goes one step further.

Every successful change is automatically deployed to production without manual intervention.

---

## Workflow

```text
Code Commit
        ↓
Build & Tests
        ↓
Deployment Pipeline
        ↓
Automatic Production Deployment
```

---

## Benefits

- Very fast delivery
- Rapid feedback from users
- Smaller release changes
- Continuous improvements

---

# CI/CD Pipeline

A CI/CD pipeline is a sequence of automated stages.

## Common Pipeline Stages

### 1. Source Stage
Code is pulled from Git repositories.

### 2. Build Stage
Application is compiled and packaged.

### 3. Test Stage
Automated tests are executed.

### 4. Deploy Stage
Application is deployed to environments.

### 5. Monitoring Stage
System health and logs are monitored.

---

# Common CI/CD Tools

| Category | Tools |
|---|---|
| Source Control | Git, GitHub, GitLab |
| CI/CD Platforms | GitHub Actions, GitLab CI, Jenkins |
| Containers | Docker |
| Orchestration | Kubernetes |
| Monitoring | Prometheus, Grafana |

---

# GitHub Actions Example

## CI Pipeline Example

```yaml
name: .NET CI

on:
  push:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: 9.0.x

      - name: Restore Packages
        run: dotnet restore

      - name: Build Project
        run: dotnet build --no-restore

      - name: Run Tests
        run: dotnet test --no-build
```

---

# Deployment Strategies

## 1. Rolling Deployment
Deploy updates gradually without downtime.

---

## 2. Blue-Green Deployment
Two environments are maintained:
- Blue (current)
- Green (new)

Traffic switches after validation.

---

## 3. Canary Deployment
Release changes to a small group of users first.

---

# CI/CD in Microservices

CI/CD is critical in microservices because:
- Services are deployed independently
- Frequent updates occur
- Automation reduces operational complexity

Each microservice may have:
- Separate pipelines
- Independent deployments
- Independent testing

---

# Best Practices

- Keep pipelines fast
- Automate testing
- Use Infrastructure as Code
- Monitor deployments
- Use rollback strategies
- Keep environments consistent

---

# Common Challenges

## Slow Pipelines
Large builds and tests delay feedback.

## Flaky Tests
Unstable tests reduce trust in automation.

## Environment Differences
Different environments may cause deployment issues.

## Security Risks
Secrets and credentials must be protected.

---

# Security in CI/CD

## Important Practices
- Use secret managers
- Scan dependencies
- Validate containers
- Use least privilege access
- Protect deployment environments

---

# Benefits of CI/CD

- Faster software delivery
- Better software quality
- Reduced manual work
- More reliable deployments
- Improved developer productivity

---

# Conclusion

CI/CD is a fundamental practice in modern software engineering.

It enables teams to:
- Deliver software faster
- Improve reliability
- Automate workflows
- Reduce deployment risks

CI/CD is widely used with:
- Docker
- Kubernetes
- GitHub Actions
- Jenkins
- GitLab CI/CD

It is essential for:
- Cloud-native applications
- DevOps culture
- Microservices architectures