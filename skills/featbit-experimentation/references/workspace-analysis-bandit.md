---
name: Bandit Analysis Reference
description: Thompson Sampling algorithm, analysisResult fields, FeatBit API weight integration, stopping condition, and post-bandit final analysis.
---

# Bandit Analysis Reference

Thompson Sampling multi-armed bandit for dynamic traffic allocation.

Unlike Bayesian A/B testing (fixed 50/50 split, one-shot analysis), a bandit experiment is a **continuous feedback loop**: traffic weights are adjusted repeatedly as data accumulates, shifting more traffic toward the arm that is performing better.

---

## When to Use Bandit vs. A/B

| Goal | Run's `method` field |
|------|----------------------|
| "Did this feature improve the metric? By how much?" | `bayesian_ab` |
| "Which variant should get more traffic right now to maximize reward during the experiment?" | `bandit` |

Bandit is appropriate when:
- Minimizing regret during the experiment matters (e.g. revenue-critical features)
- You are comparing multiple variants (3+) and want to converge faster
- The experiment will run for a long time and you want to avoid "wasting" traffic

Bandit is **not** appropriate when:
- You need a precise δ (delta) estimate with a valid confidence interval — dynamic allocation biases the estimate
- The experiment is short (< a few days) — burn-in leaves too little time for reweighting
- You need a clean causal estimate (e.g. for pricing, regulatory reporting)

> **Book reference** — *Experimentation for Engineers*, Chapter 3: the book draws a clear distinction between A/B testing ("what is the effect?") and bandits ("which arm should get traffic now?"). It frames bandit as the right tool when cumulative reward during the experiment matters more than a precise post-hoc effect estimate.

---

## How It Works

Every time the web `/analyze` endpoint runs a bandit:

1. Builds a Gaussian posterior for each arm from current data (same CLT approximation as the Bayesian A/B path)
2. Draws 10,000 samples from a multivariate normal across all arm posteriors
3. Computes `best_arm_probabilities`: how often each arm drew the highest value
4. Applies **Top-Two strategy**: only the top two arms compete for majority traffic; all others hold the minimum floor
5. Enforces a **1% minimum floor** on every arm — no arm is ever fully cut off

> **Book reference** — *Experimentation for Engineers*, Chapter 3: the book warns against allocating zero traffic to any arm. Early data is noisy; an arm that looks bad after 50 samples may be the true winner. The minimum floor ensures you can always recover from early misreadings. The Top-Two strategy is the book's recommended refinement for reducing regret while keeping exploration focused on real contenders.
6. Writes recommended weights to the run's `analysisResult` in the database

**Burn-in guard**: if any arm has fewer than 100 users, dynamic weighting does not activate. The response reports the shortfall and no weights are written. This prevents early noise from corrupting allocation decisions.

> **Book reference** — *Experimentation for Engineers*, Chapter 3: the book notes that the bootstrap posterior (and by extension the Gaussian CLT approximation) converges reliably only above ~100 samples per arm. Acting on weights computed from fewer samples introduces harmful early skew that can be hard to recover from.

---

## Triggering Bandit Analysis

The web app's analyze endpoint picks `bandit` automatically from the run's `method` field — no separate script invocation:

Call `featbit_experiment_analyze_run` with `experimentId`, `runId`, and `forceFresh: true`.

Reads:
- Run record from the database (variant names, metric events, `method: "bandit"`)
- Per-variant counts from `track-service`

Writes:
- `inputData` and `analysisResult` back to the run record

`inputData` format is identical across methods — proportion `{"n", "k"}` or continuous `{"n", "sum", "sum_squares"}` per variant.

---

## Output: `analysisResult` (bandit type)

```json
{
  "type":          "bandit",
  "algorithm":     "thompson_sampling_top_two",
  "experiment":    "my-feature-v2",
  "computed_at":   "2026-04-01T08:00:00Z",
  "metric":        "signup_click",
  "srm_p_value":   0.42,
  "enough_units":  true,
  "update_message": "successfully updated",
  "best_arm_probabilities": {
    "control":   0.12,
    "treatment": 0.88
  },
  "bandit_weights": {
    "control":   0.10,
    "treatment": 0.90
  },
  "seed": 482901
}
```

| Field | Meaning |
|-------|---------|
| `enough_units` | `false` during burn-in — do not apply weights yet |
| `update_message` | Human-readable status; explains why weights are null during burn-in |
| `best_arm_probabilities` | P(this arm is best) for each arm — the primary signal |
| `bandit_weights` | Recommended traffic fraction per arm after Top-Two + floor — use this to update FeatBit |
| `srm_p_value` | SRM check on current data; if < 0.01, investigate before applying weights |
| `seed` | Random seed used for reproducibility |

---

## Applying Weights to FeatBit

After reading the `analysisResult`, update the feature flag's variant rollout through the configured FeatBit MCP tools when they are available.

FeatBit uses cumulative range format for rollout: `[start, end]` where end − start = weight.

**Converting `bandit_weights` to FeatBit rollout ranges:**

```typescript
// Example: {"control": 0.10, "treatment": 0.90}
// → control:   rollout [0.00, 0.10]
// → treatment: rollout [0.10, 1.00]

let cumulative = 0.0;
for (const [arm, weight] of Object.entries(banditWeights)) {
  const rollout = [cumulative, cumulative + weight];
  cumulative += weight;
}
```

**MCP flow:**
```python
flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
targeting = flag.targeting
targeting.fallthrough.variations = rollout_variations_from_bandit_weights
require_explicit_user_approval("apply bandit traffic weights", targeting)
MCP("featbit_experiment_update_feature_flag_targeting", experimentId=experiment_id, key=flag_key, request={
    "confirmedByUser": True,
    "revision": flag.revision,
    "targeting": targeting,
    "comment": "Apply bandit traffic weights"
})
if not flag.isEnabled:
    require_explicit_user_approval("enable feature flag while applying bandit weights", {"key": flag_key, "isEnabled": True})
    MCP("featbit_experiment_toggle_feature_flag", experimentId=experiment_id, key=flag_key, request={
        "confirmedByUser": True,
        "isEnabled": True,
        "comment": "Enable flag while applying bandit traffic weights"
    })
    flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
    assert flag.isEnabled, "feature flag must be enabled for bandit traffic allocation"
```

Update the `fallthrough.variations[].rollout` field with the computed ranges. Targeting changes alone do not enable the flag; if the bandit should be serving traffic, verify the toggle state after applying weights.

Direct update is the default. Use change-request mode only when reviewer ids are supplied or approval is explicitly required:

```python
MCP("featbit_experiment_update_feature_flag_targeting", experimentId=experiment_id, key=flag_key, request={
    "confirmedByUser": True,
    "revision": flag.revision,
    "targeting": targeting,
    "useChangeRequest": True,
    "reviewers": reviewer_ids,
    "reason": "Apply bandit traffic weights"
})
```

This step requires FeatBit system integration. Full automation (scheduled reweighting without manual intervention) requires a scheduler that calls `featbit_experiment_analyze_run` periodically and applies the returned weights through `featbit_experiment_update_feature_flag_targeting`.

> **Book reference** — *Experimentation for Engineers*, Chapter 3: the book describes the full Thompson Sampling feedback loop as a continuous "sample → estimate posterior → allocate → repeat" cycle. The scheduling interval (how often to reweight) is a tuning parameter: shorter intervals react faster but add noise; longer intervals are more stable but slower to adapt.

---

## Stopping Condition

Stop the bandit experiment when:

```
best_arm_probabilities[arm] >= 0.95
```

At this point:
- Set the winning arm to 100% traffic in FeatBit (or end the experiment)
- The dynamic reweighting loop is complete

> **Book reference** — *Experimentation for Engineers*, Chapter 3: the book uses `pbest(arm) ≥ 0.95` as the explicit stopping threshold for Thompson Sampling — the same value we use here. It is not a hard mathematical requirement but a practical convention: it means you are 95% confident this arm is the best, which is sufficient for most product decisions.

---

## Transition to Final Analysis

After stopping, switch the run's `method` to `bayesian_ab` with `featbit_experiment_update_run` and trigger a final analysis on the full collected dataset:

Call `featbit_experiment_analyze_run` with `forceFresh: true`.

**Important caveat**: bandit experiments produce unequal traffic splits (e.g. 90/10 by the end). This means:
- The δ (delta) estimate in `analysisResult` is valid but has wider uncertainty than a balanced 50/50 design
- `best_arm_probabilities` from the final bandit `analysisResult` is the most reliable decision signal
- Use both together when handing off to the evidence analysis stage

Hand off to the evidence analysis stage with:
- The experiment's `analysisResult` (final Bayesian analysis)
- The bandit `analysisResult` (final best_arm_probabilities)
- The experiment record fields

---

## Recommended Cadence

| Phase | Action | Frequency |
|-------|--------|-----------|
| Burn-in | Collect data, do not reweight | Until every arm ≥ 100 users |
| Exploit | Trigger `featbit_experiment_analyze_run` (run is on `method: bandit`), apply the returned weights with `featbit_experiment_update_feature_flag_targeting`, and verify the flag is enabled with `featbit_experiment_toggle_feature_flag` / readback when needed | Every 6–24 hours |
| Stopping | `best_arm_probabilities >= 0.95` | Check each cycle |
| Wrap-up | Switch run to `method: bayesian_ab`, trigger `featbit_experiment_analyze_run`, hand off to the evidence analysis stage | Once, after stopping |
