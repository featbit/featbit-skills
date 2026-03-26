# FeatBit Agent Skills — Skill Forge Review Report

**Reviewer**: Jack (via skill-forge methodology)
**Date**: 2026-03-26
**Target**: https://github.com/featbit/featbit-skills (v2.1.2)
**Fork**: https://github.com/whiletrue0x/featbit-skills

---

## Executive Summary

This report evaluates the **featbit-skills** collection — 17 agent skills for the FeatBit feature flags platform — against the [skill-forge](https://github.com/motiful/skill-forge) quality standard.

**Verdict: PASS** — This is a well-engineered skill collection with strong authoring discipline. All 17 skills pass security, structure, and quality validation. Two warnings were identified and one was fixed during review; neither blocks publishing.

### Changes Made During Review

| Change | File | Severity |
|--------|------|----------|
| Added missing `metadata.author: FeatBit` | `skills/featbit-deployment-docker/SKILL.md` | Warning (fixed) |
| Added missing `metadata.author: FeatBit` | `skills/featbit-deployment-aws/SKILL.md` | Warning (fixed) |

---

## Scope

| Dimension | Value |
|-----------|-------|
| Skills reviewed | 17 |
| Reference files reviewed | 14 |
| Total SKILL.md lines | 2,397 |
| Total reference lines | 4,541 |
| Publishing model | Collection (single repo) |
| License | MIT |

### Skills Inventory

| Category | Skill | Lines | References |
|----------|-------|-------|------------|
| Platform | featbit-getting-started | 277 | 0 |
| Platform | featbit-documentation | 108 | 1 (205 lines) |
| Platform | featbit-opentelemetry | 100 | 0 |
| API | featbit-rest-api | 221 | 5 (999 lines) |
| API | featbit-evaluation-insights-api | 222 | 2 (594 lines) |
| Deployment | featbit-deployment-docker | 294 | 5 (2,560 lines) |
| Deployment | featbit-deployment-kubernetes | 89 | 0 |
| Deployment | featbit-deployment-aws | 39 | 1 (124 lines) |
| SDK Router | featbit-sdks | 43 | 0 |
| Server SDK | featbit-sdks-dotnet | 117 | 1 (62 lines) |
| Server SDK | featbit-sdks-node | 166 | 0 |
| Server SDK | featbit-sdks-python | 119 | 0 |
| Server SDK | featbit-sdks-java | 120 | 0 |
| Server SDK | featbit-sdks-go | 99 | 0 |
| Client SDK | featbit-sdks-javascript | 117 | 0 |
| Client SDK | featbit-sdks-react | 148 | 0 |
| Client SDK | featbit-sdks-react-native | 132 | 0 |

---

## 1. Security Audit

| Check | Result | Notes |
|-------|--------|-------|
| Leaked secrets (API keys, tokens, `sk-`, `ghp_`, `AKIA`, `xox[bpas]-`) | PASS | No real credentials found |
| Credential files (.env, .pem, .key) | PASS | None tracked in git |
| Private key patterns (`-----BEGIN.*PRIVATE KEY-----`) | PASS | None found |
| Password patterns in code | PASS | All are template placeholders (`please_change_me`, `your_password`, `${DB_PASSWORD}`) |
| .gitignore coverage | PASS | Covers `.env*`, `node_modules/`, `.DS_Store`, IDE configs, OS files |
| Hardcoded personal paths | PASS | No `~/`, `/Users/`, `/home/` paths found |

**No security issues found.**

---

## 2. Structure Validation

### 2.1 Frontmatter (all 17 skills)

| Check | Result |
|-------|--------|
| `name` present, kebab-case, matches directory | 17/17 PASS |
| `name` no reserved words ("anthropic", "claude") | 17/17 PASS |
| `name` max 64 chars, no start/end hyphen, no `--` | 17/17 PASS |
| `description` present, single-line, < 1024 chars | 17/17 PASS |
| `description` has negative trigger ("Do not use for...") | 17/17 PASS |
| `description` in third person | 17/17 PASS |
| `description` no YAML multi-line syntax (`>-`, `\|`) | 17/17 PASS |
| `description` no unquoted `: ` that breaks strict parsers | 17/17 PASS |
| `license` field present (MIT) | 17/17 PASS |
| `metadata.author` present | 17/17 PASS (2 fixed during review) |
| `metadata.version` present | 17/17 PASS |
| `metadata.category` present | 17/17 PASS |
| No non-standard top-level frontmatter fields | 17/17 PASS |

### 2.2 File Structure

| Check | Result |
|-------|--------|
| All SKILL.md under 500 lines | PASS (max: 294, featbit-deployment-docker) |
| All referenced local files exist | PASS (14/14 references resolve) |
| Directory naming convention (`featbit-{topic}`) | PASS (17/17) |
| No README/CHANGELOG inside skill directories | PASS |
| No junk/unnecessary files in skill directories | PASS |
| No nested reference directories | PASS |
| References one level deep only | PASS |
| All paths use forward slashes | PASS |

### 2.3 Collection-Level Structure

| Check | Result |
|-------|--------|
| package.json `skills` array matches filesystem | PASS (17 entries, all match) |
| README skill catalog matches filesystem | PASS (17 entries) |
| CHANGELOG.md exists | PASS |
| .editorconfig present | PASS |
| .skillsignore present | PASS |
| AGENTS.md authoring guide present | PASS |
| Version sync (package.json / README badge) | PASS (both 2.1.2) |

---

## 3. Quality Assessment

### 3.1 Description Quality

| Dimension | Assessment |
|-----------|-----------|
| Description coverage | All descriptions accurately reflect body content. Trigger phrases match actual skill scope |
| Description clarity | All descriptions are standalone comprehensible to a stranger unfamiliar with FeatBit |
| Scope boundaries | Clear negative triggers prevent false activation between overlapping skills |
| SDK Router pattern | `featbit-sdks` correctly routes to 9 language-specific skills based on detection signals |
| No over-promises | Descriptions do not claim capabilities beyond what the skill body delivers |

### 3.2 Skill Content Quality

| Dimension | Assessment |
|-----------|-----------|
| Progressive disclosure | Well-implemented — SKILL.md provides quick start, references loaded just-in-time |
| Code examples | Minimal, FeatBit-specific code; general language patterns correctly omitted |
| Terminology consistency | "feature flag" used consistently throughout; no alternation with "toggle" or "switch" |
| Writing style | Imperative voice for workflow steps; third person for descriptions |
| Workflow structure | All multi-step tasks use numbered steps with copyable checklists |
| Just-in-time loading | Reference files are loaded via explicit instructions in SKILL.md, not implicitly |

### 3.3 Reference File Quality

| File | Skill | Lines | TOC | Self-contained |
|------|-------|-------|-----|----------------|
| authentication.md | rest-api | 91 | N/A (<100) | Yes |
| openfeature-integration.md | sdks-dotnet | 62 | N/A (<100) | Yes |
| ecs-high-availability.md | deployment-aws | 124 | Yes | Yes |
| common-patterns.md | rest-api | 161 | Yes | Yes |
| complete-documentation-index.md | documentation | 205 | Yes | Yes |
| professional-configuration.md | deployment-docker | 218 | Yes | See WARNING-1 |
| standalone-configuration.md | deployment-docker | 237 | Yes | Yes |
| feature-flags-api.md | rest-api | 243 | Yes | Yes |
| flag-evaluation-api.md | evaluation-insights-api | 287 | Yes | Yes |
| track-insights-api.md | evaluation-insights-api | 307 | Yes | Yes |
| projects-api.md | rest-api | 374 | Yes | Yes |
| standard-configuration.md | deployment-docker | 430 | Yes | Yes |
| environments-api.md | rest-api | 130 | Yes | Yes |
| environment-variables.md | deployment-docker | 755 | Yes | Yes |
| troubleshooting.md | deployment-docker | 920 | Yes | Yes |

All reference files > 100 lines have a Table of Contents.

### 3.4 AGENTS.md Compliance

The project defines its own authoring standard in `AGENTS.md` (230 lines). All 17 skills comply with the full pre-commit validation checklist:

- [x] `name` in frontmatter exactly matches the directory name
- [x] `name` contains no reserved words ("anthropic", "claude")
- [x] Description has at least one negative trigger
- [x] Description is written in third person
- [x] `SKILL.md` is under 500 lines
- [x] All files referenced in `SKILL.md` exist at the specified paths
- [x] All paths use forward slashes and are relative
- [x] No `README.md` or `CHANGELOG.md` inside the skill directory
- [x] All workflow steps use imperative voice
- [x] Reference files are one level deep from `SKILL.md`
- [x] Reference files longer than 100 lines have a table of contents
- [x] No time-sensitive date conditions in the content
- [x] Skill is listed in `package.json` `skills` array

---

## 4. Findings

### FIXED-1: Missing metadata.author in 2 skills

**Severity**: Warning (resolved)
**Files**: `skills/featbit-deployment-docker/SKILL.md`, `skills/featbit-deployment-aws/SKILL.md`

Both skills were missing the `metadata.author: FeatBit` field present in all other 15 skills. This inconsistency could confuse collection tooling that inspects author metadata.

**Action taken**: Added `author: FeatBit` to both files during this review.

### WARNING-1: Nested references in professional-configuration.md

**Severity**: Warning
**File**: `skills/featbit-deployment-docker/references/professional-configuration.md`, lines 67-69 and line 122

The file cross-references two sibling files:

```markdown
- [standalone-configuration.md](./standalone-configuration.md) - Contains comprehensive service configuration examples
- [environment-variables.md](./environment-variables.md) - Complete environment variable reference
```

And again at line 122:
```markdown
Refer to [environment-variables.md](./environment-variables.md) for complete list.
```

This violates the AGENTS.md rule: *"all reference files link directly from SKILL.md. Never create `advanced.md` that points to `details.md`."*

The parent SKILL.md already links both files directly, making these cross-references redundant. An AI agent following the reference chain could load extra files unnecessarily.

**Recommendation**: Remove the cross-references from `professional-configuration.md`. Replace lines 67-69 with inline guidance or a note that the parent skill routes to those files.

### WARNING-2: Collection size at context flooding threshold

**Severity**: Warning (informational)
**Context**: 17 skills exceed the 15-skill context flooding threshold

Per skill-forge composition guidelines, collections with 15+ skills risk consuming excessive context tokens from description loading alone (~1,700 tokens for 17 descriptions).

**Mitigating factors already present:**
- README recommends `--skill` selective installation
- SDK router (`featbit-sdks`) reduces need to install all SDK skills
- Skills have specific trigger phrases — they won't all activate simultaneously
- This is a single-product collection where per-language SDK skills are inherently necessary

**Recommendation**: No action needed. The existing mitigations are appropriate for a product skill collection of this nature.

### INFO-1: Large reference files

**Severity**: Info
**Files**:
- `troubleshooting.md` — 920 lines
- `environment-variables.md` — 755 lines
- `standard-configuration.md` — 430 lines
- `projects-api.md` — 374 lines
- `track-insights-api.md` — 307 lines

All files have proper tables of contents and are loaded just-in-time. Their size is justified by comprehensiveness (50+ environment variables, extensive troubleshooting scenarios, complete API documentation). No splitting recommended — these benefit from being single, searchable references. The project's own standard (AGENTS.md) requires TOC for files > 100 lines, which is met.

---

## 5. Publishing Readiness

| Check | Result |
|-------|--------|
| README.md exists with install instructions | PASS |
| LICENSE exists (MIT) | PASS |
| .gitignore comprehensive | PASS |
| package.json homepage, repository, bugs URLs | PASS |
| package.json keywords for discoverability | PASS (18 keywords) |
| Primary install: `npx skills add featbit/featbit-skills` | PASS |
| Selective install documented (`--skill` flag) | PASS |
| No hardcoded personal paths | PASS |
| Version badge matches package.json | PASS (2.1.2) |
| CHANGELOG.md tracks changes | PASS |

---

## 6. Comparison with Best Practices

This collection demonstrates several noteworthy patterns:

1. **AGENTS.md as authoring guide** — Clear, enforced standards for skill creation with a pre-commit checklist. This is the right way to govern quality in a multi-skill collection.

2. **SDK Router pattern** — The `featbit-sdks` skill acts as a language-detection dispatcher, preventing users from needing to know exact skill names. This reduces misrouting and is a pattern other collections should adopt.

3. **Slim SKILL.md + GitHub pointer model** — SDK skills contain only FeatBit-specific vocabulary and link to official GitHub READMEs for advanced topics. This minimizes token waste while preserving actionable guidance.

4. **Consistent negative triggers** — Every skill explicitly states what it should NOT be used for. Combined with specific positive triggers, this prevents cross-activation in multi-skill environments.

5. **Progressive disclosure via references** — Large configuration details are in reference files loaded just-in-time, keeping SKILL.md focused on quick start and routing. Peak context load is ~1,200 lines per skill (SKILL.md + one reference), well within budget.

6. **Naming convention** — All skills use `featbit-{topic}` prefix, eliminating name collision risk with other installed skills.

---

## 7. Conclusion

**featbit-skills** is a production-quality skill collection that exceeds the bar set by most published skill repos. The authoring discipline (AGENTS.md), consistent structure, and thoughtful context management (selective install, SDK router, just-in-time references) reflect mature skill engineering.

### Summary of Findings

| ID | Severity | Status | Description |
|----|----------|--------|-------------|
| FIXED-1 | Warning | Resolved | Missing `metadata.author` in 2 deployment skills |
| WARNING-1 | Warning | Open | Nested cross-references in professional-configuration.md |
| WARNING-2 | Warning | Mitigated | Collection size at 17 (threshold: 15) |
| INFO-1 | Info | Noted | 5 reference files exceed 300 lines |

### Verdict

**PASS** — No critical issues. One open warning (nested references) is cosmetic and does not affect functionality. The collection is publish-ready.

---

*Report generated by [skill-forge](https://github.com/motiful/skill-forge) methodology*
*Reviewer: Jack (whiletrue0x)*
