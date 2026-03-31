<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset=".github/logo-light.svg">
    <img alt="FeatBit Skills" src=".github/logo-light.svg" width="440">
  </picture>

  <p>Give your AI coding agent expert-level FeatBit knowledge — SDKs, deployments, and APIs — without loading irrelevant context.</p>
</div>

<div align="center">

[![License: MIT][license-shield]][license-url]
[![Version][version-shield]][version-url]
[![skills.sh][skills-shield]][skills-url]

</div>

<div align="center">
  <a href="#install">Install</a> &middot;
  <a href="#available-skills">Skills Catalog</a> &middot;
  <a href="https://docs.featbit.co">Docs</a> &middot;
  <a href="https://featbit.co">FeatBit</a>
</div>

---

## The Problem

General-purpose coding agents know how to write application code, but they do not know FeatBit-specific terminology, deployment tradeoffs, or which official resource is authoritative for a given task.

This collection gives agents focused, low-noise guidance for the exact FeatBit workflow in front of them: choose the right SDK, deploy the platform, call the management APIs, or route to the right documentation page — without loading unrelated context.

## Features

- **17 skills across 5 categories** — SDK integration, deployment, REST APIs, observability, and platform onboarding
- **7 language SDKs** — .NET, Node.js, Python, Java, Go, JavaScript, React / React Native with framework-specific quick starts
- **3 deployment targets** — Docker Compose (Standalone / Standard / Professional), Kubernetes with Helm, AWS ECS Fargate with Terraform
- **Smart routing** — the `featbit-sdks` router skill detects the user's language and loads only the relevant SDK skill
- **Selective loading** — install individual skills instead of the full collection to save context tokens

## Available Skills

| Category | Skill | Description |
|---|---|---|
| Platform | [featbit-getting-started][gs] | First-run guide for onboarding, initial feature flags, and SDK handoff |
| Platform | [featbit-documentation][doc] | Routes unanswered questions to the most relevant official FeatBit docs |
| Platform | [featbit-opentelemetry][otel] | Configures FeatBit metrics, traces, and logs with OpenTelemetry |
| API | [featbit-rest-api][rest] | Manages projects, environments, and feature flags via FeatBit REST APIs |
| API | [featbit-evaluation-insights-api][eval] | Calls evaluation and track-insights APIs for custom SDKs and unsupported platforms |
| Deployment | [featbit-deployment-docker][docker] | Deploys FeatBit with Docker Compose across Standalone, Standard, and Professional tiers |
| Deployment | [featbit-deployment-kubernetes][k8s] | Deploys FeatBit to Kubernetes with Helm across managed and self-managed clusters |
| Deployment | [featbit-deployment-aws][aws] | Deploys FeatBit on AWS with ECS Fargate, ALB, and Terraform patterns |
| SDK Router | [featbit-sdks][sdks] | Routes FeatBit SDK questions to the correct language-specific skill |
| Server SDK | [featbit-sdks-dotnet][dotnet] | Integrates the FeatBit .NET Server SDK with C# and ASP.NET Core applications |
| Server SDK | [featbit-sdks-node][node] | Integrates the FeatBit Node.js Server SDK in backend JavaScript or TypeScript services |
| Server SDK | [featbit-sdks-python][python] | Integrates the FeatBit Python Server SDK in backend Python applications |
| Server SDK | [featbit-sdks-java][java] | Integrates the FeatBit Java Server SDK in JVM backend services |
| Server SDK | [featbit-sdks-go][go] | Integrates the FeatBit Go Server SDK in Go services and APIs |
| Client SDK | [featbit-sdks-javascript][js] | Integrates the FeatBit JavaScript Client SDK in browser applications |
| Client SDK | [featbit-sdks-react][react] | Integrates the FeatBit React Client SDK with React and Next.js frontends |
| Client SDK | [featbit-sdks-react-native][rn] | Integrates the FeatBit React Native SDK in React Native and Expo apps |

## Install

> **Prerequisite**: [Node.js](https://nodejs.org/) v16+ is required to run `npx`.

```bash
npx skills add featbit/featbit-skills
```

The wizard detects your installed AI agents (Claude Code, Cursor, GitHub Copilot, Windsurf, etc.) and lets you select which skills to install. No configuration needed.

```bash
# Install specific skills only
npx skills add featbit/featbit-skills --skill featbit-sdks-dotnet --skill featbit-deployment-docker

# Install to a specific agent
npx skills add featbit/featbit-skills -a claude-code

# List all available skills without installing
npx skills add featbit/featbit-skills --list
```

> Skills are installed to your agent's skills directory (e.g., `.claude/skills/` for Claude Code, `.agents/skills/` for Cursor/Copilot). The CLI supports 40+ agents — see [skills.sh][skills-url] for the full list.

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
- Skills ecosystem: [skills.sh][skills-url]

## License

MIT

---

Crafted with [Readme Craft](https://github.com/anthropics/claude-code)

<!-- Badge references -->
[license-shield]: https://img.shields.io/badge/License-MIT-yellow.svg
[license-url]: LICENSE
[version-shield]: https://img.shields.io/badge/version-2.1.2-blue.svg
[version-url]: CHANGELOG.md
[skills-shield]: https://img.shields.io/badge/install-skills.sh-black.svg
[skills-url]: https://skills.sh/featbit/featbit-skills

<!-- Skill link references -->
[gs]: skills/featbit-getting-started/SKILL.md
[doc]: skills/featbit-documentation/SKILL.md
[otel]: skills/featbit-opentelemetry/SKILL.md
[rest]: skills/featbit-rest-api/SKILL.md
[eval]: skills/featbit-evaluation-insights-api/SKILL.md
[docker]: skills/featbit-deployment-docker/SKILL.md
[k8s]: skills/featbit-deployment-kubernetes/SKILL.md
[aws]: skills/featbit-deployment-aws/SKILL.md
[sdks]: skills/featbit-sdks/SKILL.md
[dotnet]: skills/featbit-sdks-dotnet/SKILL.md
[node]: skills/featbit-sdks-node/SKILL.md
[python]: skills/featbit-sdks-python/SKILL.md
[java]: skills/featbit-sdks-java/SKILL.md
[go]: skills/featbit-sdks-go/SKILL.md
[js]: skills/featbit-sdks-javascript/SKILL.md
[react]: skills/featbit-sdks-react/SKILL.md
[rn]: skills/featbit-sdks-react-native/SKILL.md
