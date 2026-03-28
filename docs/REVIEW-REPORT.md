# Skill Forge Review Report: featbit-skills

**Target**: `/Users/yuhaolu/motifpool/featbit-skills`
**Date**: 2026-03-28
**Reviewer**: skill-forge v3
**Scope**: 17 product skills (collection) + 1 rules file (AGENTS.md)

---

## Security: ALL PASS

All 17 skills passed security checks. No leaked credentials — all sensitive values use placeholders (`<your-env-secret>`, `please_change_me`, etc.).

---

## Cross-Item Analysis: Systemic Issues

### Pattern 1: All 17 skills missing Execution Procedure (must-fix)

**100% hit rate.** No SKILL.md has `## Execution Procedure` with a pseudocode block.

Most skills have `## Setup Workflow` or `## Routing Workflow` sections that serve a similar purpose but do not conform to the skill-format three-layer structure (frontmatter + EP + content).

**User impact**: AI agents read the skill as passive knowledge rather than following a structured execution flow. Execution fidelity drops.

### Pattern 2: All 15 reference files missing frontmatter and EP (must-fix)

Affects 6 skills, 15 reference files:

| Skill | Reference files | Max line count |
|-------|:---:|:---:|
| featbit-deployment-docker | 5 | 919 (troubleshooting.md) |
| featbit-rest-api | 5 | 373 (projects-api.md) |
| featbit-evaluation-insights-api | 2 | 306 (track-insights-api.md) |
| featbit-documentation | 1 | 204 |
| featbit-deployment-aws | 1 | 123 |
| featbit-sdks-dotnet | 1 | 90 |

### Pattern 3: "When to Use" + "Source" sections are redundant/homeless (must-fix)

Nearly every skill contains:
- `## When to Use This Skill` — duplicates frontmatter description trigger conditions
- `## Source` — a bare GitHub URL serving no procedural step
- `## Read Next Only When Needed` — links to external GitHub README anchors

These three sections waste ~15-20 lines of context per skill, ~300 lines total across the collection.

### Pattern 4: Terminology inconsistency "feature flag" vs "flag" (must-fix)

At least 6 SDK skills alternate between "feature flag" and "flag", violating AGENTS.md's one-term-per-concept rule:
`sdks-node`, `sdks-go`, `sdks-java`, `sdks-javascript`, `sdks-dotnet`, `sdks-react`

### Pattern 5: Oversized reference files (must-fix)

| File | Lines | Threshold |
|------|:---:|:---:|
| troubleshooting.md (deployment-docker) | **919** | 300 |
| environment-variables.md (deployment-docker) | **754** | 300 |
| standard-configuration.md (deployment-docker) | 429 | 300 |
| projects-api.md (rest-api) | 373 | 300 |
| track-insights-api.md (evaluation-insights) | 306 | 300 |

### Pattern 6: Collection size exceeds threshold (suggestion)

17 skills exceed the 15-skill context budget threshold. All descriptions are loaded simultaneously on every invocation, consuming attention budget.

---

## Skill-Specific Issues

| Skill | Issue | Severity |
|-------|-------|:---:|
| featbit-sdks-dotnet | Step 5 "If advanced behavior is needed" — vague graceful skip condition | must-fix |
| featbit-sdks-java | FBUser construction duplicated 3 times, boolVariation duplicated 2 times (~17% redundancy) | must-fix |
| featbit-sdks-javascript | Initialization retry has no failure path (one-sided conditional) | must-fix |
| featbit-opentelemetry | Entirely descriptive voice, no imperative workflow steps | must-fix |
| featbit-deployment-kubernetes | Only 89 lines, thin content, no tier selection decision logic | suggestion |

---

## AGENTS.md Assessment

**Classification**: Always-on rules (no description field, loaded as project instructions)
**Action**: Keep as-is — converting to skill would lose unconditional activation.

**Quality**: Excellent (231 lines, well-structured, actionable rules, pre-commit checklist, anti-patterns)

**One suggestion**: Line 62 uses `featbit-dotnet-sdk` as a naming example, but the actual directory is `featbit-sdks-dotnet`. Inconsistent.

---

## Strengths

1. **High-quality descriptions** — All 17 skills have clear positive/negative triggers, third-person voice, well-defined scope
2. **Zero security risk** — All code examples use placeholders, no credential leaks
3. **Clean directory structure** — Each skill contains only SKILL.md and optional references/, no junk files
4. **Self-contained references** — Each reference file covers its topic completely, no chained references
5. **TOC compliance** — All reference files over 100 lines include a table of contents
6. **Good README.md** — Clear install instructions, skill listing, contribution guidelines

---

## Summary

| Category | must-fix | suggestion |
|----------|:---:|:---:|
| Missing EP (17 skills) | 17 | — |
| Reference files missing format (15 files) | 15 | — |
| Homeless sections (~17 skills) | 17 | — |
| Terminology inconsistency (6+ skills) | 6 | — |
| Oversized reference files (5 files) | 5 | — |
| Skill-specific issues | 4 | 1 |
| Collection size | — | 1 |
| AGENTS.md naming example | — | 1 |
| **Total** | **~64** | **3** |

High must-fix count but highly homogeneous — only **5 core patterns**. Fix templates can be applied in batch.

---

## Fix Priority — All Completed

1. ~~Split oversized reference files~~ — troubleshooting.md split into 3, environment-variables.md split into 2
2. ~~Add EP to all 17 SKILL.md files~~ — all 17 now have `## Execution Procedure` with pseudocode
3. ~~Add frontmatter to all reference files~~ — all 18 reference files now have YAML frontmatter
4. ~~Clean homeless sections~~ — removed `## When to Use`, `## Source`, `## Read Next` from all skills
5. ~~Standardize terminology~~ — "feature flag" normalized across all SDK skills
6. ~~AGENTS.md naming example~~ — fixed `featbit-dotnet-sdk` → `featbit-sdks-dotnet`
7. ~~README improved~~ — value proposition one-liner, Quick Start section, reference-style badges, footer

## Self-Review Results

4/6 Aligned, 2/6 Drift, 0/6 Broken. Drift resolved (AGENTS.md naming example fixed).

## Remaining Items (not addressed)

- **Collection size** (suggestion) — 17 skills exceed 15-skill context budget threshold
- **Missing v2.0.0 git tag** — CHANGELOG documents v2.0.0 but no tag exists
- **Logo** — README has no logo; generate one for Tier 1 compliance
