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

```
1. CHECK prerequisites
   - Kubernetes >= 1.23 running and reachable
   - Helm >= 3.7.0 installed
   - kubectl configured for target cluster
   - IF upgrading from < v0.9.0 → run migration scripts FIRST

2. DETERMINE architecture tier
   - IF user needs analytics/insights → Professional (Kafka + ClickHouse)
   - ELSE → Standard (PostgreSQL/MongoDB + optional Redis)

3. CONFIGURE values.yaml
   - SET architecture.tier ("standard" | "pro")
   - SET architecture.database ("postgres" | "mongodb")
   - CONFIGURE service exposure (Ingress / LoadBalancer / NodePort)
   - IF external DB → disable bundled DB, set connection strings
   - IF TLS needed → configure cert-manager or manual certs
   - REFERENCE official examples for the chosen scenario

4. INSTALL or UPGRADE via Helm
   - helm repo add featbit https://featbit.github.io/featbit-charts
   - helm repo update
   - IF fresh install → helm install featbit featbit/featbit -f values.yaml -n <ns> --create-namespace
   - IF upgrade → review migration/RELEASE-v{version}.md, then helm upgrade featbit featbit/featbit -f values.yaml -n <ns>

5. VERIFY deployment
   - kubectl get pods -n <ns>  → all pods Running/Ready
   - kubectl get svc -n <ns>   → endpoints reachable
   - Access UI dashboard and confirm login
```

## Complete Guide

**Official Repository**: https://github.com/featbit/featbit-charts

**Primary Documentation**: https://raw.githubusercontent.com/featbit/featbit-charts/refs/heads/main/README.md

## Overview

**Prerequisites**:
- Kubernetes >= 1.23
- Helm >= 3.7.0
- kubectl configured

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

## Configure Services

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

## Upgrade and Migrate

**Important**: Starting from v0.9.0, database migrations are NOT automatic.

- **Migration Scripts**: https://github.com/featbit/featbit-charts/tree/main/migration
- **Migration Guide**: Review `migration/RELEASE-v{version}.md` before upgrading

## Consult These Resources

- **Values Reference**: https://github.com/featbit/featbit-charts/blob/main/charts/featbit/values.yaml
- **Chart README**: https://github.com/featbit/featbit-charts/blob/main/README.md
- **FeatBit Documentation**: https://docs.featbit.co
- **Main Repository**: https://github.com/featbit/featbit
