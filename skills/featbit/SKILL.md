---
name: featbit
description: Routes FeatBit engineering questions to current official sources. Use for integrating any FeatBit SDK or OpenFeature provider; managing FeatBit resources with Terraform; deploying, upgrading, maintaining, or observing FeatBit with Docker Compose, Kubernetes and Helm, Microsoft Aspire, AWS or Azure Terraform, or OpenTelemetry; using Management, Evaluation, and Track Insights REST APIs; or finding official documentation for feature flag operations, experimentation, IAM, integrations, data import and export, or relay proxy. Do not use for general feature flag advice unrelated to FeatBit.
license: MIT
metadata:
  author: FeatBit
  version: 0.1.1
  category: platform
---

# FeatBit

Use this skill to locate the current authoritative source for FeatBit engineering work without loading copied SDK documentation, API schemas, deployment commands, or configuration catalogs into context.

Read only the reference relevant to the request. Treat its linked FeatBit documentation, GitHub repository, release tag, or live OpenAPI schema as the source of truth, and match guidance to the user's SDK or deployed FeatBit version.

**SDK:** Read [references/sdk.md](references/sdk.md) when application code needs a FeatBit SDK or OpenFeature provider, or when an unsupported platform needs the Evaluation and Track Insights APIs.

**REST API:** Read [references/rest-api.md](references/rest-api.md) when the request involves programmatic resource management, the live Management OpenAPI contract, direct feature flag evaluation, or Track Insights.

**Terraform Provider:** Use the official [FeatBit Terraform Provider](https://github.com/featbit/terraform-provider-featbit) when managing FeatBit resources with Terraform, and follow the repository's current documentation and releases.

**Deployment and maintenance:** Read [references/deployment.md](references/deployment.md) for installation, deployment, upgrades, maintenance, troubleshooting, or observability with Docker Compose, Kubernetes and Helm, Microsoft Aspire, AWS or Azure Terraform, or OpenTelemetry.

**Other FeatBit documentation:** For FeatBit topics outside SDKs, REST APIs, and deployment—such as feature flag configuration, targeting, experimentation, IAM, integrations, data import and export, or relay proxy—search [docs.featbit.co](https://docs.featbit.co/) using the user's terminology, open only the relevant pages, and use the [documentation sitemap](https://docs.featbit.co/sitemap.xml) only when search cannot locate them. Do not use this fallback for topics already covered by the three references.
