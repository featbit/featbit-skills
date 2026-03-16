---
name: self-review
description: Run a 4-pillar, 6-dimension alignment audit on the current project. Use when the user says "self-review", "审视一下", or "audit". Report-only — never auto-fix.
license: MIT
metadata:
  author: motiful
  version: "2.0"
---

# Self-Review

Audit process: discover anchors in 4 pillars (Design, Artifact, Skill, Progress), lock scope from Progress, then check alignment across all 6 pillar-pair dimensions. Platform-agnostic, language-agnostic.

## Audit Principles

Six principles guide all dimension checks. They are embedded as check questions in each dimension — you don't need to apply them separately.

| # | Principle | One-line definition |
|---|-----------|-------------------|
| T1 | Verifiable & Verified | Claims must be testable, and evidence of passing must exist |
| T2 | No Internal Contradiction | No conflicting statements across pillars |
| T3 | Simplest Sufficient Solution | Don't introduce what isn't needed |
| E1 | Feasible | Achievable with current resources |
| E2 | Boundary-Complete | No fatal gaps within current scope |
| E3 | Maintainable | Others can understand and continue the work |

## The 4 Pillars

| Pillar | Core question | What it covers | Common carriers (examples, not definitions) |
|--------|--------------|----------------|---------------------------------------------|
| **Design** | Why are we doing this? What should it become? | Intent, decisions, constraints, specs | CLAUDE.md, design docs, requirements, even commit messages or verbal agreements |
| **Artifact** | What was actually produced? | Any deliverable — code, documents, designs, videos, skills, configs | src/, articles, exported videos, design files, SKILL.md itself |
| **Skill** | How do we do things? What did we learn? | Reusable methods, standards, accumulated know-how | Style guides, workflows, conventions, templates, coding patterns |
| **Progress** | Where are we? What's next? | Plans, status, milestones, tracking | progress.md, changelogs, git log, TODO comments, roadmaps |

## Process

### Step 1: Discover Anchors

Scan the project for anchors in each pillar (all optional — skip pillars with no anchors after attempting inference):

| Pillar | Typical anchors |
|--------|----------------|
| Design | `CLAUDE.md`, `**/design/*.md`, `**/docs/*.md`, `README.md` |
| Artifact | `src/`, `lib/`, `app/`, `dist/`, `output/`, `**/types.ts` |
| Skill | `.claude/skills/`, `skill/`, `**/references/*.md`, `**/templates/` |
| Progress | `**/progress.md`, `**/plan/*.md`, `CHANGELOG.md`, `**/roadmap.md` |

Report which anchors you found and which pillars have none.

#### Project-Specific Standards

Beyond the standard anchors, scan for project-specific quality standards that should inform the audit. These files vary by platform — check whichever exist:

| Platform | Project instructions | Local/private overrides | Rules/standards |
|----------|---------------------|------------------------|-----------------|
| Claude Code | `CLAUDE.md` | `CLAUDE.local.md` | `.claude/rules/*.md` |
| Cursor | `.cursor/rules/*.mdc` | `.cursor/rules/personal.mdc` (gitignored) | `.cursorrules` (legacy) |
| Codex | `AGENTS.md` | `AGENTS.override.md` | — |
| OpenClaw | `AGENTS.md`, `SOUL.md` | via `openclaw.json` | — |
| Generic | `README.md`, `CONTRIBUTING.md` | — | `.editorconfig`, linter configs |

If found, these standards become additional checks in cross-pillar dimensions. For example:
- `CLAUDE.md` says "capture-pane is debug only" → Dimension 4 checks code doesn't use capture-pane for core logic
- `AGENTS.override.md` says "run tests before committing" → Dimension 2 checks if test evidence exists
- `CONTRIBUTING.md` says "no direct pushes to main" → Dimension 6 checks branch workflow

#### Domain Skills

Scan `.claude/skills/` (or platform equivalent) for skills whose description matches the current project's domain. If found, treat the standards and rules defined in those skills as additional check criteria — layer them on top of the built-in principles when running dimensions.

Example: A `video-editing` skill defines "jump cuts must not exceed 3 frames" → Dimension 4 and 6 check this rule against the artifact.

Only include skills that are clearly relevant to the project. Do not force-apply unrelated skills.

#### Implicit Anchor Inference

When a pillar has no dedicated files, attempt to infer its content from implicit sources before skipping:

| Pillar | Implicit sources to check |
|--------|--------------------------|
| Design | Commit messages, PR descriptions, README intro, inline comments stating intent |
| Artifact | Any output file (this pillar is almost never empty) |
| Skill | Recurring code patterns, consistent conventions across files |
| Progress | Git log timeline, branch names, TODO/FIXME comments |

Mark inferred anchors as `[inferred]` in the report. If inference produces nothing useful, then skip the pillar with a note.

#### Design Introspection

When Design anchors are found, evaluate the design itself — not just its alignment with other pillars. Apply the following checks:

- **Clarity**: Is the stated goal well-defined? Could someone new understand the intent?
- **Value**: Is this a well-formulated problem? Is it worth solving in this form?
- **Scope**: Is the scope appropriate — not too broad, not too narrow?
- **Assumptions**: Are there stale assumptions, blind spots, or unconsidered alternatives?
- **Currency**: Is the project solving the right problem with current tools? (See detailed check below.)
- **Simplicity**: Is this the simplest sufficient solution? Can anything be removed without losing capability? Are there simpler alternatives with equal explanatory/functional power?

#### Currency & Assumptions Check

**Scope rule**: Only check currency for in-scope items that are pending, in-progress, or blocked. Completed, working items are NOT checked — don't fix what isn't broken. Don't search dependency versions (that's a package manager's job).

**Check process**:

1. Extract unsolved problems from Progress (pending/blocked items only)
2. For each unsolved problem, search: `"<problem description> solution <current year>"`
3. Evaluate search results:

| Finding | Label | Action |
|---------|-------|--------|
| No relevant new solution | Current | No action |
| Community solution exists but immature (<6 months, few adopters, no major platform support) | Emerging | Note in report, don't recommend switching |
| Community solution with significant adoption (major platform support, established project) | Problem Dissolved | Flag: "This problem may already be solved by X. Verify before continuing custom work." Include source URL |
| A key design assumption has been invalidated (API added, limitation removed, standard changed) | Assumption Invalidated | Flag: "Design assumes X, but Y is now supported as of [date]." Include source URL |

4. For Assumptions specifically: extract hard assumptions from Design anchors (patterns like "because X doesn't support Y", "since there's no way to Z", "limited by W"). Search each to verify it still holds.

**Always add the current year to search queries** (e.g., "React server components 2026").

For each major design decision identified above, ask these three questions:
1. Can this actually be built/delivered with the resources available? (What would block it?)
2. What scenario within the current scope would break this? (What's the weakest point?)
3. Is there a way to achieve the same goal with fewer moving parts? (What can be removed?)

If any question reveals a concern, report it. These questions only apply to decisions that significantly affect the project direction — skip for minor choices.

Report design introspection findings as a preamble before dimensional checks.

### Step 2: Lock Scope

Before running any dimension checks, establish the audit boundary from Progress:

1. **Read current phase** from Progress anchors. Identify what the project claims to be working on *right now*.

2. **Classify all items into three categories:**

| Category | Definition | Audit treatment |
|----------|-----------|-----------------|
| **In-scope** | Items belonging to the current phase/milestone | Full dimensional check against all 6 principles |
| **Deferred** | Items explicitly postponed to a future phase | Only check: is it tracked somewhere? Not lost silently? |
| **Out-of-scope** | Items not in any plan | Do not check. Do not flag absence as a defect |

3. **State the locked scope explicitly in the report** so the reader knows what was and wasn't audited.

**The core rule: do not judge current work by future plans.** A feature designed for Phase 3 that doesn't exist yet is not a defect in Phase 1. An architecture that will need refactoring later is acceptable if the current phase is independently testable and shippable.

Flag Phase 1 artifacts that depend on unbuilt Phase 2 — those are real defects. But "Phase 2 hasn't started yet" is never a finding.

### Step 3: Run 6 Dimensions

Each dimension checks drift between two pillars. This is the complete C(4,2) combination — no exceptions, no artificial extras.

**Priority (Progress-centric) — always run:**
1. **Progress <> Design** — Are we aligned with design intent?
2. **Progress <> Artifact** — Does claimed status match actual deliverables? **Must execute verification.**
3. **Progress <> Skill** — Any lessons to capture? Existing skills need updating?

**Deep audit — run in deep audits or when specifically relevant:**
4. **Design <> Artifact** — Does the output match the design?
5. **Design <> Skill** — Do our methods support our design goals?
6. **Artifact <> Skill** — Does the output follow established methods?

See `references/dimensions.md` for detailed checks per dimension.

### Step 4: Report

Output a structured report:

```
## Self-Review Report

### Design Introspection
- [findings on clarity, value, scope, assumptions, currency, simplicity]

### Scope Lock
- **Current phase**: [phase name/description]
- **In-scope**: [items being audited]
- **Deferred**: [items tracked but not yet due]
- **Out-of-scope**: [not applicable to this audit]

### Anchors Found
- **Design**: [paths]
- **Artifact**: [paths]
- **Skill**: [paths]
- **Progress**: [paths]

### Priority Dimensions (Progress-centric)

#### 1. Progress <> Design [Aligned / Drift / Broken]
- [specific findings with file paths, scoped to current phase]

#### 2. Progress <> Artifact [Aligned / Drift / Broken]
- [specific findings, with verification evidence]

#### 3. Progress <> Skill [Aligned / Drift / Broken]
- [specific findings]

### Deep Dimensions

#### 4. Design <> Artifact [Aligned / Drift / Broken]
- [specific findings, scoped to current phase]

#### 5. Design <> Skill [Aligned / Drift / Broken]
- [specific findings]

#### 6. Artifact <> Skill [Aligned / Drift / Broken]
- [specific findings]

### Summary
- X/6 aligned, Y/6 drifted, Z/6 broken
- [top priority fixes, if any]
- [skill deposit candidates with recommendation]
```

#### Standard Recommendations

If the audit finds recurring quality issues with no corresponding standard, include a **Standard Recommendation** in the report:
1. Specify the exact file path where the standard should be persisted (use the platform table in Step 1)
2. Draft the standard text ready to be added
3. Explain why this standard is worth adding (what recurring issue it prevents)

**Persistence routing:**
- **Project-local standards** (single-project, simple) → platform-native file: `.claude/rules/<topic>.md` (CC), `AGENTS.override.md` (Codex), `.cursor/rules/<topic>.mdc` (Cursor)
- **Cross-project reusable standards** → suggest creating a rule-skill via Skill-Forge. Rule-skills are portable across platforms and publishable — see the [rules-as-skills](https://github.com/motiful/rules-as-skills) methodology
- **Platform lacks native rules** (e.g., OpenClaw) → rule-skill is the only persistent option

Default to platform-native files (lower friction). Suggest rule-skills when the standard could benefit other projects. Always present both options — the user decides.

## Rules

- **Report only, never auto-fix** — flag issues, do not modify files. Exception: Dimension 2 MUST execute verification (build, run, CLI) — read-only execution.
- **Scope first** — lock scope from Progress before any dimension. Never flag deferred/out-of-scope items.
- **Progress first** — start from "what are we doing now?" and radiate outward.
- **Scan by content, not by file type** — a commit message can carry Design intent, a code comment can carry Progress status. Never skip content because "it's not in the right file type for this pillar."
- **Infer before skipping** — when a pillar has no explicit files, check implicit sources before declaring it empty.
- **Built-in principles + user standards** — the 6 principles are the baseline; project rules and domain skills add to it, never replace it.
- **Be specific** — cite file paths and line numbers.
- **No false positives** — only flag real drift. "Not yet built" is not drift if deferred.
- **Skill deposits need criteria** — evaluate against deposit criteria in dimensions.md (Dimension 3).
- **Always add current year to search queries** — for Currency and Assumptions checks.
- **Platform-agnostic, language-agnostic.**
