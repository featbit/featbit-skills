---
name: featbit-documentation
description: FeatBit documentation router that provides likely relevant docs.featbit.co URLs when other FeatBit skills cannot fully answer. Use when user asks about FeatBit features, concepts, deployment, SDKs, API, integrations, or architecture and the response should point to official documentation for deeper detail.
license: MIT
metadata:
  author: FeatBit
  version: 2.0.0
  category: documentation-router
appliesTo:
  - "**"
---

# FeatBit Documentation Router

This skill is a **fallback** for FeatBit questions. Use it **only after** other FeatBit skills do not provide a complete answer. Its job is to return a short list of **likely relevant documentation URLs** so the agent can open those pages and extract the final details.

## When to Use This Skill

Activate this skill when:
- A FeatBit question is not fully answered by more specific FeatBit skills.
- The request requires authoritative details from FeatBit docs.
- The best next step is to point the agent to official documentation URLs.

Do **not** use this skill when another FeatBit skill already provides a full answer.

## Output Rules (Critical)

1. **Return URLs only** (plus a brief 1-line rationale per URL).
2. **Prefer docs.featbit.co URLs** from the documentation index.
3. Provide **3–7 URLs max**, ranked by relevance.
4. If a GitHub repo is more authoritative than docs, include it **in addition** to docs URLs.

## Quick Navigation Guide

FeatBit documentation is organized into 12 main sections. Use the topic keywords below to identify relevant areas, then consult the complete index for specific URLs.

**📚 Complete Documentation Index**: [references/complete-documentation-index.md](references/complete-documentation-index.md)  
Contains all 75 documentation URLs with detailed summaries. Reference this when you need specific page URLs.

### Documentation Topics

1. **Getting Started** (https://docs.featbit.co/getting-started)
   - SDK integration, flag creation tutorials, interactive demos
   - How-to guides: testing in production, remote config, beta testing, entitlement, A/B testing

2. **Feature Flags** (https://docs.featbit.co/feature-flags)
   - Flag creation, variations, lifecycle management
   - User targeting: rules, percentage rollouts, individual targeting
   - User segments: standard, shareable, global
   - Organizing: projects, environments, flag lists

3. **Experimentation** (https://docs.featbit.co/experimentation)
   - A/B testing, experiment creation, metric types
   - Statistical analysis, result interpretation

4. **Relay Proxy** (https://docs.featbit.co/relay-proxy)
   - FeatBit Agent deployment (Docker/systemd/native)
   - Auto mode (WebSocket sync) vs Manual mode (air-gapped)
   - GitHub: https://github.com/featbit/featbit-agent

5. **API Documentation** (https://docs.featbit.co/api-docs)
   - REST API authentication, endpoints, data hierarchy
   - Flag Evaluation API (client-side vs server-side)

6. **Data Import/Export** (https://docs.featbit.co/data-import-and-export)
   - End users, feature flags, segments
   - Analytics data export (ClickHouse/MongoDB)

7. **IAM** (https://docs.featbit.co/iam)
   - Teams, groups, policies
   - Role-based access control (RBAC)

8. **Installation** (https://docs.featbit.co/installation)
   - Deployment options: Standalone, Standard, Professional
   - Docker Compose, Terraform AWS, custom infrastructure

9. **Integrations** (https://docs.featbit.co/integrations)
   - Authentication: API tokens, SSO (Okta, KeyCloak, Auth0)
   - Observability: OpenTelemetry, New Relic, Grafana, DataDog
   - Chat apps: Slack
   - Analytics: GrowthBook

10. **Licenses** (https://docs.featbit.co/licenses)
    - Open Core model, enterprise features
    - Pricing: https://featbit.co/pricing

11. **Tech Stack** (https://docs.featbit.co/tech-stack)
    - Architecture, services (UI/API/Evaluation/Analytics)
    - Infrastructure components, data flows, performance benchmarks

12. **GitHub Repositories**
    - Main: https://github.com/featbit/featbit
    - Relay Proxy: https://github.com/featbit/featbit-agent
    - Terraform AWS: https://github.com/featbit/featbit-terraform-aws

## Usage Example

**User query**: "How do I gradually roll out a feature to 20% of users?"

**Your response**:
```
Here are the relevant documentation pages:

1. https://docs.featbit.co/feature-flags/targeting-users-with-flags/percentage-rollouts - Primary guide for percentage-based rollouts
2. https://docs.featbit.co/feature-flags/targeting-users-with-flags/targeting-rules - Understanding targeting rules structure
3. https://docs.featbit.co/feature-flags/create-flag-variations - How to set up flag variations

These pages explain how to enable percentage rollouts within targeting rules to serve features to a specified percentage of users with consistent assignment.
```
