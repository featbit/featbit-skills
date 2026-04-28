---
name: featbit-experimentation
description: Expert guidance for instrumenting FeatBit experiments and A/B tests — recording flag-evaluation exposures and metric events so the experimentation engine can analyze variant performance. Use when user asks about "experimentation", "A/B test", "AB test", "AB testing", "split test", "multivariate test", "variant test", "control vs treatment", "experiment instrumentation", "flag exposure", "track-service", "track event", "metric event", "conversion tracking", "experiment analysis", "sendToExperiment", or wires variant traffic into a FeatBit experiment. Pairs with featbit-sdks-* (flag evaluation) and featbit-evaluation-insights-api (custom-platform SDKs). Do not use for general flag-management API operations — see featbit-rest-api.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: api-reference
---

# FeatBit Experimentation & A/B Testing

Wire flag exposures and metric events into FeatBit's experimentation engine so a hypothesis becomes a measurable result.

## When to Use This Skill

Activate when users:
- Run an A/B test, multivariate test, or split test on a FeatBit feature flag
- Need to record which variant a user saw (`flag_evaluation` events)
- Need to record what users did afterwards (metric events: conversion, revenue, latency)
- Pair `boolVariation()` (or another SDK call) with experiment instrumentation
- Implement the same wrapping helper across multiple services / languages
- Troubleshoot attribution failures (variant counts present, metric counts dropped)
- Decide whether experiment data should land in FeatBit's track-service or their own warehouse

## How an Experiment Works

Four steps. You only write code for two of them.

```
  ┌──────────────────────┐
  │      your app        │
  │                      │
  │  ② flag evaluated ───┼──►  POST /api/track/event
  │                      │     { user, variations }    ┐
  │                      │                              ├─►  experiment data pool
  │  ③ user converted ───┼──►  POST /api/track/event   │           │
  │                      │     { user, metrics }       ┘           │
  └──────────────────────┘                                          ▼
                                              ④ analysis engine ─►  result
```

1. **Hypothesis.** Open an experiment in the UI. Nothing to instrument.
2. **Record exposure.** Whenever a flag evaluation routes a user to a variant, fire one event. *(your code)*
3. **Record outcome.** Whenever the metric of interest happens — checkout, page load, purchase — fire one event. *(your code)*
4. **Analysis.** Track-service buffers events into ClickHouse; stats-service computes results. *(automatic)*

Two contracts bind ② and ③ together:
- **Same `user.keyId`** on the exposure and the metric event — that's the join key.
- **Metric `timestamp` ≥ exposure `timestamp`** — earlier metric events are dropped from attribution.

## Two Event Shapes, One Endpoint

`POST /api/track/event` accepts both. Different body, same URL.

| Event | Top-level field | Fires |
|---|---|---|
| Flag exposure | `variations[]` | once per evaluation site (after `boolVariation()` etc.) |
| Metric (binary conversion) | `metrics[]` without `numericValue` | once when the goal happens (e.g. checkout completed) |
| Metric (continuous value) | `metrics[]` with `numericValue` | per occurrence (revenue per purchase, ms per page load) |

Wire format, per-language SDK wrappers, and timestamp / queue / flush semantics live in `references/tracking-api-and-sdks.md`.

## Two Rules When Calling From Code

1. **Wrap the track API once.** A project-internal helper (`trackFlagForExpt(...)`) keeps URL, env-secret, and transport in one place. Every call site becomes one line. Swap to batch / fire-and-forget later without touching business code.
2. **Fire it immediately after the SDK evaluation.** Exposure is the moment the variant decides behavior, not the moment the UI renders. Same code path, same `user.keyId` the SDK evaluated against.

Per-language helper + call-site examples (Node.js, .NET, Java, Go, Python, Browser JS, React) are in `references/tracking-api-and-sdks.md` §3.

## Where Should Events Land?

Choose by where your **analysis** runs, not by where your flag service runs.

| You have… | You want… | Record to |
|---|---|---|
| FeatBit flags only | Variant distribution, flag health (no business metrics) | FeatBit flag-evaluation insights — zero instrumentation |
| FeatBit flags + managed analysis | Full experiment analysis without standing up a warehouse | FeatBit track-service (this skill's main path) |
| Existing data warehouse | Experiment events alongside other product data | Your own warehouse — same two-rule pattern, your endpoint |

Full trade-offs and pointers in `references/tracking-api-and-sdks.md` §4.

## Reference Files

| File | Read when |
|---|---|
| `references/tracking-api-and-sdks.md` | User asks about wire format, exact request bodies, SDK helper code, timestamp rules, batching / queueing behavior, attribution failures, or chooses between track-service and a self-hosted warehouse |

> Additional references covering experiment design, integration with the
> `featbit-release-decision` skill, and deployment will be added as the skill
> grows. The single reference above is authoritative for instrumentation.

## Related Skills

- `featbit-sdks-*` — language SDKs that produce the variant before tracking
- `featbit-evaluation-insights-api` — direct HTTP for platforms without an SDK
- `featbit-release-decision` (planned) — promote a winning variant to full rollout
