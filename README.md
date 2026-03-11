# FeatBit Agent Skills

[![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skills.sh](https://img.shields.io/badge/install-skills.sh-black.svg)](https://skills.sh/featbit/featbit-skills)

Skills, deployment guides, and SDK integration knowledge for AI coding agents working with [FeatBit](https://featbit.co) — an open-source feature flags and A/B testing platform.

## Quick Start

> **Prerequisite**: [Node.js](https://nodejs.org/) v16+ is required to run `npx`.

```bash
npx skills add featbit/featbit-skills
```

The wizard will detect your installed AI agents (Claude Code, Cursor, GitHub Copilot, Windsurf, etc.) and let you select which skills to install. No configuration needed.

```bash
# Install specific skills only
npx skills add featbit/featbit-skills --skill featbit-sdks-dotnet --skill featbit-deployment-docker

# Install to a specific agent
npx skills add featbit/featbit-skills -a claude-code

# List all available skills without installing
npx skills add featbit/featbit-skills --list
```

> Skills are installed to your agent's skills directory (e.g., `.claude/skills/` for Claude Code, `.agents/skills/` for Cursor/Copilot). The CLI supports 40+ agents — see [skills.sh](https://skills.sh) for the full list.

## What's Inside

| | |
|---|---|
| 16 Skills | SDK integration, deployment, API, and platform knowledge |
| 7 Languages | .NET, Node.js, Python, Java, Go, JavaScript, React / React Native |
| 3 Deployment targets | Docker Compose, Kubernetes/Helm, AWS ECS Fargate |

Use skills selectively. Loading all skills when only one is relevant wastes context tokens and dilutes agent attention.

## Skill Catalog

### Platform

| Skill | Description |
|---|---|
| `featbit-getting-started` | First-run guide: create a feature flag, evaluate it, integrate an SDK |
| `featbit-documentation` | Routes questions to official docs.featbit.co pages |
| `featbit-rest-api` | Manage projects, environments, and feature flags via HTTP API |
| `featbit-opentelemetry` | Configure metrics, traces, and logs with OpenTelemetry backends |

### Deployment

| Skill | Description |
|---|---|
| `featbit-deployment-docker` | Docker Compose: Standalone, Standard, and Professional tiers |
| `featbit-deployment-kubernetes` | Helm Charts for AKS, EKS, GKE with ingress and auto-scaling |
| `featbit-deployment-aws` | ECS Fargate with ALB and Terraform; multi-AZ high availability |

### Server-Side SDKs

| Skill | Triggers |
|---|---|
| `featbit-sdks-dotnet` | `.cs`, `.csproj`, ASP.NET Core, dependency injection |
| `featbit-sdks-node` | `server.js`, Express, NestJS, server-side TypeScript |
| `featbit-sdks-python` | `.py`, Flask, Django, FastAPI |
| `featbit-sdks-java` | `.java`, `pom.xml`, `build.gradle`, Spring Boot |
| `featbit-sdks-go` | `.go`, `go.mod`, Gin, Echo |

### Client-Side SDKs

| Skill | Triggers |
|---|---|
| `featbit-sdks-javascript` | Browser `.js`/`.html`, vanilla JS, SPA |
| `featbit-sdks-react` | `.jsx`, `.tsx`, React hooks, Next.js |
| `featbit-sdks-react-native` | React Native, Expo, iOS/Android |
| `featbit-sdks` | Language router — use when no specific language is mentioned |

## Repository Structure

```
skills/
└── featbit-{topic}/
    ├── SKILL.md          # Agent instructions (required, <500 lines)
    └── references/       # Supporting reference docs (one level deep)

AGENTS.md                 # Authoring rules and contribution guidelines
CHANGELOG.md
package.json              # Skills manifest
```

## Contributing

New skills and improvements are welcome. Before submitting, read [AGENTS.md](AGENTS.md) for authoring rules, naming conventions, and the pre-commit checklist.

```bash
# Create a new skill scaffold
npx skills init featbit-{topic}
```

Commit format: `feat(topic): description` for new skills, `fix(topic): description` for corrections.

## Resources

- FeatBit platform: [featbit.co](https://featbit.co)
- Documentation: [docs.featbit.co](https://docs.featbit.co)
- FeatBit GitHub: [github.com/featbit/featbit](https://github.com/featbit/featbit)
- Skills ecosystem: [skills.sh](https://skills.sh)

## License

MIT

