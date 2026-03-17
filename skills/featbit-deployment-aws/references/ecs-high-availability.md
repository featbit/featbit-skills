# FeatBit on AWS ECS Fargate — High Availability Setup

> **Baseline reference**: The [featbit-terraform-aws](https://github.com/featbit/featbit-terraform-aws) project implements the architecture described in this document. Use it as your starting point for AWS deployments.
>
> **Using Kubernetes on AWS?** See the **featbit-deployment-kubernetes** skill instead.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Understanding the Difference: Fault Tolerance vs. High Availability](#understanding-the-difference-fault-tolerance-vs-high-availability)
3. [Step 2 — Increase the Task Count](#step-2--increase-the-task-count)
4. [Step 3 — Spread Tasks Across Availability Zones](#step-3--spread-tasks-across-availability-zones)
5. [For Complete High Availability](#for-complete-high-availability)
6. [Reference Implementation](#reference-implementation)

---

## Architecture Overview

```
Internet
    │
    ▼
┌─────────────────────────────────┐
│  Application Load Balancer (ALB)│
│  (public subnets, multi-AZ)     │
└──────┬──────┬──────┬────────────┘
       │      │      │      │
  host routing rules (per subdomain)
       │      │      │      │
  ┌────▼┐ ┌───▼─┐ ┌──▼──┐ ┌▼──┐
  │ ui  │ │ api │ │eval │ │ da│   ← ALB Target Groups
  └──┬──┘ └──┬──┘ └──┬──┘ └─┬─┘
     │       │       │      │
  ┌──▼──┐ ┌──▼──┐ ┌──▼──┐ ┌▼───┐
  │ECS  │ │ECS  │ │ECS  │ │ECS │   ← ECS Services
  │task │ │task │ │task │ │task│     (desired_count = 2+)
  │task │ │task │ │task │ │task│
  └─────┘ └─────┘ └─────┘ └────┘
       (private subnets, multi-AZ)
          │
     ┌────┴──────────────┐
     │ Aurora PostgreSQL  │   (Multi-AZ for full HA)
     │ ElastiCache/Redis  │   (with replication)
     └───────────────────┘
```

## Understanding the Difference: Fault Tolerance vs. High Availability

These two terms are often used interchangeably, but they mean different things in practice:

| Scenario | What it protects against | Term |
|----------|--------------------------|------|
| Multiple tasks in the **same AZ** | Individual task crashes — ECS replaces a failed task, other tasks keep serving traffic immediately | **Fault tolerance** |
| Tasks spread across **multiple AZs** | Full AZ outage — traffic is rerouted to tasks in healthy AZs | **High Availability (HA)** |

**In practice**: Simply increasing the task count — even without cross-AZ placement — already significantly improves service reliability. If one task crashes, the ALB instantly stops routing to it and the remaining tasks absorb the traffic. ECS then launches a replacement in the background.

This is a meaningful improvement over running a single task, but it is not HA. If the underlying host, subnet, or AZ experiences an infrastructure failure, all tasks in that location are affected.

---

Create one ALB with four target groups, one per service. The ALB uses **host-based routing** to direct traffic to the correct target group based on the subdomain:

| Subdomain | Target group | Service |
|-----------|-------------|---------|
| `app.{domain}` | ui-tg | ui |
| `api.{domain}` | api-tg | api-server |
| `eval.{domain}` | eval-tg | evaluation-server |
| `da.{domain}` | da-tg | data-analytics-server |

### Target group health checks

Each target group should have a health check configured. ECS will only route traffic to tasks that are currently healthy:

| Service | Health check path |
|---------|-----------------|
| ui | `/health` |
| api-server | `/health/liveness` |
| evaluation-server | `/health/liveness` |
| data-analytics-server | `/health/liveness` |

### Note on evaluation-server and WebSocket

The `evaluation-server` uses long-lived **WebSocket** connections for real-time flag updates to SDKs. ALB supports WebSocket natively — no special configuration is required. Once a WebSocket connection is established, it stays on the same task. New connections from new SDK clients are load-balanced across all healthy tasks.

---

## Step 2 — Increase the Task Count

With the ALB in place, increase `desired_count` for each ECS service from `1` to `2` (or higher). ECS will start the additional tasks and register them with the ALB target group automatically.

Do this for all four services: `ui`, `api-server`, `evaluation-server`, and `data-analytics-server`.

---

## Step 3 — Spread Tasks Across Availability Zones

ECS Fargate will spread tasks across the AZs covered by the configured subnets automatically. Make sure your ECS tasks run in **at least 2 private subnets in different AZs**. With `desired_count = 2`, ECS will place one task per AZ — so the service stays up even if a full AZ goes down.

---

## For Complete High Availability

Running multiple ECS tasks protects against individual **task-level failures**. For full infrastructure HA, the underlying stateful services also need to be resilient:

| Layer | Recommendation |
|-------|---------------|
| ECS tasks | `desired_count = 2+`, subnets across 2+ AZs |
| Database (PostgreSQL) | Aurora Multi-AZ or RDS Multi-AZ |
| Cache (Redis) | ElastiCache with Multi-AZ replication group |
| Load balancer | ALB is already multi-AZ by default |

---

## Reference Implementation

The official **FeatBit Terraform AWS** project implements this exact architecture (ALB + target groups + ECS services with health checks):

**https://github.com/featbit/featbit-terraform-aws**

- ALB and target groups: `modules/ecs-services/main.tf`
- ECS task definitions and services: `modules/ecs-services/ecs.tf`
