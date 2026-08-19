---
name: Bayesian A/B Analysis — Usage Patterns
description: Six patterns for Bayesian A/B analysis — proportion, continuous, inverse, multi-arm, informative prior, and primary+guardrails — with inputData examples and output columns reference.
---


## Usage Patterns

### Pattern 1 — Basic proportion metric (conversion rate, click-through rate)

Input data: `n` users exposed, `k` who converted.

```json
"cta_clicked": {
  "control":   {"n": 5000, "k": 300},
  "treatment": {"n": 5050, "k": 350}
}
```

Trigger analysis through the web app:

Call `featbit_release_decision_analyze_run` with `experimentId`, `runId`, and `forceFresh: true`.

The `analysisResult` returned (and stored on the run record) includes: `n`, `conv`, `rate`, `rel Δ`, `95% credible CI`, `P(win)`, `risk[ctrl]`, `risk[trt]`.

`n` is the analyzed sample after run traffic gates: audience filters, optional layer eligibility, and per-served-variation `analysisSamplingPlan`. It is not automatically forced to equal raw flag traffic. Choose each `includeRate` from the actual exposure distribution inside the run window: desired analyzed users for that variation divided by observed served users for that variation, capped at 100%.

---

### Pattern 2 — Continuous metric (revenue, session duration, score)

Input data: `n` observations, `sum` of values, `sum_squares` of values.

```json
"revenue_per_user": {
  "control":   {"n": 5000, "sum": 45000.0,  "sum_squares": 820000.0},
  "treatment": {"n": 5050, "sum": 48020.0,  "sum_squares": 901000.0}
}
```

The script computes `mean = sum / n` and `variance = (sum_squares − sum²/n) / (n−1)` internally.

Output table includes: `n`, `mean`, `rel Δ`, `95% credible CI`, `P(win)`, `risk[ctrl]`, `risk[trt]`.

---

### Pattern 3 — Inverse metric (lower is better: error rate, latency, p99)

Add `"inverse": true` at the metric level. P(win) is then P(treatment has a *lower* value).

```json
"error_rate": {
  "inverse":   true,
  "control":   {"n": 5000, "k": 90},
  "treatment": {"n": 5050, "k": 70}
}
```

Risk directions are also flipped: `risk[trt]` is the cost of adopting treatment if control's lower value is actually better.

---

### Pattern 4 — Multiple treatment arms (A/B/C test)

Add variant keys in the experiment record and `inputData` matching each arm. The script runs a separate Bayesian comparison (each treatment vs. the single control) for every arm.

```json
"cta_clicked": {
  "control":     {"n": 3300, "k": 198},
  "treatment_a": {"n": 3350, "k": 228},
  "treatment_b": {"n": 3300, "k": 215}
}
```

Each treatment arm gets its own row in the output table and its own decision hint.

**Multiple comparison caution:** running two treatments against the same control means you are making two independent tests. The probability that at least one of them shows P(win) ≥ 95% by chance alone is higher than 5% — roughly `1 − 0.95²  ≈ 10%` for two arms. To maintain the same level of confidence when choosing a winner:

- Raise the P(win) threshold to **≥ 97–98%** before acting on any single arm
- Or use risk as the tiebreaker: prefer the arm with the lowest `risk[trt]`, not the highest P(win)
- Do not pick the "best-looking" arm mid-experiment and treat it as a two-way comparison — that inflates the false positive rate further

The same caution applies to guardrail metrics (documented in the Decision Guide).

---

### Pattern 5 — Informative Gaussian prior

Use when you have historical data from similar experiments and want to regularise noisy early results.

In the experiment record:

```
priorProper:  true
priorMean:    0.0    # no expected direction (use historical lift if available)
priorStddev:  0.3    # ±30% is the plausible lift range for this product area
```

Effect by sample size:

| Sample size | Prior influence |
|---|---|
| Small (n < 200) | Strong — result pulled toward `mean` |
| Medium | Partial shrinkage toward prior |
| Large (n > 1000) | Data dominates — prior washed out |

When to use:
- You have multiple past experiments in this area and know the typical lift range
- You want to reduce false positives on early peeks with small samples

When to keep flat (`proper: false`):
- First experiment in this area — no prior knowledge to encode
- You want results to be purely data-driven

The `analysisResult` output always states which mode was used: `flat/improper (data-only)` or `proper (mean=X, stddev=Y)`.

#### How to derive `mean` and `stddev` from historical experiments

If you have results from a past experiment, read `rel Δ` and the `95% credible CI` from its `analysisResult`:

```
prior mean   = rel Δ  (e.g. +0.12 for a +12% lift)
prior stddev = (ci_upper − ci_lower) / (2 × 1.96)
               (e.g. CI [+4%, +20%] → stddev = (0.20 − 0.04) / 3.92 ≈ 0.041)
```

Note: `se` is not directly shown in `analysisResult`. Derive it from the credible interval width as above.

If you have multiple past experiments, use the average `rel Δ` as `mean` and the standard deviation across those lifts as `stddev`.

#### Two-phase approach: run a pilot first, then start a fresh experiment with a prior

A valid and practical workflow when you have no prior history:

```
Phase A — pilot (days 1–5):
  Run the experiment normally with proper: false (flat prior).
  After phase A, read rel Δ and 95% credible CI from analysisResult.
  Compute: mean = rel Δ,  stddev = (ci_upper − ci_lower) / (2 × 1.96)

Phase B — main experiment (day 6 onward):
  Reset the observation window. inputData must contain only data from day 6 onward.
  Set prior using phase A results:
    mean:   <rel Δ from phase A>
    stddev: <derived from CI above>
    proper: true
```

**Critical rule:** the data from phase A and phase B must never overlap. If you re-use phase A data in phase B's `inputData`, the early data is counted twice and the posterior will be biased. Reset the window completely before collecting phase B data.

---

### Pattern 6 — Primary metric + guardrail metrics

The experiment record supports one primary metric and multiple guardrails. The script runs Bayesian analysis on all of them.

```
primaryMetricEvent:    cta_clicked
guardrailEvents:      ["error_rate", "page_load_time"]
```

In `analysisResult`, the primary metric is in the `primary_metric` object and guardrails are in the `guardrails` array. Decision hints are only generated for the primary metric — guardrails are checked separately for harm signals.

---

## Output Columns Reference

| Column | Description |
|--------|-------------|
| `n` | Users / observations in this variant |
| `conv` | Conversions (proportion metrics only) |
| `rate` | Conversion rate (proportion metrics only) |
| `mean` | Mean value (continuous metrics only) |
| `rel Δ` | Relative change vs control: (trt − ctrl) / ctrl |
| `95% credible CI` | Bayesian 95% credible interval for the relative effect |
| `P(win)` | Posterior probability that this treatment is better than control |
| `risk[ctrl]` | Expected opportunity cost of keeping control (fraction of control mean) |
| `risk[trt]` | Expected loss from adopting treatment (fraction of control mean) |

---
