# FeatBit Agent Skills

[![Version](https://img.shields.io/badge/version-0.1.2-blue.svg)](https://github.com/featbit/featbit-skills/releases/tag/0.1.2)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Two Agent Skills for AI coding agents working with [FeatBit](https://featbit.co): **FeatBit** routes engineering work to current official sources, while **FeatBit Experimentation** guides end-to-end experiments and release decisions for FeatBit v6.0.0 and later. Both skills are compatible with Codex, Claude Code, Cursor, GitHub Copilot, Windsurf, and other tools supported by `skills.sh`.

## What Is Included

| | |
|---|---|
| 2 Skills | Route FeatBit engineering questions to official sources and guide end-to-end experimentation and release decisions |
| Focused References | Source maps for FeatBit engineering plus stage-specific experimentation guidance |
| 0 Documentation Mirrors | Detailed instructions remain in current FeatBit repositories, docs, releases, and live API contracts |

## Skills

| Skill | Description |
|---|---|
| [FeatBit](skills/featbit/SKILL.md) | Routes requests to the relevant reference, the official Terraform Provider repository, or current FeatBit documentation |
| [FeatBit Experimentation](skills/featbit-experimentation/SKILL.md) | Guides product intent, hypotheses, rollout strategy, measurement, analysis, release decisions, and learning capture. Requires FeatBit v6.0.0 or later |

## Install

```bash
npx skills add featbit/featbit-skills
```

The installer detects supported agents and lets you install the same skill for Codex, Claude Code, GitHub Copilot, Cursor, or another supported agent.

Install either skill explicitly, or list all available skills:

```bash
npx skills add featbit/featbit-skills --skill featbit
npx skills add featbit/featbit-skills --skill featbit-experimentation
npx skills add featbit/featbit-skills --list
```

## License

MIT
