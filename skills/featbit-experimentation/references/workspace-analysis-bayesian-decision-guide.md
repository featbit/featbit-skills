---
name: Bayesian A/B Analysis — Decision Guide
description: Decision table (P(win) + risk), guardrail interpretation, observation window rules, SRM investigation steps, sequential testing guidance, and family-wise error handling.
---


## Decision Guide

Use P(win) and risk together to frame the decision for the evidence analysis stage:

| P(win) | risk[trt] | Interpretation |
|--------|-----------|----------------|
| ≥ 95% | low | Strong signal — adopt treatment |
| 80–95% | low | Leaning treatment — accept or extend window |
| 80–95% | high | Leaning treatment but downside risk is real — extend window |
| 20–80% | — | Inconclusive — extend observation window |
| ≤ 20% | — | Treatment is likely not better — lean control |
| ≤ 5% | — | Treatment appears harmful — consider stopping |

P(win) alone is the primary signal. `risk[trt]` tells you how costly the wrong choice is if P(win) is near a boundary.

### Multiple metrics and the guardrail intent

The script runs independent Bayesian analyses on each metric. There is no multiple-comparison correction across metrics. This is intentional, but it has an implication you need to understand:

**Guardrail metrics are for detecting harm, not for confirming lift.**

If you have 4 guardrail metrics, the probability that at least one of them shows a spuriously high P(win) just by chance is substantially higher than 20%. Do not celebrate a guardrail P(win) of 80% — it does not mean treatment improved that guardrail.

The correct reading of guardrail results:

| Guardrail P(win) | How to read it |
|-----------------|----------------|
| ≥ 80% | Likely a true lift on this guardrail — treat as a bonus, not a decision signal |
| 20%–80% | No detectable effect — guardrail is safe |
| ≤ 20% | Possible harm signal — investigate before shipping |
| ≤ 5% | Strong harm signal — do not ship until root cause is understood |

In short: use guardrails asymmetrically. A low P(win) on a guardrail is actionable. A high P(win) on a guardrail is noise unless it's your primary metric.

### Observation window

**Do not stop an experiment early because P(win) looks high.**

P(win) fluctuates during the observation window. An early peak at 95% can easily revert to 60% as more data arrives — especially in the first few days when user behaviour is atypical.

Minimum observation window rules:
- Run for **at least one full business cycle** — typically 7 days to capture weekly patterns (weekday vs weekend behaviour often differs significantly)
- For features that affect infrequent actions (e.g. checkout, onboarding), run until those actions have had a realistic chance to occur for most exposed users
- For high-traffic surfaces, even a short window accumulates sample fast — but still complete the full cycle for behaviour validity

**Novelty effect:** users often interact differently with a new UI element in the first 1–3 days simply because it is new. If your observation window is shorter than a week, early conversion lifts may not persist. When in doubt, extend the window to see if the signal stabilises.

A good signal is one that **holds steady** as more data arrives, not one that spikes and fades.

### SRM (Sample Ratio Mismatch)

A χ² p-value < 0.01 is a red flag: the traffic split is not what you configured. **Do not draw conclusions from the metric results until the root cause is fixed.**

**When SRM passes (p ≥ 0.01):** traffic allocation looks normal based on the `n` values you provided. This check is only as reliable as your data. Ensure `n` is the **unique user exposure count** (DISTINCT users assigned to each variant), not the number of flag evaluation events — the same user can trigger multiple evaluations, which inflates counts and can mask or create a false SRM signal.

Common causes to investigate in order:

1. **Redirect or page load asymmetry** — if one variant triggers a redirect that the other does not, users may drop out before being counted, creating an imbalance
2. **SDK initialisation timing** — flag evaluation happens before the SDK finishes loading for some users; they get a variant but are not logged as exposed
3. **Bot or crawler traffic** — automated traffic may be bucketed but behaves differently from real users; check if `n` values contain non-human spikes
4. **Assignment cache inconsistency** — a user is assigned to treatment on visit 1 but control on visit 2 (e.g. cookie cleared, logged out); double-counting inflates one variant
5. **Deployment timing** — treatment was rolled out gradually rather than all at once; the observation window start does not match when 100% of treatment users were active

Fix the root cause, reset the observation window, and collect fresh data. Do not adjust the data manually to compensate for SRM.

---

## Where This Fits in the Release Decision Flow

This doc sits between data collection and the final decision:

```
measurement stage           ← defines the metric and instrumentation
    ↓
experiment workspace stage         ← creates the experiment + run records
    ↓
POST /analyze (bayesian_ab)  ← YOU ARE HERE: writes inputData + analysisResult
    ↓
evidence analysis stage            ← reads analysisResult, frames CONTINUE / PAUSE / ROLLBACK
    ↓
learning capture stage             ← records what was learned
```

The agent triggers analysis via `featbit_release_decision_analyze_run`. After `analysisResult` is written to the run record, the agent hands off to the evidence analysis stage so the decision can be tied back to the original hypothesis.

---

## Re-running After New Data

The `/analyze` endpoint is idempotent — re-hit it with `"forceFresh": true` whenever you want fresh numbers:

Call `featbit_release_decision_analyze_run` with `experimentId`, `runId`, and `forceFresh: true`.

`inputData` and `analysisResult` on the run record are both refreshed.

---

## On Sequential Testing (Peeking)

This implementation does not include frequentist Sequential Testing (e.g. always-valid confidence sequences). This is intentional.

**Why Bayesian analysis does not need it:**

Bayesian posteriors have a property called *posterior coherence* — the posterior is a complete, valid description of current beliefs at any sample size. Unlike p-values, P(win) does not rely on a "look only once" assumption and does not inflate false positive rates when checked repeatedly.

**What we use instead:**

| Safeguard | How it helps |
|-----------|-------------|
| `minimum_sample_per_variant` | Prevents running analysis on noisy small-sample posteriors |
| `risk[trt]` alongside P(win) | Harder to trigger spuriously — requires both correct direction and acceptable expected loss |

**Recommended discipline:**

- Fix the experiment horizon upfront; do not stop early because results look promising
- If you must look mid-experiment, raise the stopping threshold (e.g. P(win) ≥ 98%)
- Never act on P(win) alone — use `risk[trt]` as a second signal

**Why not implement Bayesian sequential methods (Bayes Factors, ROPE+HDI)?**

These methods are mathematically rigorous but add interpretation complexity that outweighs their benefit for typical product experimentation. The fixed-horizon approach with `minimum_sample_per_variant` covers the practical need.

> *Experimentation for Engineers*, Chapter 8: fixed-horizon testing — decide sample size upfront, look exactly once — is the simplest and most reliable safeguard against peeking.

---

## On Family-wise Error (Multiple Comparisons)

When an experiment tests multiple metrics simultaneously, checking each at a 95% threshold means the **overall** false positive rate is higher than 5%.

```
P(at least one false positive) = 1 - (1 - 0.05)^M
M=5  → 22.6%    M=10 → 40.1%
```

> *Experimentation for Engineers*, Chapter 8: the book identifies this as a significant source of false positives in multi-metric experiments and recommends Bonferroni correction (`adjusted threshold = 1 - alpha/M`) when testing multiple metrics simultaneously.

**However, the correction applies differently by metric type:**

| Metric type | Recommendation |
|-------------|---------------|
| Single primary optimizing metric | No correction needed — one metric, one test |
| Guardrail metrics | Do **not** apply Bonferroni — raising the threshold makes it harder to detect real harm. Keep guardrails sensitive. |
| Multiple treatment arms (A/B/C/n) | Raise threshold: `1 - (0.05 / M)` where M = number of arms |
| Multiple primary optimizing metrics | Redesign — split into separate experiments, one question per experiment |

**Threshold guidance for multi-arm experiments:**

| Arms | Suggested P(win) threshold |
|------|--------------------------|
| 2    | 97.5% |
| 3    | 98.3% |
| 5    | 99.0% |

**Why we do not implement automatic correction:**

Bonferroni and Benjamini-Hochberg corrections are designed for p-values. P(win) is a posterior probability with different statistical properties. For the standard setup of 1 primary metric + N guardrail metrics, no correction is needed. For multi-arm experiments, the threshold adjustment is simple enough to apply manually and benefits from the user's judgment about risk tolerance.

> *Experimentation for Engineers*, Chapter 8: "define exactly one primary optimizing metric and treat everything else as guardrails. One experiment should answer one question."
