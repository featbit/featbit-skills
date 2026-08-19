---
name: Decision Framing Guide
description: CONTINUE / PAUSE / ROLLBACK CANDIDATE / INCONCLUSIVE language, decision statement template, and common framing mistakes for evidence analysis stage.
---

# Decision Framing Guide

## TOC

- [The Four Categories](#the-four-categories)
- [Common Framing Mistakes](#common-framing-mistakes)
- [Decision Statement Template](#decision-statement-template)

## The Four Categories

These are **action categories**, not statistical verdicts. They exist to produce a clear next step, not a scientific conclusion.

---

### CONTINUE

**Meaning:** The evidence supports moving the feature flag toward the treatment/candidate variant. In ordinary product language: ship the winning version, either by setting treatment to 100% or by expanding in monitored steps if rollout constraints remain.

**Conditions (read from experiment's `analysisResult`):**
- Primary metric P(win) ≥ 95%
- Primary metric risk[trt] is low (< 0.01 as a general reference — calibrate to your metric's business impact)
- All guardrail P(win) > 20% (no harm signal on any guardrail)
- Observation window is complete (≥ one full business cycle)
- SRM check passed

**What to say:**
> "Move the feature flag toward treatment. The candidate improved [metric name] from [control rate] to [treatment rate], with P(win) = [X]% and risk[trt] = [value]. Guardrails are healthy. If no rollout constraints remain, set treatment to 100%; otherwise expand gradually, for example 50% -> 80% -> 100%, while watching guardrails."

---

### PAUSE

**Meaning:** Something needs investigation before the rollout expands. Not necessarily harmful — signal is mixed or incomplete. In ordinary product language: keep the current flag split and do not send more users to treatment yet.

**Conditions (read from experiment's `analysisResult`):**
- Primary metric P(win) is 80–95% (leaning positive but not conclusive)
- Or a guardrail P(win) ≤ 20% (possible harm — not yet confirmed)
- Or risk[trt] is above 0.01 despite high P(win) (the downside of being wrong is meaningful)
- Or SRM check failed (χ² p < 0.01) — traffic split is compromised, investigate before expanding
- Or an instrumentation anomaly was detected

**What to say:**
> "Hold the current rollout. Do not increase treatment exposure yet because [specific metric/guardrail/SRM issue] is not clean. Keep the flag at the current split while investigating [specific signal]."

---

### ROLLBACK CANDIDATE

**Meaning:** Evidence indicates the candidate variant is causing harm. In ordinary product language: stop sending users to the candidate and return them to the safe/default experience.

**Conditions (read from experiment's `analysisResult`):**
- A guardrail P(win) ≤ 5% (strong harm signal on a protected metric)
- Or primary metric P(win) ≤ 5% (treatment is very likely worse)
- Or critical errors or regressions are directly attributable to the candidate variant

**What to say:**
> "Rollback the candidate. Route users back to control/default, or disable the candidate flag path, because [primary metric or guardrail] shows harm: [concrete number]. Investigate [root cause area] before exposing treatment again."

Do NOT soften ROLLBACK CANDIDATE language. Clarity is operational here.

---

### INCONCLUSIVE

**Meaning:** The collected evidence is genuinely insufficient to support a directional decision. In ordinary product language: do not change rollout based on this run yet.

**Conditions (read from experiment's `analysisResult`):**
- Sample per variant is below `minimumSample` in the experiment record
- Or risk[trt] and risk[ctrl] are both still high (> 0.02) — posterior has not yet narrowed enough
- Or primary metric P(win) is 20–80% after a full observation window has elapsed
- Or external contamination (holiday, marketing event, outage) compromised the window

**What to say:**
> "Keep observing before changing rollout. Current evidence is insufficient: [N] exposures per variant over [X days], P(win) = [X]%, and [sample/risk/SRM/instrumentation issue]. Extend the window, collect enough sample, or fix measurement before deciding."

---

## Common Framing Mistakes

**Calling INCONCLUSIVE when you mean CONTINUE**  
If the primary metric is positive but you're uncertain, that's CONTINUE with a confidence note — not INCONCLUSIVE.

**Calling PAUSE when you mean ROLLBACK CANDIDATE**  
Do not soften "evidence of harm" into "let's investigate." If a pre-defined rollback threshold was crossed, the category is ROLLBACK CANDIDATE regardless of discomfort with the decision.

**Citing P(win) alone without risk**
P(win) = 94% sounds convincing, but if risk[trt] is still high (> 0.01), the downside of being wrong is meaningful. Always pair P(win) with risk[trt] when framing a CONTINUE recommendation. "P(win) = 96%, risk[trt] = 0.003 across 4,200 exposures" is an actionable statement. "P(win) = 94%" alone is not.

**Describing magnitude in statistical terms instead of business terms**
"P(win) = 97%" tells reviewers nothing about the actual change. "Click rate increased from 5.1% to 6.3% (+24% relative) with P(win) = 97%" does.

**Waiting for certainty before framing**  
These categories support decisions under uncertainty. INCONCLUSIVE is a valid and honest frame — use it rather than delaying.

---

## Decision Statement Template

```
Experiment:         [flag key / experiment name]
Observation window: [start date] to [end date]
Sample:             [N users per variant] — minimum required: [minimum_sample_per_variant]
SRM check:          [✓ ok / ⚠ failed — p = X]

Hypothesis: [from project state]

Primary metric: [metric name]
  Baseline (control):  [rate or mean]
  Candidate:           [rate or mean]
  Relative change:     [rel Δ from analysisResult]
  P(win):              [X]%
  risk[trt]:           [value]
  risk[ctrl]:          [value]
  95% credible CI:     [lower, upper]

Guardrails:
  [guardrail 1]: P(win) = [X]%  — [healthy / possible harm / strong harm]
  [guardrail 2]: P(win) = [X]%  — [healthy / possible harm / strong harm]

Decision: [CONTINUE / PAUSE / ROLLBACK CANDIDATE / INCONCLUSIVE]

Reasoning: [2–3 sentences tying the evidence to the hypothesis and the decision category]

Plain-language action: [what to do to the feature flag now: move treatment to 100%, expand 50% -> 80% -> 100%, hold current split, rollback to control/default, or keep observing]
Next action: [specific action: ask for approval, then update rollout through `featbit_release_decision_update_feature_flag_targeting` with `confirmedByUser: true` and enable/disable through `featbit_release_decision_toggle_feature_flag` with `confirmedByUser: true` when available; otherwise investigate guardrail, fix instrumentation, or extend window]
```
