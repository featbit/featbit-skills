# FeatBit Skills — Agent Instructions

This repo contains agent skills for FeatBit. The primary task is authoring and maintaining `SKILL.md` files under `skills/`. All other work (releases, versioning) supports that purpose.

## Project Structure

```
featbit-skills/
├── skills/
│   └── featbit-{topic}/
│       ├── SKILL.md         # required, <500 lines
│       ├── references/      # one level deep only
│       ├── scripts/         # deterministic helper CLIs
│       └── assets/          # output templates, schemas
├── package.json             # version must stay in sync
└── README.md                # version badge must stay in sync
```

## Skill Authoring Rules

### Core Principle: Concise is Key

The context window is a shared resource. Only add context the agent doesn't already have. Challenge each piece of information:

- "Does the agent really need this explanation?"
- "Can I assume the agent knows this?"
- "Does this paragraph justify its token cost?"

At startup, only `name` and `description` are pre-loaded. `SKILL.md` is read only when the skill becomes relevant, and reference files are read only as needed.

### Directory and Naming

- Every skill directory must be named `featbit-{topic}`.
- The `name` field in `SKILL.md` frontmatter must exactly match the directory name (e.g., `name: featbit-sdks-dotnet` lives in `skills/featbit-sdks-dotnet/SKILL.md`).
- `references/` is one level deep only. Never create `references/db/v1/schema.md`; use `references/schema.md`.
- Never create `README.md`, `CHANGELOG.md`, or `INSTALLATION.md` inside a skill directory.

### Frontmatter

```yaml
---
name: featbit-{topic}
description: <trigger-optimized, max 1024 chars>
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration | deployment | api-reference | platform
---
```

`name` rules: 1–64 characters, lowercase letters, numbers, and hyphens, no consecutive hyphens. No reserved words: "anthropic", "claude".

`description` rules:
- Third person, describes capability and when to use it.
- Include explicit positive triggers (user phrases, file extensions, keywords).
- Include at least one negative trigger ("Do not use for...") to prevent false activation.
- No XML tags in the value.

Bad: `FeatBit SDK help.`

Good: `Expert guidance for integrating FeatBit .NET Server SDK. Use when user asks about ".NET SDK", "C# feature flags", "ASP.NET Core FeatBit", or mentions .cs, .csproj files. Do not use for Node.js, Python, or client-side JavaScript questions.`

### SKILL.md Content

- Hard limit: 500 lines. Move excess content to `references/`.
- `SKILL.md` is the navigation brain — quick start + decision routing only.
- Do not inline large code blocks, full config files, or exhaustive API schemas. Put them in `references/` or `assets/`.
- Load reference files just-in-time: instruct explicitly when to read them.

Just-in-time loading pattern:
```markdown
Read `references/environment-variables.md` only when the user asks about a specific variable or config option.
```

### Degrees of Freedom

Match instruction specificity to the task's fragility:

- **High freedom** (prose guidance): Use when multiple valid approaches exist or context determines the best path. Example: "Review the flag evaluation logic and suggest improvements."
- **Medium freedom** (pseudocode/templates with parameters): Use when a preferred pattern exists but some variation is acceptable. Example: provide a code template with placeholder comments.
- **Low freedom** (exact scripts, no variation): Use when operations are fragile or a specific sequence must be followed. Example: "Run exactly: `python scripts/migrate.py --verify --backup`. Do not modify the command."

### Writing Style

- Use numbered steps for every workflow. Define a strict chronological sequence.
- Write in imperative for workflow steps: "Extract the flag key from..." not "You should extract..."
- Description field must be third person: "Provides guidance for..." not "I can help you..."
- Map decision branches explicitly: "Step 3: If the user is on ASP.NET Core, proceed to Step 4. Otherwise skip to Step 5."
- Use one term per concept throughout the file. Never alternate between "feature flag", "flag", and "toggle".
- Use domain-native terminology (e.g., "dependency injection container" for .NET, not just "setup").
- Do not include time-sensitive information. Use `<details>` blocks for legacy/deprecated patterns.

### Reference Files

- Each reference file addresses one topic completely and is self-contained.
- Name descriptively: `references/openfeature-integration.md`, not `references/extra.md`.
- `SKILL.md` must state exactly when and why to open each reference file.
- References longer than 100 lines should include a table of contents at the top.
- Never nest references: all reference files link directly from `SKILL.md`. Never create `advanced.md` that points to `details.md`.

### Workflows

For multi-step tasks, provide a copyable checklist followed by per-step detail:

```markdown
## Setup workflow

Copy and track progress:
- [ ] Step 1: Install SDK
- [ ] Step 2: Configure environment
- [ ] Step 3: Initialize client
- [ ] Step 4: Evaluate first flag

**Step 1: Install SDK**
Run: `pip install fb-python-sdk`

**Step 2: Configure environment**
...
```

For error-prone operations, include a validation feedback loop: run validator → fix errors → repeat.

### Scripts

- Scripts in `scripts/` must be tiny single-purpose CLIs (Python, Bash, or Node).
- Scripts emit descriptive stdout/stderr so the agent can self-correct without user intervention.
- Do not bundle library or application code in scripts.
- Provide utility scripts for deterministic, repeatable operations — pre-made scripts are more reliable than generated code.
- Make execution intent explicit: "Run `validate.py`" (execute) vs "See `validate.py` for the algorithm" (read).

### MCP Tool References

When referencing MCP tools in a skill, always use the fully qualified format to avoid "tool not found" errors:

```
Use the FeatBit:get_feature_flags tool to retrieve flags.
```

Format: `ServerName:tool_name`

### What to Remove

- Instructions for tasks the agent can handle reliably without help.
- Prose explanations — convert to numbered steps or move to `references/`.
- Marketing text ("FeatBit is a powerful platform that...").
- Information already covered by another skill (e.g., auth basics from `featbit-rest-api` need not repeat in SDK skills).
- Time-sensitive content ("before August 2025, use X") — use deprecated sections instead.

### Anti-Patterns

- **Too many options**: Don't list 5 libraries. Pick one default and mention the alternative only when the default won't work.
- **Deeply nested references**: Never chain `SKILL.md → advanced.md → details.md`. Keep all references one level deep from `SKILL.md`.
- **Windows-style paths**: Always use forward slashes (`references/guide.md`), never backslashes.
- **Vague descriptions**: Avoid `description: Helps with FeatBit`. Be specific about what and when.

### New Skill Checklist

When adding a new skill, complete all steps in order:

1. Create `skills/featbit-{topic}/SKILL.md` with valid frontmatter.
2. Add entry to `package.json` → `skills` array.
3. Update skill count in `README.md` ("What is This?" section).
4. Add skill entry to `README.md` ("Available Skills" section).
5. Update `CHANGELOG.md`.
6. Bump MINOR version in `package.json` and `README.md` version badge.

### Pre-commit Validation

Verify before every skill commit:

- [ ] `name` in frontmatter exactly matches the directory name
- [ ] `name` contains no reserved words ("anthropic", "claude")
- [ ] Description has at least one negative trigger
- [ ] Description is written in third person
- [ ] `SKILL.md` is under 500 lines
- [ ] All files referenced in `SKILL.md` exist at the specified paths
- [ ] All paths use forward slashes and are relative
- [ ] No `README.md` or `CHANGELOG.md` inside the skill directory
- [ ] All workflow steps use imperative voice
- [ ] Reference files are one level deep from `SKILL.md` (no chained references)
- [ ] Reference files longer than 100 lines have a table of contents
- [ ] No time-sensitive date conditions in the content
- [ ] Skill is listed in `package.json` `skills` array

## Versioning

Follow Semantic Versioning (`MAJOR.MINOR.PATCH`).

| Change | Bump |
|--------|------|
| Breaking changes, removed skills, incompatible manifest changes | MAJOR |
| New skill, new feature, non-breaking enhancement | MINOR |
| Bug fix, typo, doc update, minor improvement | PATCH |

Keep version in sync across both files simultaneously:
- `package.json`
- `README.md` version badge

## Release

Release title format: `v{MAJOR}.{MINOR}.{PATCH} - {3–6 word description in Title Case}`

Examples: `v1.5.0 - AWS Deployment Support`, `v2.0.0 - Breaking Architecture Changes`

Release checklist:
- [ ] Version bumped in both files
- [ ] `CHANGELOG.md` updated
- [ ] `git tag -a v{VERSION} -m "{title}"`
- [ ] `git push origin main && git push origin v{VERSION}`
- [ ] GitHub Release created with notes

## Commit Messages

Format: `<type>(<scope>): <subject>`

Types: `feat` (MINOR), `fix` (PATCH), `docs` (PATCH), `refactor` (PATCH), `chore` (no bump).

Scope is the skill name (e.g., `dotnet-sdk`, `deployment-docker`) or `release` for version bumps.

Examples:
```
feat(deployment-aws): add AWS ECS Fargate deployment skill
fix(dotnet-sdk): correct namespace in ASP.NET Core examples
chore(release): bump version to 1.5.0
```

## Language

Use English (US) for all skill content, commit messages, and documentation.
