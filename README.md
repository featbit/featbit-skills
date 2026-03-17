# FeatBit Agent Skills

[![Version](https://img.shields.io/badge/version-2.1.2-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skills.sh](https://img.shields.io/badge/install-skills.sh-black.svg)](https://skills.sh/featbit/featbit-skills)

Agent Skills compatible with Claude Code, Cursor, GitHub Copilot, Windsurf, and other tools supported by `skills.sh`.

Skills, deployment guides, and SDK integration knowledge for AI coding agents working with [FeatBit](https://featbit.co) — an open-source feature flags and A/B testing platform.

## The Problem

General-purpose coding agents know how to write application code, but they usually do not know FeatBit-specific terminology, deployment tradeoffs, or which official resource is authoritative for a given task.

This collection gives agents focused, low-noise guidance for the exact FeatBit workflow in front of them: choose the right SDK, deploy the platform, call the management APIs, or route to the right documentation page without loading unrelated context.

## What Is Included

| | |
|---|---|
| 17 Skills | SDK integration, deployment, API, and platform knowledge |
| 7 Languages | .NET, Node.js, Python, Java, Go, JavaScript, React / React Native |
| 3 Deployment targets | Docker Compose, Kubernetes/Helm, AWS ECS Fargate |

Use skills selectively. Loading all skills when only one is relevant wastes context tokens and dilutes agent attention.

## Available Skills

| Category | Skill | Description |
|---|---|---|
| Platform | [featbit-getting-started](skills/featbit-getting-started/SKILL.md) | First-run guide for onboarding, initial feature flags, and SDK handoff |
| Platform | [featbit-documentation](skills/featbit-documentation/SKILL.md) | Routes unanswered questions to the most relevant official FeatBit docs |
| Platform | [featbit-opentelemetry](skills/featbit-opentelemetry/SKILL.md) | Configures FeatBit metrics, traces, and logs with OpenTelemetry |
| API | [featbit-rest-api](skills/featbit-rest-api/SKILL.md) | Manages projects, environments, and feature flags via FeatBit REST APIs |
| API | [featbit-evaluation-insights-api](skills/featbit-evaluation-insights-api/SKILL.md) | Calls evaluation and track-insights APIs for custom SDKs and unsupported platforms |
| Deployment | [featbit-deployment-docker](skills/featbit-deployment-docker/SKILL.md) | Deploys FeatBit with Docker Compose across Standalone, Standard, and Professional tiers |
| Deployment | [featbit-deployment-kubernetes](skills/featbit-deployment-kubernetes/SKILL.md) | Deploys FeatBit to Kubernetes with Helm across managed and self-managed clusters |
| Deployment | [featbit-deployment-aws](skills/featbit-deployment-aws/SKILL.md) | Deploys FeatBit on AWS with ECS Fargate, ALB, and Terraform patterns |
| SDK Router | [featbit-sdks](skills/featbit-sdks/SKILL.md) | Routes FeatBit SDK questions to the correct language-specific skill |
| Server SDK | [featbit-sdks-dotnet](skills/featbit-sdks-dotnet/SKILL.md) | Integrates the FeatBit .NET Server SDK with C# and ASP.NET Core applications |
| Server SDK | [featbit-sdks-node](skills/featbit-sdks-node/SKILL.md) | Integrates the FeatBit Node.js Server SDK in backend JavaScript or TypeScript services |
| Server SDK | [featbit-sdks-python](skills/featbit-sdks-python/SKILL.md) | Integrates the FeatBit Python Server SDK in backend Python applications |
| Server SDK | [featbit-sdks-java](skills/featbit-sdks-java/SKILL.md) | Integrates the FeatBit Java Server SDK in JVM backend services |
| Server SDK | [featbit-sdks-go](skills/featbit-sdks-go/SKILL.md) | Integrates the FeatBit Go Server SDK in Go services and APIs |
| Client SDK | [featbit-sdks-javascript](skills/featbit-sdks-javascript/SKILL.md) | Integrates the FeatBit JavaScript Client SDK in browser applications |
| Client SDK | [featbit-sdks-react](skills/featbit-sdks-react/SKILL.md) | Integrates the FeatBit React Client SDK with React and Next.js frontends |
| Client SDK | [featbit-sdks-react-native](skills/featbit-sdks-react-native/SKILL.md) | Integrates the FeatBit React Native SDK in React Native and Expo apps |

## Install

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

Built for the Agent Skills ecosystem with focused, progressive-disclosure FeatBit guidance.

## License

MIT

