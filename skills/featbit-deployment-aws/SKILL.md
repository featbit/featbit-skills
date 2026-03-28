---
name: featbit-deployment-aws
description: Guidance for deploying FeatBit on AWS, including ECS Fargate, EKS (Kubernetes), and Terraform. Use when user asks about deploying or running FeatBit on AWS. Do not use for Docker Compose deployments or Kubernetes deployments on non-AWS clouds.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: deployment
---

# FeatBit on AWS

Deploy FeatBit on AWS using ECS Fargate with Terraform (recommended) or EKS with Helm.

## Execution Procedure

```
1. Determine deployment method
   ├─ ECS Fargate (default for most teams)
   │   ├─ Use the official featbit-terraform-aws Terraform module
   │   ├─ Read references/ecs-high-availability.md for multi-AZ guidance
   │   └─ Scale stateless services behind ALB as needed
   └─ EKS (team already runs Kubernetes on AWS)
       └─ Redirect to featbit-deployment-kubernetes skill
```

## ECS Fargate + Terraform (Recommended)

Use the official [featbit-terraform-aws](https://github.com/featbit/featbit-terraform-aws) Terraform module to deploy FeatBit on ECS Fargate with ALB, Aurora PostgreSQL, and ElastiCache. Start here for most AWS deployments.

For EKS deployments, use the **featbit-deployment-kubernetes** skill instead.

## High Availability on ECS Fargate

All four FeatBit services (`ui`, `api-server`, `evaluation-server`, `data-analytics-server`) are stateless. Scale them by placing an ALB in front of each service and increasing the task count.

Running multiple tasks in the **same AZ** improves **fault tolerance** (a crashed task is replaced while others keep serving). For true **High Availability**, spread tasks across multiple subnets in different AZs so the service survives an entire AZ outage.

Prioritize scaling `evaluation-server` and `api-server` first — they have direct user impact. `ui` only affects the management portal. `data-analytics-server` only affects analytics; feature flags keep working without it.

See [references/ecs-high-availability.md](references/ecs-high-availability.md) for detailed guidance.

## Related Skills

- **featbit-deployment-docker**: Docker Compose deployment
- **featbit-deployment-kubernetes**: Kubernetes/Helm deployments (also covers AWS EKS)
