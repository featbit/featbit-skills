---
name: Holdout Group Reference
description: Post-launch holdout group setup, three effect patterns (holds / decays / improves), and checkpoint cadence for long-term validation.
---

# Holdout Group Reference

A holdout group is a small percentage of users kept permanently on the old version after a feature has been fully launched. It is a **post-launch validation tool**, not a pre-launch decision tool.

> *Experimentation for Engineers*, Chapter 8: the book explicitly recommends maintaining a permanent holdout group after full launch. Short-term experiments cannot capture long-term behavioral changes — novelty effects, seasonal patterns, and event-driven spikes all decay over time. A holdout group running for 30–90 days reveals whether the effect measured during the experiment actually persists.

---

## Why Short-Term Experiments Can Mislead

Many transient factors inflate experiment results during the observation window:

| Effect | What happens |
|--------|-------------|
| **Novelty effect** | Users interact with a new UI element out of curiosity; engagement decays once the novelty wears off |
| **Hot topic / event effect** | Launch coincides with a campaign, viral moment, or seasonal spike; traffic quality is unrepresentative |
| **Seasonal effect** | Holidays or promotions produce behavior that doesn't reflect normal usage |
| **Primacy effect** | Existing users resist change initially; a genuinely better feature looks worse short-term |
| **Hawthorne effect** | Users behave differently when they sense they are being observed |

These effects share one property: **they are temporary**. A holdout group running through multiple cycles reveals whether the measured lift was real or transient.

> *Experimentation for Engineers*, Chapter 8: the book frames these as "confounds that resolve over time" and notes that comparing a permanent holdout group against the fully-launched population is the most reliable way to measure sustained impact.

---

## How It Works

At full launch, adjust the feature flag traffic split to keep a small holdout group on the old version. Use `featbit_release_decision_get_feature_flag` to read the latest flag revision and `featbit_release_decision_update_feature_flag_targeting` to apply the split when FeatBit MCP tools are available, but only after the user approves the exact holdout rollout. Targeting changes do not enable the flag; if launch begins now, ask for approval to enable the flag, call `featbit_release_decision_toggle_feature_flag` with `confirmedByUser: true` and `isEnabled: true`, and read the flag back before recording `launched_at`:

```
Feature flag: new-onboarding-flow
  control (old version):   5%   ← holdout group — keep indefinitely
  treatment (new feature): 95%  ← full launch
```

Then re-trigger analysis at 30, 60, and 90 days via the web `/analyze` endpoint (the run stays on `method: bayesian_ab`).

**Why this cancels external factors:**

Both groups experience identical external conditions — same season, same trending topics, same market. The only difference is the feature version. External effects cancel out; the residual difference reflects the feature's true sustained impact.

---

## Running Holdout Analysis

Create a separate run for each time checkpoint with `featbit_release_decision_create_run` and `featbit_release_decision_update_run`, using a time-stamped slug and the checkpoint's `observationStart` / `observationEnd` dates, then trigger analysis on each:

Call `featbit_release_decision_analyze_run` for each holdout run with `forceFresh: true`.

Each checkpoint needs its own run record. The run is identical to the original except for `observationStart` / `observationEnd` dates.

---

## Interpreting Results Over Time

Look for one of three patterns:

**Pattern 1 — Effect holds (genuine improvement)**
```
Original (day 14):   P(win) = 97%,  rel Δ = +8%
Holdout day 30:      P(win) = 95%,  rel Δ = +7.5%
Holdout day 60:      P(win) = 94%,  rel Δ = +7%
Holdout day 90:      P(win) = 93%,  rel Δ = +6.8%
```
Signal is stable → the experiment result was real.

**Pattern 2 — Effect decays (novelty or event-driven)**
```
Original (day 14):   P(win) = 97%,  rel Δ = +8%
Holdout day 30:      P(win) = 85%,  rel Δ = +4%
Holdout day 60:      P(win) = 62%,  rel Δ = +2%
Holdout day 90:      P(win) = 51%,  rel Δ = +0.5%
```
Signal collapses → original result was mostly transient. Consider reverting or redesigning.

**Pattern 3 — Effect improves over time (primacy effect resolved)**
```
Original (day 14):   P(win) = 72%,  rel Δ = +3%
Holdout day 30:      P(win) = 85%,  rel Δ = +5%
Holdout day 60:      P(win) = 93%,  rel Δ = +7%
Holdout day 90:      P(win) = 96%,  rel Δ = +8%
```
Users initially resisted the change; after adaptation the feature's value becomes clear.

---

## Holdout Plan in Experiment Record

Record the holdout plan in the experiment record (e.g. in a `holdoutPlan` JSON field or as part of the experiment's notes):

```yaml
holdout:
  enabled: true
  percentage: 5
  check_at_days: [30, 60, 90]
  launched_at: 2026-04-01
  # Reminder: collect fresh inputData and re-run analyze-bayesian.ts at each checkpoint.
  # Use slugs: <original-slug>-holdout-30d, -60d, -90d
```

This block is documentation only — it does not affect script behavior. It ensures the holdout plan is visible during the evidence analysis stage and the learning capture stage.

---

## Difference from A/B Testing and Bandit

| | A/B Test | Bandit | Holdout Group |
|--|---------|--------|--------------|
| **Question** | Should we ship? | Which arm gets more traffic now? | Did the effect last? |
| **Timing** | Before launch | During experiment | After full launch |
| **Duration** | Days–weeks | Days–weeks | Months |
| **Traffic** | 50/50 | Dynamic | 95/5 |
| **Method** | `bayesian_ab` | `bandit` | `bayesian_ab` |

Holdout groups sit **after** the A/B or Bandit experiment concludes. They are not a replacement for pre-launch testing — they are long-term insurance against transient effects.

---

## Recommended Cadence

| Checkpoint | Action |
|-----------|--------|
| Launch day | After explicit user approval, set feature flag to 95/5 with `featbit_release_decision_update_feature_flag_targeting` and `confirmedByUser: true` when available; enable with `featbit_release_decision_toggle_feature_flag` and `confirmedByUser: true` if the flag is off; note `launched_at` in experiment record only after readback confirms the intended state |
| Day 30 | Trigger analysis on the holdout-30d run via `featbit_release_decision_analyze_run` with `forceFresh: true` |
| Day 60 | Repeat |
| Day 90 | Repeat; decide whether to fully close the holdout group |
| Any point | If effect has clearly collapsed, consider rollback investigation |
