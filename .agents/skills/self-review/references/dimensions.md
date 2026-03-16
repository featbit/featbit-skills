# Dimension Details

## TOC

- [Priority Dimensions (Progress-centric)](#priority-dimensions-progress-centric)
  - [1. Progress <> Design](#1-progress--design)
  - [2. Progress <> Artifact](#2-progress--artifact)
  - [3. Progress <> Skill](#3-progress--skill)
- [Deep Dimensions](#deep-dimensions)
  - [4. Design <> Artifact](#4-design--artifact)
  - [5. Design <> Skill](#5-design--skill)
  - [6. Artifact <> Skill](#6-artifact--skill)

Detailed checks for all 6 audit dimensions. Dimensions 1-3 are Progress-centric (highest priority). Dimensions 4-6 are cross-pillar deep checks. This is the complete C(4,2) combination of the 4 pillars.

All dimensions are guided by the **Audit Principles** defined in SKILL.md. The 6 principles are embedded as check questions in the dimensions where they apply most — look for the `[principle]` tags.

---

## Priority Dimensions (Progress-centric)

### 1. Progress <> Design

Are we aligned with design intent?

**Scope rule:** Only check alignment for in-scope items. Deferred items only need to be tracked, not completed.

| What to check | How | Common drifts |
|---|---|---|
| Phase alignment | Compare progress phase status vs design milestones | Progress says "Phase 2 done", design goals not met |
| Priority ordering | Verify progress priorities match design priorities | Design says A before B, progress has B first |
| Scope consistency | Compare planned scope in progress vs design scope | Design expanded, progress still lists old scope |
| Direction check | Is the trajectory moving toward the design vision? | Progress shows activity, but drifting from design goals |
| Deferred items | Verify deferred items are tracked somewhere | Silently dropped — not in progress, not in design |
| Dependency ordering | Check progress respects design's dependency chain | Phase 3 item started, but Phase 2 dependency unfinished |
| Decision recording | Non-obvious design decisions documented with rationale | "Why did we do X?" — no one remembers |
| Changelog coverage | New features/breaking changes have design docs or changelog entries | Feature shipped, no documentation of why or how |
| `[E1 Feasible]` | Can current tech/dependencies/resources support this design? Any dependency on capabilities that don't exist yet? | Design requires API that was deprecated; timeline assumes 2 engineers but only 1 available |

**Non-code examples:**
- Video project: Script calls for 5 scenes, progress only schedules 3
- Documentation project: Outline covers 4 topics, progress only has 2 marked done

### 2. Progress <> Artifact

Does claimed status match actual deliverables?

**Scope rule:** Only verify artifacts that Progress claims are in-scope and done/in-progress. Do not flag missing artifacts for deferred items.

| What to check | How | Common drifts |
|---|---|---|
| Completion claims | Cross-ref "done" items in progress vs actual artifacts | Marked done but artifact incomplete or missing |
| Pending accuracy | Verify "pending" items are actually unstarted | Work started but progress not updated |
| Artifact inventory | Check for artifacts not reflected in progress | Built something, forgot to track it |
| Status freshness | Compare artifact timestamps vs progress update timestamps | Artifact changed after last progress update |
| Deliverable evidence | For each claimed deliverable, verify it exists and works | Progress claims "tests pass" but tests don't exist |
| Gap detection | Items in progress with no corresponding artifact | Planned but never started, or started and abandoned |
| Verification depth | Check artifact passes its own success criteria, not just exists | Marked done, artifact exists, but doesn't actually work |
| Progress accuracy | Progress file reflects actual state of work | Code merged, progress still says "in progress" |
| Cross-repo sync | If docs live in a separate repo, both repos are updated | Code committed, design doc not written |
| `[T1 Verifiable & Verified]` | For each claimed-complete deliverable: where is the verification evidence? Can you run it and see the result? | Progress says "tests pass" — but no test output, no CI log, no evidence |
| `[E2 Boundary-Complete]` | Within current scope, is there any input/scenario that causes unhandled failure? Are error paths covered? | Happy path works, but invalid input crashes without message |
| `[E3 Maintainable]` | Could someone else pick this up and continue? Are key decisions recorded? | Code works but no comments on non-obvious choices; no onboarding path |

**Non-code examples:**
- Video project: Progress says "rough cut complete", but exported file is only half the expected duration
- Design project: Progress says "homepage design done", but mockup is missing mobile breakpoint

#### Artifact Verification Depth

Checking "does the artifact exist?" is necessary but insufficient. Dimension 2 must both **assess** and **execute** verification to the appropriate depth.

| Level | What it checks | Code example | Non-code example |
|-------|---------------|-------------|-----------------|
| **L1 Exists** | Artifact was produced | File exists | Video/article generated |
| **L2 Build** | Follows conventions and standards | tsc passes, lint clean | Format correct, axes labeled |
| **L3 Integration** | Business logic works correctly | Runtime execution, CLI commands produce expected output | Content reads correctly, config takes effect |
| **L4 E2E** | Meets design intent end-to-end | Full user workflow verified | Complete user scenario validated |

**Minimum bar: L3 (Integration).** L2-only (compilation) is insufficient for any artifact with behavioral changes. L4 when design expectations demand it.

#### Verification Execution Protocol

Self-review MUST execute verification, not just report that it's missing. The process:

1. **Determine design expectation**: Read the design/progress docs. What does "done" mean for this artifact? "Compiles" vs "runs correctly" vs "handles edge cases" are different bars.

2. **Execute tests at the appropriate level**:
   - L2 (Build): Run the build/compile/lint command. Report pass/fail.
   - L3 (Integration): Run the artifact. For code: execute CLI commands, call functions, verify output. For configs: apply and check effect. For content: verify it renders/reads correctly.
   - L4 (E2E): Run the full user workflow. For a CLI tool: simulate a real usage scenario. For an API: call it with real inputs and verify responses.

3. **Report results quantitatively**:
   - What was tested (specific commands/scenarios)
   - What passed / what failed (with actual output)
   - Current verification level achieved (L1/L2/L3/L4)
   - Gap to design expectation (e.g., "design expects L4, achieved L3 — E2E not tested")

4. **If a test cannot be automated**, you must:
   - Exhaust all options first. "I can't test this" is often wrong — investigate before declaring it untestable.
   - If truly untestable (requires human interaction, costs excessive tokens, needs external services): document exactly what needs manual verification and surface it as a **human-test-required** item in the report.

#### Verification Depth by Artifact Type

| Artifact type | Minimum level | How to verify |
|---------------|--------------|---------------|
| Code (library/module) | L3 | Import and call key functions, verify return values |
| Code (CLI command) | L3 | Execute the command with real args, check output |
| Code (daemon/service) | L3 | Start it, send requests, verify responses |
| Code (bug fix) | L3 | Reproduce the original bug scenario, verify it's fixed |
| Configuration | L3 | Apply the config, verify the system behaves differently |
| Skill/documentation | L2 | Verify structure, check internal links resolve, no stale references |
| Infrastructure (CI/CD) | L3 | Trigger the pipeline or simulate trigger, verify it runs |
| Design document | L2 | Structure complete, internally consistent, covers declared scope, no contradictory statements |
| Article / blog post | L2 | Arguments supported, no factual errors, structure matches outline |
| Video | L2 | Check file exists + metadata (duration, codec). Visual/audio content requires human verification — surface as human-test-required |
| Visual design (UI/graphic) | L2 | Check file exists + dimensions/format. Visual quality requires human verification — surface as human-test-required |
| Presentation / slides | L2 | Flow logical, key points covered, no orphan slides |

**L4 (E2E) is required when**: the design doc, progress, or acceptance criteria explicitly state end-to-end behavior, or when the artifact is a user-facing workflow with multiple interacting components.

### 3. Progress <> Skill

Any lessons to capture? Existing skills need updating?

| What to check | How | Common drifts |
|---|---|---|
| New skill opportunities | Review completed work for reusable patterns worth capturing | Solved a hard problem, didn't deposit the knowledge |
| Skill staleness | Check if existing skills still match current practices | Skill says "do X", but we've since learned "do Y" |
| Process compliance | Did execution follow documented skills/methods? | Skill says "always review", no review evidence |
| Conflict detection | Do recent learnings contradict existing skills? | New finding invalidates old skill rule |
| Skill coverage gaps | Are there active areas with no corresponding skill? | Doing complex work repeatedly with no documented method |
| Deposit backlog | List skills that should exist but don't yet | Pattern used 3+ times but never formalized |

**Non-code examples:**
- Video project: Discovered a reliable audio-sync workflow during rough cut, but didn't document it — next editor will reinvent it
- Research project: Developed a citation verification checklist across 3 papers, never formalized into a reusable template

#### Skill Deposit Criteria

When this dimension flags potential skill deposits, evaluate each candidate against these criteria. A pattern is worth depositing when it meets **at least one** of:

1. **Cross-project reusable** — The pattern applies across 3+ different projects. Deposit as a global skill.
2. **User-facing in a skills project** — The project itself IS a skills/tools project. Patterns that capture user intent, trigger timing, or solve user-facing interaction problems are worth depositing because users will hit these same issues.
3. **Intra-project SOP** — Even within a single project, if the pattern forms a repeatable workflow (editing technique, debugging SOP, review checklist) that you'll use again and again, it's worth depositing.

**NOT worth depositing:**

- **Code is the documentation** — If the pattern is already well-expressed in code + inline comments, extracting it adds maintenance burden with no value. Code IS the best documentation for implementation patterns.
- **Pure annotations** — Comments-level insights don't need separate files.
- **Wrong granularity** — Too fine (no application scenario, e.g. "always use setTimeout not setInterval for one-shot delays") or too coarse (covers everything, e.g. "write good code"). Right granularity: someone encountering the same class of problem would benefit from reading this before starting.
- **No counter-intuitive trap** — If the "obvious" approach works fine, there's nothing to deposit. Skills capture the non-obvious.

#### How to Report Skill Deposit Candidates

For each candidate pattern flagged by this dimension:

1. State the pattern briefly
2. Evaluate against the criteria above — which conditions does it meet?
3. Give a clear recommendation: **deposit** / **don't deposit** / **revisit when [condition]**
4. If "don't deposit": explain why (usually "code is the doc" or "not reusable enough")

---

## Deep Dimensions

### 4. Design <> Artifact

Does what was built match what was designed?

**Scope rule:** Only compare design specs against artifacts that are in-scope per the locked scope. A designed feature that belongs to a future phase is not a defect.

| What to check | How | Common drifts |
|---|---|---|
| Spec vs output | Compare design specs/requirements against actual deliverables | Feature designed but not built, or built differently |
| Architecture adherence | Compare designed structure vs actual structure | Layer bypass, unauthorized dependency |
| Naming consistency | Check artifact names match design vocabulary | Design says "deck", artifact says "task" |
| Decision compliance | For each recorded decision, verify artifact follows it | Decision made but artifact does the opposite |
| Rejected alternatives | Verify artifact doesn't use explicitly rejected approaches | Rejected approach snuck in during execution |
| Scope boundaries | Compare designed scope vs what artifact actually covers | Scope creep or under-delivery |
| Simplicity check | Is the artifact the simplest sufficient implementation of the design? | Design calls for X, artifact implements X + Y + Z "just in case" |
| User-facing doc | Changes to user-visible behavior reflected in README/SKILL.md | CLI flag added, no mention in docs |
| `[T3 Simplest Sufficient]` | Can any part be removed without losing designed capability? Is there a simpler approach with equal effect? | Three abstraction layers where one would do; config system for two options |
| `[E1 Feasible]` | Does the artifact depend on anything unavailable or unrealistic? | Uses an API that requires enterprise license; assumes 100ms latency on a 2s connection |

**Non-code examples:**
- Documentation project: Design says "for beginners", but article uses heavy jargon without explanation
- Video project: Design requires "light, humorous tone", but the final cut has slow, somber pacing

#### Simplicity Check (Principle: Simplest Sufficient Solution)

This is the primary dimension where the simplicity principle is evaluated:

- **Over-engineering**: Code — unnecessary abstraction layers. Docs — unnecessary chapters or classification schemes. Video — unnecessary effects or transitions that don't serve the story.
- **Unnecessary entities**: Code — modules that could be deleted. Design — concepts or roles that add complexity without value. Video — scenes that don't advance the narrative.
- **Premature generalization**: Code — solving a more general problem than designed. Docs — covering topics outside the stated scope "just in case". Design — accommodating users that aren't in the target audience.
- **Simpler alternatives**: Across all domains — could the same design intent be achieved with fundamentally fewer moving parts?

Flag only when a simpler alternative exists with equal capability. "Simple" does not mean "fewer lines" — it means fewer concepts, fewer moving parts, fewer things that can break.

### 5. Design <> Skill

Do our methods support our design goals?

| What to check | How | Common drifts |
|---|---|---|
| Principle-method alignment | For each design principle, check if a supporting skill exists | Design says "test-driven" but no testing skill |
| Constraint enforcement | Are design constraints backed by enforceable methods? | Design says "no direct DB access" but no skill enforces it |
| Vocabulary alignment | Do skills use the same terms as design docs? | Design says "artifact", skill says "deliverable" |
| Coverage | Are all critical design areas supported by skills? | Design has 5 pillars, skills only cover 2 |
| Anti-pattern prevention | Do skills guard against design's rejected alternatives? | Design rejected approach X, no skill warns against it |

**Non-code examples:**
- Video project: Design principle says "prioritize storytelling over production value", but no editing skill enforces this — editors default to polishing visuals
- Documentation project: Design says "write for international audience", but no style guide addresses localization-friendly writing

### 6. Artifact <> Skill

Does the output follow established methods?

| What to check | How | Common drifts |
|---|---|---|
| Standards adherence | Check artifact against documented standards | Standard exists but artifact violates it |
| Template conformance | Compare templates vs what artifact actually produces | Template updated, artifact still uses old format |
| Skill accuracy | Compare skill descriptions vs actual artifact capabilities | Skill claims feature X, artifact doesn't have it |
| Behavioral rules | Check if skill rules are actually enforced | Rule says "never do X", but artifact does X |
| Quality gates | Verify documented quality checks were actually applied | Checklist exists but wasn't followed |
| Artifact-as-skill check | If artifact IS a skill, does it follow skill conventions? | Built a SKILL.md that doesn't match the skill template |
| Template sync | Duplicated docs (e.g., check.md in two locations) are in sync | Source of truth updated, copy stale |
| `[E3 Maintainable]` | Can someone new understand and modify this artifact by following the documented skills? | Skill says "use pattern X", artifact uses undocumented pattern Y — handoff breaks |

**Non-code examples:**
- Video project: Team style guide says "standard transitions use dissolve", final cut uses jump cuts
- Documentation project: Writing standard says "max 5 sentences per paragraph", article has multiple 10+ sentence paragraphs
