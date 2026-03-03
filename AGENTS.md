# AI Coding Agents Guidelines

This document provides guidelines and standards for AI coding agents working on the FeatBit Skills project. All agents should follow these conventions to maintain consistency across the codebase.

---

## 📋 Table of Contents

- [Release Management](#release-management)
- [Versioning](#versioning)
- [Commit Messages](#commit-messages)
- [Documentation](#documentation)

---

## 🚀 Release Management

### Release Title Format

**Standard Format:**
```
v{MAJOR}.{MINOR}.{PATCH} - {Brief Feature Description}
```

**Examples:**
- `v1.2.0 - OpenTelemetry Observability Support`
- `v1.3.0 - Multi-Environment Configuration`
- `v2.0.0 - Breaking Changes and New Architecture`

**Rules:**
- Always start with `v` followed by semantic version
- Use a dash `-` to separate version from description
- Description should be 3-6 words
- Use title case for description
- Focus on the main feature/change

---

### Release Notes Template

**MUST follow this exact structure (keep it concise):**

```markdown
## What's New

- ✨ {New feature or skill added with brief description}
- 📚 {Additional feature or documentation}
- 🔧 {Configuration or setup improvements}

**Documentation**: {Link if applicable}

## Bug Fixes

- 🐛 {Bug fix description}
- 🐛 {Bug fix description}

*(Omit this section if no bug fixes)*

## Breaking Changes

- ❌ {Breaking change description and migration guide}
- ⚠️ {Deprecation notice}

*(Omit this section if no breaking changes)*

---

**Full Changelog**: https://github.com/featbit/featbit-skills/compare/v{PREVIOUS_VERSION}...v{CURRENT_VERSION}
```

**Real Example (v1.2.0):**

```markdown
## What's New

- ✨ Added new `featbit-opentelemetry` skill for comprehensive observability support
- 📊 Configuration guidance for Api, Evaluation-Server, and Data Analytic services  
- 🐳 Ready-to-run Docker examples with Seq, Jaeger, and Prometheus
- 🔌 Integration guidance for Datadog, New Relic, Grafana, and OpenTelemetry-compatible backends
- 📈 Updated plugin from 13 to 14 specialized skills

**Documentation**: https://docs.featbit.co/integrations/observability/opentelemetry

---

**Full Changelog**: https://github.com/featbit/featbit-skills/compare/v1.1.2...v1.2.0
```

---

### Emoji Usage in Release Notes

**Required Emojis for Sections:**
- `🎉` - Main "What's New" section header
- `🚀` - Getting Started section
- `📦` - Skills list section
- `🔄` - What Changed section
- `📚` - Resources section
- `🙏` - Support section
- `📊` - Statistics section

**Category Emojis:**
- `✨` - Added features
- `📈` - Improvements/Enhancements
- `🔧` - Changes/Updates
- `📝` - Documentation updates
- `🐛` - Bug fixes
- `⚠️` - Deprecations
- `❌` - Removals
- `🔒` - Security fixes
- `⭐` - Highlight/NEW badge

**Feature Description Emojis:**
- `📊` - Monitoring/Analytics/Metrics
- `🔧` - Configuration/Setup
- `🐳` - Docker/Containers
- `🔌` - Integrations/Plugins
- `📈` - Performance/Optimization
- `☸️` - Kubernetes
- `🌐` - Web/Browser/Network
- `📱` - Mobile
- `⚡` - Fast/Quick features
- `🛡️` - Security/Safety
- `📖` - Documentation
- `🎯` - Targeting/Precision

---

## 📌 Versioning

Follow [Semantic Versioning 2.0.0](https://semver.org/):

### Version Format: `MAJOR.MINOR.PATCH`

- **MAJOR**: Incompatible API changes or breaking changes
- **MINOR**: New features added in a backwards-compatible manner
- **PATCH**: Backwards-compatible bug fixes

### When to Increment

**MAJOR (x.0.0):**
- Breaking changes to skill structure
- Removing skills
- Incompatible plugin manifest changes
- Major architecture changes

**MINOR (0.x.0):**
- Adding new skills
- Adding new features to existing skills
- Non-breaking enhancements
- New integrations

**PATCH (0.0.x):**
- Bug fixes
- Documentation updates
- Typo corrections
- Minor improvements

### Examples

- `1.0.0` → `1.1.0`: Added new OpenTelemetry skill (new feature)
- `1.1.0` → `1.1.1`: Fixed typo in documentation (bug fix)
- `1.9.0` → `2.0.0`: Removed deprecated skills (breaking change)

---

## 💬 Commit Messages

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat`: New feature (correlates with MINOR version)
- `fix`: Bug fix (correlates with PATCH version)
- `docs`: Documentation only changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Examples

**Good Commit Messages:**
```
feat(opentelemetry): add OpenTelemetry observability skill

- Add comprehensive guidance for monitoring FeatBit services
- Include configuration for Api, Evaluation-Server, and Data Analytic
- Provide ready-to-run examples with Seq, Jaeger, and Prometheus

Closes #123
```

```
fix(dotnet-sdk): correct namespace in code examples

The namespace was incorrectly shown as FeatBit.Sdk instead of 
FeatBit.Sdk.Server in ASP.NET Core examples.
```

```
docs(readme): update installation instructions for v1.2.0

- Update version badges
- Add new OpenTelemetry skill to skills list
- Renumber all subsequent skills
```

---

## 📝 Documentation

### Skill Documentation Format

Each skill MUST follow this structure in `SKILL.md`:

```markdown
---
name: skill-name
description: One-line description of the skill
---

# Skill Title

One paragraph introduction.

## Overview

Brief overview of what this skill covers.

## Core Knowledge Areas

### 1. {Topic Area}

**{Subsection}:**
- Point 1
- Point 2

### 2. {Topic Area}

...

## Best Practices

1. **Practice 1**: Description
2. **Practice 2**: Description

## Documentation Reference

Official documentation: {URL}

## Related Topics

- Related Topic 1
- Related Topic 2
```

### README Updates

When adding a new skill:

1. Update skill count in `## 🎯 What is This?` section
2. Add new skill entry in `## 🎓 Available Skills` section
3. Maintain proper numbering for all skills
4. Update version badge
5. Add example usage if applicable

---

## 🔄 Release Checklist

Before creating a release, ensure:

- [ ] Version bumped in `package.json`
- [ ] Version bumped in `.claude-plugin/plugin.json`
- [ ] Version bumped in `.claude-plugin/marketplace.json`
- [ ] Version badge updated in `README.md`
- [ ] Skill count updated in all files (if applicable)
- [ ] New skill added to `package.json` skills array (if applicable)
- [ ] `CHANGELOG.md` updated with new version
- [ ] All files committed with proper commit message
- [ ] Git tag created: `git tag -a v{VERSION} -m "{MESSAGE}"`
- [ ] Changes pushed: `git push origin main`
- [ ] Tag pushed: `git push origin v{VERSION}`
- [ ] GitHub Release created using template from this guide

---

## 📄 File Structure

```
featbit-skills/
├── .claude-plugin/
│   ├── plugin.json          # Claude plugin manifest (version must match)
│   └── marketplace.json     # Marketplace catalog (version must match)
├── docs/
│   ├── INSTALLATION.md      # Installation guide
│   ├── QUICKSTART.md        # Quick start guide
│   └── ...
├── skills/
│   ├── {skill-name}/
│   │   └── SKILL.md         # Skill documentation
│   └── ...
├── AGENTS.md                # This file - guidelines for AI agents
├── CHANGELOG.md             # Version history (must be updated)
├── package.json             # NPM package manifest (version must match)
├── README.md                # Main readme (version badge must match)
└── LICENSE

```

---

## ✅ Quality Standards

### Code Examples

- Always use proper syntax highlighting
- Include complete, runnable examples
- Provide framework-specific examples when relevant
- Show both simple and advanced usage

### Documentation

- Use clear, concise language
- Include emojis for better readability
- Provide links to official documentation
- Add "activates when" section for each skill

### Testing

Before any release:
- Verify all links are working
- Check that version numbers match across all files
- Ensure skill files are properly formatted
- Test installation instructions

---

## Language and Tone

**Use English (US)** consistently throughout all documentation and commit messages. Maintain a professional yet approachable tone. Avoid jargon unless necessary, and always explain technical terms when first introduced.

## 🤝 Collaboration Guidelines

When working on this project:

1. **Always reference this guide** before creating releases
2. **Maintain consistency** in formatting and structure
3. **Update this guide** if you identify missing standards
4. **Use the provided templates** - don't freestyle
5. **Keep version numbers in sync** across all files

---

*Last Updated: January 30, 2026*
*Version: 1.0.0*
