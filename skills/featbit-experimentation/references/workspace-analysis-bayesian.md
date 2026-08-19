---
name: Bayesian A/B Analysis — Algorithm
description: Bayesian posterior math, metric types, prior configuration, analysisResult output schema, and minimumSample validity floor.
---

# Bayesian Analysis — How It Works and How to Use It

The analysis script reads `inputData` from the experiment's database record and writes `analysisResult` back.
No dashboard required. No online account. Runs locally.

---

## Requirements

```
Python >= 3.10
pip install numpy scipy
```

The analysis is implemented in Python using numpy and scipy for statistical computation.

---

## What the Script Does

1. Reads the experiment record from the DB — variant names, metric events, observation window, optional prior
2. Parses `inputData` — aggregated per-variant counts
3. For each metric and each treatment arm:
   - Computes the **Bayesian posterior** over the relative effect δ = (mean_trt − mean_ctrl) / mean_ctrl
   - Derives **P(win)**, **95% credible interval**, and **risk / expected loss**
4. Runs an **SRM check** — flags trafficking imbalances before you interpret any result
5. Runs two validity checks on the primary metric:
   - `n ≥ minimumSample` — exposure floor from the experiment record
   - `k ≥ 30` per variant — conversion floor for Gaussian approximation reliability; warns if not met even when n passes
6. Writes `analysisResult` to the experiment's database record

---

## Conceptual Overview — What Bayesian A/B Testing Means

If you are new to Bayesian experimentation, here is the mental model the script uses.

### The question being answered

> "Given the data I collected, how probable is it that treatment is better than control?"

This is different from asking "is this result statistically significant?". The Bayesian approach gives you a **direct probability statement** about which variant is better, which is easier to act on.

### What the posterior distribution is

After observing `n` users and `k` conversions, the script constructs a **Gaussian posterior** over the relative effect δ:

```
δ = (mean_treatment − mean_control) / mean_control

posterior ~ N(μ_rel, se²)

where:
  μ_rel  = δ computed from your data (the point estimate)
  se     = standard error from the delta method (how uncertain we are)
```

The wider the posterior (larger `se`), the more uncertain the estimate — typically because sample sizes are small.

### The three outputs and how to read them

| Output | What it means |
|--------|---------------|
| **P(win)** | Posterior probability that treatment is better. 97% means "given what we observed, there is a 97% chance treatment is truly better." |
| **95% credible CI** | The relative lift is very likely somewhere in this range. `[+5%, +22%]` means you can say: "the true lift is probably between 5% and 22%." |
| **risk[ctrl]** | If you *stay on control* and treatment is actually better — your expected opportunity cost (as a fraction of the control mean). Lower is better when you want to hold. |
| **risk[trt]** | If you *adopt treatment* and control is actually better — your expected loss (as a fraction of the control mean). Lower is better when you want to ship. |

Risk is the most actionable output for decisions near the boundary. If `risk[trt] = 0.002`, shipping the wrong variant costs at most 0.2% — the downside is small even if you are wrong.

**The formula behind risk** (so you understand what the number represents):

```
risk_ctrl = E[max(0,  δ)]  =  ∫₀^∞  δ · p(δ) dδ
risk_trt  = E[max(0, -δ)]  =  ∫_{-∞}^0  |δ| · p(δ) dδ
```

Where `p(δ)` is the posterior distribution of the relative effect. In plain language:
- `risk_ctrl` = the average upside you leave on the table if treatment is better and you keep control
- `risk_trt` = the average downside you absorb if control is better and you ship treatment

Both are weighted by how probable each direction is — so a large absolute difference matters less if the posterior says that direction is very unlikely.

**Practical reference ranges for risk[trt]:**

| risk[trt] value | Interpretation |
|----------------|----------------|
| < 0.001 | Downside < 0.1% of control mean — negligible, safe to ship |
| 0.001 – 0.01 | Downside 0.1%–1% — acceptable for most product decisions |
| > 0.01 | Downside > 1% — meaningful; weigh against business context before shipping |

These ranges are heuristics, not hard rules. Calibrate against the business impact of a 1% error in your specific metric.

### Flat prior vs. informative prior

By default the script uses a **flat (improper) prior**: the posterior equals the data likelihood. This is the safest default — no assumptions injected.

If you set `priorProper: true` in the experiment record, the script applies a **conjugate Gaussian prior update**:

```
post_mean = (data_mean/data_var + prior_mean/prior_var) / (1/data_var + 1/prior_var)
post_std  = sqrt(1 / (1/data_var + 1/prior_var))
```

With a small sample the posterior is pulled toward the prior mean. With a large sample the data dominates and the prior is washed out.

**Design note — why the prior is on δ, not on p:**

For proportion metrics, the textbook Bayesian approach puts a Beta prior directly on the conversion rate `p`:

```
p ~ Beta(α, β)  →  posterior: p | data ~ Beta(α+k, β+n-k)  (exact)
```

This script instead puts a Gaussian prior on the *relative effect* δ = (p_b − p_a) / p_a. This is a deliberate engineering trade-off:

- **Advantage:** the same code handles both proportion and continuous metrics with one unified interface
- **Advantage:** analytical (closed-form) solution — no MCMC sampling needed, runs instantly
- **Trade-off:** the Gaussian approximation is less accurate for small samples (n < 100 per variant) or extreme conversion rates (< 2% or > 98%)

With `proper: false` (the default), the flat prior means the posterior equals the likelihood — the script is using Bayesian language to describe what is mathematically equivalent to a likelihood-based estimate. The results are identical to what you would get from a normal approximation to the conversion rate difference. The Bayesian framing becomes meaningful when you use `proper: true` and inject real prior knowledge.

---

## Before You Run: Setting `minimumSample`

`minimumSample` in the experiment record is a **validity floor** — the minimum number of exposed users per variant before the script's Gaussian approximation can be trusted. It is not a stopping rule.

The Bayesian stopping criterion is different and comes later: you stop when `risk[trt]` or `risk[ctrl]` falls below an acceptable threshold (see Decision Guide). The validity floor just ensures the math is reliable enough to read at all.

---

### What the validity floor protects

The script approximates the posterior as a Gaussian distribution. This approximation requires a sufficient number of observed conversions — not just exposures. The rule of thumb:

```
n × p_baseline ≥ 30   (at least 30 conversions per variant)
```

Below this threshold, the Gaussian curve is a poor fit for the true posterior shape, and P(win) and risk values can be misleading even when sample sizes look large.

---

### How to set `minimum_sample_per_variant` — proportion metrics

```
minimum_sample_per_variant  =  30 / p_baseline
```

| Baseline conversion rate | Minimum n per variant |
|--------------------------|-----------------------|
| 1% | 3,000 |
| 2% | 1,500 |
| 5% | 600 |
| 8% | 375 |
| 10% | 300 |
| 20% | 150 |
| 30% | 100 |

**The default of 200 is only safe when your baseline conversion rate is above 15%.** For most product metrics (CTR, signups, checkout rates) which sit between 2–10%, the floor is between 300 and 1,500.

**Example:** CTA click rate baseline is 5%.
```
minimum_sample_per_variant = 30 / 0.05 = 600
```

Set `minimumSample: 600` in the experiment record.

---

### How to set `minimum_sample_per_variant` — continuous metrics

For continuous metrics (revenue, session duration), the Gaussian approximation is generally more robust because the Central Limit Theorem kicks in faster. A floor of 100–200 per variant is usually sufficient unless the metric has extreme skew (e.g. revenue where a few large orders dominate).

If your continuous metric is highly skewed, increase the floor to 500+.

---

### Reading risk values to judge whether to decide

Reaching `minimum_sample_per_variant` only means the results are safe to read. It does not mean you should stop.

After reaching the floor, re-trigger analysis periodically via the web app:

Call `featbit_release_decision_analyze_run` with `experimentId`, `runId`, and `forceFresh: true`.

Then read `analysisResult` from the run record via `featbit_release_decision_get_experiment` and check `risk[trt]` and `risk[ctrl]`. These are outputs computed server-side by the analyze endpoint — there is no configuration field for them. The judgment of "low enough" is made by you or the agent running the evidence analysis stage by comparing the values to the reference ranges in the Decision Guide below.

How risk behaves as sample grows:

```
Small sample   →  wide posterior  →  risk[trt] and risk[ctrl] both high  →  keep running
Large sample   →  narrow posterior →  one side's risk drops               →  ready to decide
```

Risk falls as sample size grows and the posterior narrows. A large true effect causes rapid convergence. A small or absent effect causes risk to plateau — both sides stay uncertain — which is the correct signal that the experiment is genuinely inconclusive.

The judgment step — "is risk[trt] low enough to ship?" — belongs to the evidence analysis stage, not to this script.

---

### What happens if you ignore the validity floor

| Situation | Consequence |
|-----------|-------------|
| n=200, baseline rate 5% → only 10 conversions per variant | Gaussian approximation is inaccurate; P(win) and risk values are unreliable |
| P(win) = 94% with 10 conversions | Likely an artefact of the approximation — do not act on it |
| risk[trt] looks low at n=100 | The posterior is still wide; risk will rise again as more data arrives |

The validity floor is the minimum before you start reading results. Treat any output below the floor as noise, not signal.

---
