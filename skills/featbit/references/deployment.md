# FeatBit deployment and maintenance sources

Use this reference when installing, deploying, upgrading, maintaining, troubleshooting, or observing the FeatBit platform. Read the selected repository's current README, release notes, and version-specific migration files instead of relying on copied commands or configuration.

| Target | Authoritative source | Use |
|---|---|---|
| Deployment selection | https://docs.featbit.co/installation/deployment-options | Compare supported deployment topologies before choosing an implementation. |
| Docker Compose | [Installation guide](https://docs.featbit.co/installation/docker-compose) · [Source repository](https://github.com/featbit/featbit) | Use the guide for deployment choices and the version-matched repository for Compose files and platform source. |
| Kubernetes and Helm | https://github.com/featbit/featbit-charts | Use the official chart, values, examples, releases, and migration notes. |
| Microsoft Aspire | https://github.com/featbit/featbit-aspire | Run FeatBit locally or deploy it to Azure Container Apps through Aspire. |
| AWS Terraform | https://github.com/featbit/featbit-terraform-aws | Deploy and maintain FeatBit on AWS using the official Terraform project. |
| Azure Terraform | https://github.com/featbit/azure-container-apps | Deploy and maintain FeatBit on Azure Container Apps using Terraform. |
| OpenTelemetry | https://docs.featbit.co/integrations/observability/opentelemetry | Configure FeatBit metrics, traces, and logs from the current observability guide. |

Match every instruction to the user's deployed FeatBit version. For upgrades or destructive maintenance, verify migrations, backups, and rollback requirements in the selected upstream release before acting.
