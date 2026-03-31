---
name: featbit-deployment-kubernetes
description: Deploys FeatBit to Kubernetes using Helm Charts. Use when user mentions "Kubernetes", "Helm", "K8s", "kubectl", works with values.yaml files, asks about "cloud deployment", "Azure Kubernetes", "AKS", "EKS", "GKE", "ingress", or needs production-grade container orchestration setup. Do not use for Docker Compose deployments or AWS-specific Terraform questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: deployment
---

# FeatBit Kubernetes Deployment with Helm

Deploy FeatBit to Kubernetes using official Helm Charts with support for two architecture tiers, multiple service exposure methods, external databases, and auto-scaling.

## Execution Procedure

```python
def assist_kubernetes_deployment(query):
    # Route by user intent
    if mentions("quick start", "get started", "install", "deploy"):
        # Walk through Quick Start Workflow below
        pass
    elif mentions("standard", "default", "basic"):
        refer("Architecture Tiers > Standard")
        refer("Configuration Examples > Standard Tier Examples")
    elif mentions("professional", "enterprise", "kafka", "clickhouse", "analytics"):
        refer("Architecture Tiers > Professional")
        refer("Configuration Examples > Professional Tier Examples")
    elif mentions("AKS", "Azure", "Azure Kubernetes"):
        refer("Configuration Examples > Azure AKS Production Examples")
    elif mentions("EKS", "GKE", "AWS", "Google"):
        refer("Configuration Examples > Standard/Professional Tier Examples")
        # Adapt values.yaml for cloud-specific ingress/services
    elif mentions("ingress", "TLS", "SSL", "certificate", "domain"):
        refer("Configuration Examples > Standard Tier Examples")  # includes ingress/TLS
    elif mentions("values", "configure", "customize"):
        refer("Key Resources > Values Reference")
    elif mentions("upgrade", "migrate", "migration"):
        refer("Upgrading & Migrations")
    elif mentions("troubleshoot", "error", "not working", "failed"):
        # Check pod status, logs, events
        suggest("kubectl get pods -n featbit")
        suggest("kubectl describe pod <pod-name> -n featbit")
        suggest("kubectl logs <pod-name> -n featbit")
    else:
        # Default: show tier comparison, ask user to pick
        pass
```

## Complete Guide

**Official Repository**: https://github.com/featbit/featbit-charts

**Primary Documentation**: https://raw.githubusercontent.com/featbit/featbit-charts/refs/heads/main/README.md

## Overview

**Prerequisites**:
- Kubernetes >= 1.23
- Helm >= 3.7.0
- kubectl configured

## Quick Start Workflow

### Checklist

- [ ] Add the FeatBit Helm repository
- [ ] Choose deployment tier (Standard or Professional)
- [ ] Create a values override file
- [ ] Install with Helm
- [ ] Verify all pods are running
- [ ] Access the FeatBit UI

### Step 1: Add the Helm Repo

```bash
helm repo add featbit https://featbit.github.io/featbit-charts
helm repo update
```

### Step 2: Choose Deployment Tier

| Tier | Set `architecture.tier` to | Includes |
|------|---------------------------|----------|
| **Standard** (default) | `"standard"` | API + Evaluation + UI + DB + optional Redis |
| **Professional** | `"pro"` | Standard + Kafka + ClickHouse |

Pick Standard unless you need real-time analytics at enterprise scale.

### Step 3: Install with Helm

**Minimal install (Standard tier, PostgreSQL, default settings):**

```bash
helm install featbit featbit/featbit \
  --create-namespace \
  --namespace featbit
```

**Standard tier with custom values:**

```bash
# Download an example values file as a starting point
# See: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/standard

helm install featbit featbit/featbit \
  --create-namespace \
  --namespace featbit \
  -f my-values.yaml
```

**Professional tier:**

```bash
helm install featbit featbit/featbit \
  --create-namespace \
  --namespace featbit \
  --set architecture.tier=pro \
  -f my-values.yaml
```

### Step 4: Verify Deployment

```bash
# Wait for all pods to become Ready
kubectl get pods -n featbit -w

# Check services
kubectl get svc -n featbit
```

All pods should reach `Running` status with all containers ready (e.g., `1/1`).

### Step 5: Access FeatBit UI

**Option A — Port-forward (development / quick test):**

```bash
kubectl port-forward svc/featbit-ui 8081:8081 -n featbit
# Open http://localhost:8081
```

**Option B — LoadBalancer or Ingress (production):**

Configure `ui.service.type` or `ingress` in your values file. See the Standard Tier Examples for Ingress, LoadBalancer, and NodePort configurations.

**Default credentials:** `test@featbit.com` / `123456`

### Common Customizations

```yaml
# my-values.yaml — selected overrides
architecture:
  tier: "standard"        # or "pro"
  database: "postgres"    # or "mongodb"

redis:
  enabled: true           # set false to disable caching

ui:
  service:
    type: ClusterIP       # or LoadBalancer, NodePort

# Use an external database instead of the bundled one
externalPostgresql:
  host: "my-rds.example.com"
  port: 5432
  username: "featbit"
  password: "secret"
  database: "featbit"
```

For the full values reference, see: https://github.com/featbit/featbit-charts/blob/main/charts/featbit/values.yaml

## Architecture Tiers

Two official deployment tiers configured via `architecture.tier` in values.yaml:

### Standard (Default)
Database (PostgreSQL or MongoDB) with optional Redis caching.
- **Database options**: PostgreSQL or MongoDB (via `architecture.database: "postgres"` or `"mongodb"`)
- **Redis**: Optional (disable with `redis.enabled: false`)
- **Use for**: All production and development environments, scales horizontally

### Professional (Enterprise)
Standard tier + Kafka + ClickHouse for real-time analytics and insights.
- **Use for**: Enterprise deployments requiring real-time data analysis

## Configuration Examples

Official example configurations organized by deployment scenario:

### Standard Tier Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/standard

Examples include:
- PostgreSQL + Redis configuration for local Kubernetes
- MongoDB + Redis configuration for local Kubernetes
- Service exposure methods (Ingress, LoadBalancer, NodePort)
- TLS/SSL certificate configuration

### Professional Tier Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/pro

Examples include:
- Full enterprise stack with Kafka and ClickHouse
- Real-time analytics configuration
- High-availability setup

### Azure AKS Production Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/aks

Examples include:
- Complete Azure AKS production deployment
- NGINX Ingress controller setup
- cert-manager for TLS certificates
- Azure managed services integration (Azure Database for PostgreSQL, Azure Managed Redis)

### All Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples

Browse all available configuration examples for different deployment scenarios and cloud providers.

## Upgrading & Migrations

**Important**: Starting from v0.9.0, database migrations are NOT automatic.

- **Migration Scripts**: https://github.com/featbit/featbit-charts/tree/main/migration
- **Migration Guide**: Review `migration/RELEASE-v{version}.md` before upgrading

## Key Resources

- **Values Reference**: https://github.com/featbit/featbit-charts/blob/main/charts/featbit/values.yaml
- **Chart README**: https://github.com/featbit/featbit-charts/blob/main/README.md
- **FeatBit Documentation**: https://docs.featbit.co
- **Main Repository**: https://github.com/featbit/featbit
