# Self-Review

> Structured reflection for any creative work — code, content, design, research.

[Agent Skills](https://agentskills.io) compatible — works with Claude Code, Cursor, Codex, OpenClaw, and any supporting platform.

## Who Is This For

Anyone producing work with intent.

- **Developers**: Are you building what you designed? Does the code match the spec?
- **Content creators**: Does the final cut match the script? Did you capture your editing lessons?
- **Designers**: Does the mockup address the user need? Is there a simpler layout?
- **Researchers**: Do the conclusions follow from the method? Is the progress tracking accurate?
- **Teams**: Are design, delivery, and documentation in sync across contributors?

You don't need formal documents to use Self-Review. If your "design" is a mental model and your "progress" is a git log — that's enough. The 4 pillars are questions to ask, not documents to write.

## Why This Exists

Anyone who ships work in stages faces the same problem: things drift.

The design says one thing, the deliverable does another. Progress tracking says "done" but the artifact is half-finished. You learned a better method last week but the old pattern is still everywhere.

Self-Review forces a structured pause. It scans your project across **4 pillars** and checks **6 dimensions** of alignment between them. It works for anything with deliverables — code, content, research, design.

It's especially powerful as an AI agent skill — agents are goal-seeking and don't naturally pause to reflect. But the framework works for any creator reviewing their own work.

**Report-only** — it flags issues, never auto-fixes. You stay in control.

## Why These Principles

The 6 audit principles draw from two sources:

**Feynman's criteria for good theories** (The Character of Physical Law, 1964):
1. It must agree with experiment → **Verifiable & Verified**
2. It must be self-consistent → **No Internal Contradiction**
3. It must be the simplest explanation → **Simplest Sufficient Solution**

**Engineering constraints** that theory alone doesn't cover:
4. It must be buildable → **Feasible**
5. It must handle edge cases → **Boundary-Complete**
6. It must be handoff-ready → **Maintainable**

The theoretical layer tells you "is this a good idea?". The engineering layer tells you "can this actually ship?". Self-Review checks both.

## The 4 Pillars

Pillars are lenses for examining the same body of work — not file categories.

| Pillar | Core question | What it covers |
|--------|--------------|----------------|
| **Design** | Why are we doing this? What should it become? | Intent, decisions, constraints, specs |
| **Artifact** | What was actually produced? | Any deliverable — code, documents, designs, videos, skills, configs |
| **Skill** | How do we do things? What did we learn? | Reusable methods, standards, accumulated know-how |
| **Progress** | Where are we? What's next? | Plans, status, milestones, tracking |

The same deliverable can be examined through multiple pillars. A design doc is both Design (intent) and Artifact (deliverable). A pillar may have no dedicated files — intent might live only in commit messages, progress only in branch status.

## The 6 Dimensions

Checks alignment between every pillar pair — the complete C(4,2) combination with no exceptions.

**Priority (always run):**
1. **Progress <> Design** — Are we aligned with design intent?
2. **Progress <> Artifact** — Does claimed status match actual deliverables?
3. **Progress <> Skill** — Any lessons to capture?

**Deep audit:**
4. **Design <> Artifact** — Does the output match the design?
5. **Design <> Skill** — Do our methods support our design goals?
6. **Artifact <> Skill** — Does the output follow established methods?

## What Makes It Different

**Scope-aware auditing.** Before checking anything, Self-Review reads your Progress to lock the current phase. It classifies items as in-scope, deferred, or out-of-scope — then only audits in-scope work. "Phase 2 hasn't started yet" is never a finding.

**4-level verification depth.** Dimension 2 doesn't just check if files exist — it executes builds, runs CLI commands, and verifies output up to 4 levels deep:

| Level | What it checks | Example |
|-------|---------------|---------|
| L1 Exists | Artifact was produced | File exists |
| L2 Build | Follows conventions | `tsc` passes, lint clean |
| L3 Integration | Business logic works | CLI commands produce expected output |
| L4 E2E | Meets design intent end-to-end | Full user workflow verified |

Minimum bar is L3 (Integration). "Tests pass" means it ran the tests. Knows how to verify different artifact types — code, configs, documentation, video, design files — each with appropriate methods.

**Currency & assumptions check.** Before checking cross-pillar alignment, Self-Review evaluates the design itself and actively searches the web (with current-year queries) to verify assumptions haven't been invalidated. Findings are classified:

- **Current** — no relevant new solution
- **Emerging** — new solution exists but immature, noted but no action
- **Problem Dissolved** — established solution exists, flags: "verify before continuing custom work"
- **Assumption Invalidated** — a key design assumption no longer holds, flags with evidence

**Principled judgment.** Every finding is evaluated against 6 explicit principles (3 theoretical + 3 engineering), embedded as check questions directly in each dimension. This makes audits consistent and reviewable.

**Cross-platform quality standards.** Self-Review discovers your project's instruction files across platforms — `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.cursor/rules/`, `CONTRIBUTING.md` — and uses them as additional quality gates. It also discovers installed domain skills (e.g., a `video-editing` skill) and layers their rules on top.

**Standard recommendations.** When Self-Review finds recurring issues with no corresponding standard, it doesn't just flag the problem — it drafts the standard text, specifies the exact file path where it should be persisted (platform-aware: `.claude/rules/` for CC, `.cursor/rules/` for Cursor, `AGENTS.override.md` for Codex), and explains why it's worth adding.

**Implicit anchor inference.** No dedicated design docs? No progress.md? Self-Review infers from commit messages, PR descriptions, TODO comments, git log timelines, and code patterns before skipping any pillar. You don't need formal documents to get a meaningful audit.

**Skill deposit detection.** Dimension 3 evaluates whether lessons learned during execution are worth capturing as reusable skills. It applies specific deposit criteria (cross-project reusable? counter-intuitive trap? right granularity?) and gives a clear recommendation: deposit / don't deposit / revisit when [condition].

**Works for everything.** Not just code. Content projects, research, video production, documentation — anything with a design intent and deliverables.

## Usage

Say: `/self-review`, `self-review`, `audit`, or `审视一下`

The skill scans your project, discovers anchors in each pillar, locks the current scope from Progress, runs dimensional checks, and outputs a structured report with specific file paths and line numbers.

## Install

```bash
npx skills add motiful/self-review
```

Or manually:

```bash
git clone https://github.com/motiful/self-review ~/skills/self-review

# Claude Code
ln -sfn ~/skills/self-review ~/.claude/skills/self-review

# Other platforms (Cursor, Codex, Windsurf, etc.)
ln -sfn ~/skills/self-review ~/.agents/skills/self-review
```

## Example Output

### Example: Self-Review Auditing Itself (Real Output)

```
## Self-Review Report

### Design Introspection
- Clarity: well-defined — "4-pillar, 6-dimension alignment audit"
- Value: problem definition precise — "drift between intent and delivery"
- Simplicity: 6 principles = 3 theoretical + 3 engineering, no excess concepts

### Scope Lock
- Current phase: Published skill, post-refactor
- In-scope: SKILL.md, dimensions.md, README.md
- Deferred: none

### Anchors Found
- Design: README.md (design intent + principles)
- Artifact: SKILL.md, references/dimensions.md
- Skill: self-review is itself a skill (SKILL.md + references/)
- Progress: [inferred] git log on main

### Priority Dimensions

#### 1. Progress <> Design [Aligned]
- 6-dimension model consistent across all files
- Feynman + engineering principle split documented in README

#### 2. Progress <> Artifact [Aligned]
- SKILL.md frontmatter valid (name, description)
- All internal references resolve (dimensions.md exists, linked correctly)
- L2 verified: structure complete, no broken links

#### 3. Progress <> Skill [Aligned]
- No new deposit needed — stable published skill

### Deep Dimensions

#### 4. Design <> Artifact [Aligned]
- SKILL.md implements all 6 principles as embedded check questions
- dimensions.md covers all C(4,2) = 6 pillar combinations
- Report template matches process steps

#### 5. Design <> Skill [Aligned]
- Feynman principles mapped to T1/T2/T3, engineering to E1/E2/E3
- Skill deposit criteria in Dimension 3 enforces knowledge-persistence design goal

#### 6. Artifact <> Skill [Aligned]
- Frontmatter follows Agent Skills standard (name, description)
- Directory structure: SKILL.md + references/ at repo root per convention
- No junk files in skill content

### Summary
- 6/6 aligned, 0/6 drifted, 0/6 broken
```

### Example: Video Project (Hypothetical)

```
## Self-Review Report

### Scope Lock
- Current phase: Episode 3 rough cut
- In-scope: filming, rough edit, audio sync
- Deferred: color grading, final mix (post-production phase)

### Priority Dimensions

#### 1. Progress <> Design [Drift]
- Script calls for 5 scenes, only 4 filmed. Scene 3 (interview) not scheduled.

#### 2. Progress <> Artifact [Aligned]
- Rough cut exists, 12:34 duration (target: 12-15 min)
- Audio synced, no dropout detected
- L2 verified: playable, continuity intact

#### 3. Progress <> Skill [Aligned]
- Editing pattern: consistent use of J-cuts per team style guide

### Summary
- 5/6 aligned, 1/6 drifted, 0/6 broken
```

## What's Inside

```
SKILL.md              — Audit process, principles, and rules
references/
└── dimensions.md     — Detailed checks for all 6 dimensions
```

## License

[MIT](LICENSE)

---

Forged with [Skill Forge](https://github.com/motiful/skill-forge)
