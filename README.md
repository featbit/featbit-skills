<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset=".github/logo-light.svg">
    <img alt="FeatBit Skills" src=".github/logo-light.svg" width="480">
  </picture>
</div>

<div align="center">

[![Version][version-shield]][version-url]
[![License: MIT][license-shield]][license-url]
[![skills.sh][skills-shield]][skills-url]

</div>

> Give your AI coding agent expert-level FeatBit knowledge — SDKs, deployments, APIs, and platform workflows — without polluting its context with irrelevant docs.

<div align="center">
  <a href="#quick-start">Quick Start</a> &middot;
  <a href="#available-skills">All Skills</a> &middot;
  <a href="#install">Install</a> &middot;
  <a href="https://docs.featbit.co">FeatBit Docs</a>
</div>

---

## Why

General-purpose coding agents know how to write application code, but they do not know FeatBit-specific terminology, deployment tradeoffs, or which official resource is authoritative for a given task. Without specialized skills, agents hallucinate SDK APIs, mix up deployment tiers, and send users to outdated documentation pages.

This collection gives agents focused, low-noise guidance for the exact FeatBit workflow in front of them — choose the right SDK, deploy the platform, call the management APIs, or route to the right documentation page — without loading unrelated context.

## Features

- **7 language-specific SDK skills** — .NET, Node.js, Python, Java, Go, JavaScript, React / React Native, each with setup workflows, code patterns, and troubleshooting
- **3 deployment targets** — Docker Compose (Standalone / Standard / Professional tiers), Kubernetes with Helm, and AWS ECS Fargate with Terraform
- **REST API coverage** — Authentication, projects, environments, and feature flag CRUD with complete request/response examples
- **Custom SDK building** — Flag Evaluation API and Track Insights API for platforms without an official SDK (Kotlin, Swift, Android, iOS, Unity)
- **Smart routing** — SDK router skill automatically directs questions to the correct language-specific skill; documentation router finds the most relevant official docs page
- **Progressive disclosure** — Only the triggered skill loads into context. Reference files load just-in-time, keeping token usage minimal

## Quick Start

Install the collection, then use natural language to trigger any skill:

```bash
npx skills add featbit/featbit-skills
```

| Say this to your agent | Skill activated |
|---|---|
| "Set up the FeatBit .NET SDK in my ASP.NET Core app" | featbit-sdks-dotnet |
| "Deploy FeatBit with Docker Compose for production" | featbit-deployment-docker |
| "Create a feature flag via the REST API" | featbit-rest-api |
| "Which FeatBit SDK should I use?" | featbit-sdks (router) |
| "How do I build a custom SDK for iOS?" | featbit-evaluation-insights-api |

## Install

> **Prerequisite**: [Node.js](https://nodejs.org/) v16+ is required to run `npx`.

```bash
npx skills add featbit/featbit-skills
```

The wizard detects your installed AI agents (Claude Code, Cursor, GitHub Copilot, Windsurf, etc.) and lets you select which skills to install.

```bash
# Install specific skills only
npx skills add featbit/featbit-skills --skill featbit-sdks-dotnet --skill featbit-deployment-docker

# Install to a specific agent
npx skills add featbit/featbit-skills -a claude-code

# List all available skills without installing
npx skills add featbit/featbit-skills --list
```

> Skills are installed to your agent's skills directory (e.g., `.claude/skills/` for Claude Code, `.agents/skills/` for Cursor/Copilot). The CLI supports 40+ agents — see [skills.sh](https://skills.sh) for the full list.

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

---

Crafted with [Readme Craft](https://github.com/motiful/readme-craft) · Forged with [Skill Forge](https://github.com/motiful/skill-forge)

[version-shield]: https://img.shields.io/badge/version-2.1.2-blue.svg
[version-url]: CHANGELOG.md
[license-shield]: https://img.shields.io/badge/License-MIT-yellow.svg
[license-url]: LICENSE
[skills-shield]: https://img.shields.io/badge/install-skills.sh-black.svg
[skills-url]: https://skills.sh/featbit/featbit-skills
