# FeatBit Agent Skill

[![Version](https://img.shields.io/badge/version-0.1.1-blue.svg)](https://github.com/featbit/featbit-skills/releases/tag/0.1.1)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skills.sh](https://img.shields.io/badge/install-skills.sh-black.svg)](https://skills.sh/featbit/featbit-skills)

One source-first Agent Skill for AI coding agents working with [FeatBit](https://featbit.co). Compatible with Codex, Claude Code, Cursor, GitHub Copilot, Windsurf, and other tools supported by `skills.sh`.

## What Is Included

| | |
|---|---|
| 1 Skill | Routes FeatBit SDK, REST API, Terraform resource management, deployment, maintenance, observability, and other official documentation requests |
| 3 References | Small source maps for SDKs, HTTP APIs, and platform operations |
| 0 Documentation Mirrors | Detailed instructions remain in current FeatBit repositories, docs, releases, and live API contracts |

## Skill

| Skill | Description |
|---|---|
| [featbit](skills/featbit/SKILL.md) | Routes requests to the relevant reference, the official Terraform Provider repository, or current FeatBit documentation |

The skill uses only the source route needed by the request:

- [SDK sources](skills/featbit/references/sdk.md) lists every SDK and OpenFeature provider in the official SDK catalog, with its GitHub repository and intended use.
- [REST API sources](skills/featbit/references/rest-api.md) links the live Management OpenAPI contract and the official Evaluation and Track Insights guides.
- [Terraform Provider](https://github.com/featbit/terraform-provider-featbit) is the official source for managing FeatBit resources with Terraform.
- [Deployment sources](skills/featbit/references/deployment.md) links the official Compose, Helm, Aspire, AWS Terraform, Azure Terraform, and OpenTelemetry sources.
- [Other FeatBit documentation](https://docs.featbit.co/) is searched directly for topics outside those references, with the documentation sitemap used only as a discovery fallback.

## Install

```bash
npx skills add featbit/featbit-skills
```

The installer detects supported agents and lets you install the same skill for Codex, Claude Code, GitHub Copilot, Cursor, or another supported agent.

Install or list the single skill explicitly:

```bash
npx skills add featbit/featbit-skills --skill featbit
npx skills add featbit/featbit-skills --list
```

## Design Rule

Do not copy SDK READMEs, deployment manifests, environment-variable catalogs, or OpenAPI schemas into this repository. Keep only enough context to select the correct source and understand when it applies.

Add a topical reference before adding another top-level skill. Split the skill only when real task evidence shows that one trigger, three references, and the scoped documentation fallback can no longer route requests reliably.

## Contributing

Keep skill content concise, source-first, and written in English (US). Verify external links and run `npx skills add ./skills --list` before publishing changes.

## License

MIT
