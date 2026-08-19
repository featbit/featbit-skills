---
name: Event Schema Design
description: TrackPayload shape from track-service, event naming conventions, metric-to-event mapping, instrumentation checklist, and anti-patterns.
---

# Event Schema Design

How to design events that feed into experiment analysis via the FeatBit track-service.

## TOC

- [Core Event Structure — TrackPayload](#core-event-structure--trackpayload)
- [Event Naming Convention](#event-naming-convention)
- [Metric-to-Event Mapping](#metric-to-event-mapping)
- [Upstream vs. Downstream Placement](#upstream-vs-downstream-placement)
- [Common Anti-Patterns](#common-anti-patterns)
- [Instrumentation Checklist](#instrumentation-checklist)

---

## Core Event Structure — TrackPayload

The FeatBit SDK sends a `TrackPayload` to track-service for each user action. The shape is:

```typescript
// TrackPayload — sent via FeatBit SDK track() calls
{
  user: {
    keyId: string,       // stable user identifier — links events to flag evaluation records
    properties?: object  // optional; required only when audience filters or custom assignment selectors use them
  },
  variations: Array<{    // populated automatically by the SDK from active flag evaluations
    flagKey:      string,
    variant:      string,
    timestamp:    string    // ISO 8601 UTC
  }>,
  metrics: Array<{       // your instrumentation code fills this
    eventName:    string,   // must match primaryMetricEvent or guardrailEvents in the experiment record
    timestamp:    string,   // ISO 8601 UTC
    numericValue: number,   // 1 for binary events; actual measurement for continuous
    type:         string    // "binary" | "continuous"
  }>
}
```

**Your responsibility:** fire `track()` with the correct `eventName` and `numericValue` at the right point in the user journey. The SDK wires `variations[]` automatically from the active flag evaluation — you do not build that part.

---

## Event Naming Convention

Use `verb_noun` in `snake_case`:

| Good | Bad |
|---|---|
| `user_completed_onboarding` | `OnboardingComplete` |
| `flag_first_evaluated` | `featureFlagCheck` |
| `checkout_step_abandoned` | `abandon` |
| `ai_skill_session_started` | `chatOpen` |

Name from the user's perspective, not the system's. "User initiated checkout" is more useful than "checkout button clicked."

The event name you define here must be used verbatim in the experiment record's `primaryMetricEvent` and `guardrailEvents` fields, and in the `eventName` field of each `TrackPayload.metrics` entry.

---

## Metric-to-Event Mapping

| Metric | Event name | `type` | `numericValue` | Fire when |
|---|---|---|---|---|
| Onboarding completion rate | `user_completed_onboarding` | `binary` | `1` | User reaches final confirmation step |
| AI Skill adoption | `ai_skill_session_started` | `binary` | `1` | User sends first message in Chat with AI Skills |
| Checkout conversion | `order_placed` | `binary` | `1` | Payment confirmed |
| Revenue per user | `order_placed` | `continuous` | `order_total_usd` | Payment confirmed |
| Step abandonment | `onboarding_step_abandoned` | `binary` | `1` | User navigates away from an incomplete step |
| Error rate (inverse) | `request_error` | `binary` | `1` | Server or client error occurs during variant flow |

---

## Upstream vs. Downstream Placement

**Upstream event** (fires early in the funnel): Higher volume, more noise, weaker signal about the outcome you actually care about.

**Downstream event** (fires at the outcome): Lower volume, higher precision, directly tests the hypothesis.

Prefer downstream events for the primary metric. Use upstream events as diagnostics only.

To choose placement, draw the user journey:

```
Entry → Step 1 → Step 2 → [Variant branch here] → Step 3 → Outcome
                                                              ↑
                                                   fire primary metric event here
```

The event must fire AFTER the user has experienced the variant, not before.

---

## Common Anti-Patterns

**Measuring system activity instead of user outcomes**  
Counting API calls or page views when what matters is task completion.

**Conflating exposure with outcome**  
"Saw the modal" is not "completed the task." The event must fire at outcome, not at exposure.

**Missing `user.keyId` linkage**  
Events without a stable `keyId` cannot be joined to flag evaluation records. Experiment analysis becomes impossible.

**Custom selector not present in event data**  
If a run uses `assignmentUnitSelector` or an audience filter based on a custom user property, that property must be present in the evaluation/user event data. Otherwise those events are excluded from that selector-dependent analysis gate.

**Server-time vs. client-time timestamps**  
Use server-recorded time for consistency. Client timestamps can drift and skew cohort comparisons.

**Single session proxy for long-term outcome**  
Measuring "added to cart" as a proxy for "purchased" when the conversion window is 3 days. Ensure the event window covers the full conversion cycle.

**Wrong `type` field**  
Sending `type: "binary"` for a continuous metric (e.g. revenue) causes the analysis endpoint to compute the wrong aggregation. Always match `type` to how the metric will be analyzed.

---

## Instrumentation Checklist

Before starting exposure, confirm:

- [ ] `primaryMetricEvent` name is defined and matches the `eventName` in `TrackPayload.metrics`
- [ ] Each guardrail event name is defined in `guardrailEvents` and fires with the correct `eventName`
- [ ] Event fires after the variant is shown, not before
- [ ] `user.keyId` in the payload matches the user key used in flag evaluation
- [ ] Any custom property used by `assignmentUnitSelector` or audience filters is present in event data
- [ ] `type` is set correctly: `"binary"` for conversion events, `"continuous"` for value events
- [ ] `numericValue` is `1` for binary events; the measured value for continuous events
- [ ] Events are firing in staging/test environment before production
- [ ] Event volume is realistic for the expected observation window and `minimumSample`
