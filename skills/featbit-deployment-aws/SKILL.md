---
name: featbit-deployment-aws
description: Guidance for deploying FeatBit on AWS, including ECS Fargate, EKS (Kubernetes), and Terraform. Use when user asks about deploying or running FeatBit on AWS. Do not use for Docker Compose deployments or Kubernetes deployments on non-AWS clouds.
license: MIT
metadata:
  version: 1.0.0
  category: deployment
---

# FeatBit on AWS

## Recommended Deployment Options

**Option 1 — ECS Fargate + Terraform (recommended baseline)**
The official [featbit-terraform-aws](https://github.com/featbit/featbit-terraform-aws) project provides a ready-to-use Terraform setup for deploying FeatBit on AWS ECS Fargate with ALB, Aurora PostgreSQL, and ElastiCache. This is the recommended starting point for most AWS deployments.

**Option 2 — AWS Kubernetes (EKS)**
For teams already running Kubernetes on AWS, use the Helm chart approach instead.
See the **featbit-deployment-kubernetes** skill for full guidance.

---

## Q: Can I run multiple ECS Fargate tasks for High Availability?

**A:** Yes. All four FeatBit services (`ui`, `api-server`, `evaluation-server`, `data-analytics-server`) are stateless and support multiple concurrent instances. Place an ALB in front of each service first, then increase the task count.

Note the distinction: running multiple tasks in the **same AZ** improves **fault tolerance** (a crashed task is replaced while others keep serving traffic), but it is not true **High Availability**. For HA, tasks must span multiple subnets in different AZs so the service survives an entire AZ (Availability Zone) going down.

Not all services are equally critical — `evaluation-server` and `api-server` have direct user impact if they go down and should be scaled first. `ui` only affects the portal. `data-analytics-server` only affects analytics; flags keep working without it.

[references/ecs-high-availability.md](references/ecs-high-availability.md)

---

## Related Skills

- **featbit-deployment-docker**: Docker Compose deployment guide
- **featbit-deployment-kubernetes**: Kubernetes/Helm deployments (also covers AWS EKS)
