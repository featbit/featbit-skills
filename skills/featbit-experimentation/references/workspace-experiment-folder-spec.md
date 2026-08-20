---
name: Experiment Data Spec
description: API schema reference for release-decision experiments, run traffic fields, inputData, and analysisResult JSON.
---

# Experiment Data Spec

Every experiment and run is stored in FeatBit API persistence. The MCP tools read and write that API state directly; do not create local experiment files or sync scripts.

---

## Experiment And Run Fields

The experiment record owns the durable goal, intent, hypothesis, metrics, and learning. Each run owns one observation window and one analysis configuration.

| Field | Scope | Purpose |
|---|---|---|
| `id` | experiment | Experiment identifier passed to MCP tools |
| `stage` | experiment | Lifecycle stage: `intent`, `hypothesis`, `implementing`, `measuring`, `learning` |
| `goal` / `intent` / `hypothesis` | experiment | Product outcome and falsifiable causal claim |
| `primaryMetricEvent` | experiment/run | Event name for the primary metric |
| `guardrailEvents` | experiment/run | JSON/CSV list of guardrail event names |
| `slug` | run | Stable run label, usually tied to the flag key and window |
| `status` | run | `draft`, `collecting`, `analyzing`, `decided`, `archived` |
| `observationStart` / `observationEnd` | run | Time window included in analysis |
| `method` | run | `bayesian_ab` or `bandit` |
| `controlVariant` | run | Actual FeatBit variation value treated as control |
| `treatmentVariant` | run | Actual FeatBit variation value(s) treated as treatment |
| `assignmentUnitSelector` | run traffic | Field used only for optional layer eligibility hashing, normally `user.keyId` |
| `layerKey` | run traffic | Optional mutual-exclusion layer key. Empty means no layer eligibility gate |
| `layerTrafficPercent` | run traffic | Percentage of assignment units eligible for this run within the layer |
| `analysisSamplingPlan` | run traffic | JSON array defining per-served-variation include rates |
| `audienceFilters` | run traffic | Optional JSON array of user-property filters applied before analysis |
| `minimumSample` | run | Validity floor per analyzed variant |
| `priorProper` / `priorMean` / `priorStddev` | run | Bayesian prior configuration |
| `inputData` | run | Aggregated analysis input written by `featbit_experiment_analyze_run` |
| `analysisResult` | run | Computed analysis output written by `featbit_experiment_analyze_run` |

Rules:

- Read real variation values with `featbit_experiment_get_feature_flag` before configuring control/treatment roles.
- Do not change `controlVariant`, `treatmentVariant`, or traffic sampling on a collecting/analyzing/decided run unless the user explicitly approves the evidence change.
- `observationStart` must not be earlier than the time exposure really began.
- `minimumSample` is a validity floor, not a stopping rule.
- `audienceFilters` and run traffic settings affect analysis queries only. They do not mutate the live feature flag.

---

## Run Traffic Model

The analysis pipeline has three separate concepts. Keep them separate.

| Concept | Source of truth | What it decides |
|---|---|---|
| Feature flag targeting and split | FeatBit feature flag | Which variation the user actually receives |
| Layer eligibility | Run traffic config | Whether an exposure is eligible to enter this run |
| Analysis sampling | Run traffic config | What fraction of each actual served variation is analyzed |

The server never reassigns variants during analysis. It reads evaluation events and uses the variation that FeatBit actually served.

### Layer Eligibility

Layer eligibility is optional mutual-exclusion gating. It is useful when multiple concurrent runs share a surface and should not analyze the same assignment units.

Fields:

```json
{
  "layerKey": "homepage",
  "assignmentUnitSelector": "user.keyId",
  "layerTrafficPercent": 30
}
```

Semantics:

- `layerKey` names the layer. Empty/null disables the layer gate.
- `assignmentUnitSelector` selects the assignment unit from each evaluation event. Use `user.keyId` unless the event payload reliably contains the custom property needed for the layer.
- `layerTrafficPercent` is the eligible percentage of assignment units for this run, from `0` to `100`.
- The layer gate does not decide control/treatment. It only says whether this exposure can be analyzed by this run.
- If a custom selector is missing from an event, that event cannot be assigned to the layer and is excluded from layer-gated analysis. Prefer `user.keyId` unless custom properties are known to be present in event data.

### Analysis Sampling

Analysis sampling runs after layer/audience filtering and after reading the actual served variation. It samples inside each served variation.

Use `featbit_experiment_update_run_traffic`:

```json
{
  "method": "bayesian_ab",
  "controlVariant": "control_current",
  "treatmentVariant": "treatment_dotnet",
  "assignmentUnitSelector": "user.keyId",
  "layerKey": null,
  "layerTrafficPercent": 100,
  "analysisSamplingPlan": "[{\"variation\":\"control_current\",\"role\":\"control\",\"includeRate\":100},{\"variation\":\"treatment_dotnet\",\"role\":\"treatment\",\"includeRate\":100}]"
}
```

Fields inside `analysisSamplingPlan`:

| Field | Meaning |
|---|---|
| `variation` | Actual FeatBit variation value |
| `role` | `control`, `treatment`, `holdout`, or `exclude` |
| `includeRate` | Percentage of users in that actual served variation to include, from `0` to `100` |

Examples:

| Live flag split | Desired analysis sample | Sampling plan |
|---|---|---|
| 50/50 control/treatment | Analyze all traffic | control `100`, treatment `100` |
| 90/10 control/treatment | Analyze 10% total control + all treatment | control `11.111111`, treatment `100` |
| 80/20 control/treatment | Analyze 20% total control + all treatment | control `25`, treatment `100` |
| 80/10/10 control/t1/t2 | Analyze 10% total per arm | control `12.5`, t1 `100`, t2 `100` |
| Bandit 70/10/10/10 | Analyze actual served traffic | every arm `100` |

These are examples, not defaults. Compute `includeRate` from the observed served-variation distribution in the run window. If a flag was changed from 90/10 to 80/20 after data was already collected, old 90/10 exposure events still analyze as 90/10 until a new window or fresh data is used.

---

## Duplicate Events Are Expected

Within an observation window:

- The same user can produce multiple evaluation events for the same flag.
- The same user can produce multiple metric events for the same metric.

The analysis service handles this by:

1. Reading evaluation events within the run window.
2. Applying audience filters and optional layer eligibility.
3. Applying the sampling plan inside each actual served variation.
4. Selecting the first valid assignment per assignment unit and variation role.
5. Aggregating metric events that occur after that assignment.

Metric aggregation still follows the metric definition:

| Aggregation | Meaning |
|---|---|
| `once` | User counts as converted if the event happens at least once after assignment |
| `count` | Count all matching events after assignment |
| `sum` | Sum numeric values after assignment |
| `average` | Average numeric values after assignment |

---

## `inputData` Format

`inputData` is written by `featbit_experiment_analyze_run` after querying live stats.

```json
{
  "metrics": {
    "cta_clicked": {
      "control_current": {"n": 773, "k": 39},
      "treatment_dotnet": {"n": 770, "k": 52}
    },
    "client_error": {
      "control_current": {"n": 773, "k": 3},
      "treatment_dotnet": {"n": 770, "k": 4}
    }
  }
}
```

Rules:

- Outer keys match `primaryMetricEvent` and `guardrailEvents`.
- Inner keys match actual FeatBit variation values configured as control/treatment.
- `n` is the analyzed user/assignment-unit count for that variation.
- `k` is the aggregated metric result for that variation.
- Do not edit `inputData` by hand when live stats are available; rerun analysis.

---

## `analysisResult` Format

`analysisResult` is written by the API analysis flow. Example:

```json
{
  "type": "bayesian",
  "experiment": "homepage-h1",
  "computed_at": "2026-06-27T09:00:00Z",
  "window": { "start": "2026-06-25T00:00:00Z", "end": "2026-06-27T09:00:00Z" },
  "control": "control_current",
  "treatments": ["treatment_dotnet"],
  "srm": {
    "chi2_p_value": 0.48,
    "ok": true,
    "observed": { "control_current": 773, "treatment_dotnet": 770 }
  },
  "primary_metric": {
    "event": "cta_clicked",
    "rows": [
      { "variant": "control_current", "n": 773, "conversions": 39, "rate": 0.0505, "is_control": true },
      { "variant": "treatment_dotnet", "n": 770, "conversions": 52, "rate": 0.0675, "rel_delta": 0.337, "p_win": 0.94, "is_control": false }
    ]
  }
}
```

Do not edit `analysisResult` by hand. Re-run `featbit_experiment_analyze_run` when data or run configuration changes.
