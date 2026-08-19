---
name: Data Source Guide
description: Track-service query endpoint, ExperimentQueryRequest and ExperimentQueryResponse shapes, and canonical data flow for experiment inputData.
---

# Data Source Guide

How `inputData` is produced for experiment analysis, and how to verify it.

---

## Input Contract

`inputData` is stored as a JSON string in the experiment run record:

```json
{
  "metrics": {
    "click_start_chat": {
      "false": {"n": 1234, "k": 89},
      "true":  {"n": 1198, "k": 112}
    },
    "error_rate": {
      "false": {"n": 1234, "k": 12},
      "true":  {"n": 1198, "k": 19}
    }
  }
}
```

Keys:
- Outer keys are metric event names — must match `primaryMetricEvent` and `guardrailEvents` in the experiment record
- Inner keys are variant values — must match `controlVariant` and `treatmentVariant` in the experiment record
- `n` = unique users exposed to that variant in the observation window
- `k` = unique users who fired the metric event at least once, out of those `n`

---

## Canonical Data Flow

```
Instrumentation code
  → track-service (receives flag_evaluation + metric events)
      → ClickHouse (stores raw events)
          → FeatBit API analysis via featbit_release_decision_analyze_run
              → POST /api/query/experiment (track-service query endpoint)
                  → assembles inputData + runs analysis
                      → writes inputData + analysisResult to run record
```

Your job is to make sure instrumentation sends evaluation and metric events with the correct `envId`, `flagKey`, user key, variation, event names, timestamps, and any user properties required by `audienceFilters` or a custom `assignmentUnitSelector`. Once events land, `featbit_release_decision_analyze_run` handles the rest — no manual data assembly needed.

---

## Track-Service Query Endpoint

The web app calls track-service internally when `/analyze` runs. You do not call this endpoint directly, but understanding its shape helps debug missing data.

**Request — `POST /api/query/experiment`:**

```typescript
// ExperimentQueryRequest (from TrackPayload.cs conventions)
{
  envId:       string,   // environment ID
  flagKey:     string,   // feature flag key
  metricEvent: string,   // event name to count as conversion
  startDate:   string,   // ISO 8601 — matches run's observationStart
  endDate:     string,   // ISO 8601 — matches run's observationEnd (or now if still open)
  controlVariant?: string,
  treatmentVariants?: string[],
  audienceFilters?: string,
  layerKey?: string,
  assignmentUnitSelector?: string,
  layerTrafficPercent?: number,
  analysisSamplingPlan?: string
}
```

**Response — `ExperimentQueryResponse`:**

```typescript
{
  variants: Array<{
    variant:     string,   // variant value (matches controlVariant / treatmentVariant)
    users:       number,   // unique users assigned to this variant
    conversions: number,   // unique users who fired the metricEvent
    sumValue:    number,   // sum of metric values (for continuous metrics)
    sumSquares:  number    // sum of squared values (for variance calculation)
  }>
}
```

The web app maps this response into `inputData` in the canonical shape before running analysis.

---

## What `track-service` Receives from Instrumentation

Track-service receives evaluation and metric payloads from the SDK/application. The exact wire shape can vary by SDK, but the analysis layer needs the same logical data:

```typescript
// TrackPayload shape (from modules/track-service)
{
  user: {
    keyId: string,              // the unique user identifier
    properties?: object         // optional; required for custom audience/layer selectors
  },
  variations: Array<{
    flagKey:      string,    // feature flag key
    variant:      string,    // which variant the user got
    timestamp:    string     // ISO 8601
  }>,
  metrics: Array<{
    eventName:    string,    // must match primaryMetricEvent or guardrailEvents
    timestamp:    string,    // ISO 8601
    numericValue: number,    // 1 for binary events; actual value for continuous
    type:         string     // "binary" | "continuous"
  }>
}
```

Duplicate events are normal. The same user can evaluate the same flag many times and can fire the same metric many times in one observation window. Analysis uses the first valid included assignment for that user/assignment unit, then aggregates metric events that occur after that assignment according to the metric aggregation mode (`once`, `count`, `sum`, or `average`).

If the run uses a custom `assignmentUnitSelector` or audience filter property, that property must be present on the evaluation/user event data. If it is missing, the event cannot pass that selector-dependent gate and may be excluded from analysis. Prefer `user.keyId` unless the custom property is known to be logged.

The SDK wires `flagKey` and `variant` from the active flag evaluation. Your code must still ensure the metric event name and numeric value match the experiment definition.

---

## Debugging Missing Data

If `/analyze` returns `{ "status": "no_data" }`:

1. **Check flag evaluations are landing** — verify `track-service` receives `variations[]` entries with the correct `flagKey` and `envId`
2. **Check metric events are landing** — verify `metrics[]` entries arrive with the correct `eventName`
3. **Check the observation window** — `startDate` must match when the flag was enabled, not before
4. **Check `envId`** — the environment ID in track-service must match the one in the experiment record
5. **Check run traffic gates** — if `layerKey`, `assignmentUnitSelector`, `layerTrafficPercent`, or `audienceFilters` are set, confirm evaluation events contain the required user key/properties
6. **Check sampling plan** — if include rates are low, confirm enough users remain after sampling

If `/analyze` returns `{ "status": "no_data", "reason": "zero_users" }`:
- Metric events are present but no users have been assigned to variants yet
- Read the flag with `featbit_release_decision_get_feature_flag` and confirm `isEnabled` is true. If collection should be active and the toggle tool is available, ask the user to approve enabling the exact flag, call `featbit_release_decision_toggle_feature_flag` with `confirmedByUser: true` and `isEnabled: true`, then read back before retrying analysis.
- Confirm the FeatBit SDK is calling `variation()` in the live codebase

---

## Verifying Input Data Quality

After triggering analysis, read the `inputData` written back to the run record via `featbit_release_decision_get_experiment` and sanity-check:

- Both variant keys match `controlVariant` and `treatmentVariant` in the experiment record
- `n` values are plausible — not 0, not absurdly high
- `k` ≤ `n` for every row
- All metrics listed in `primaryMetricEvent` and `guardrailEvents` are present
- For sampled gradual-rollout analysis, `n` should reflect the configured per-variation include rates applied to the actual exposure distribution in the run window
- If `n` values differ significantly from the intended analysis sample, check layer eligibility and sampling plan before interpreting results
