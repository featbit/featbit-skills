---
name: featbit-evaluation-insights-api
description: Expert guidance for using FeatBit's Flag Evaluation REST API and Track Insights REST API to build custom SDKs for platforms without an official FeatBit SDK. Use when user asks about "evaluation API", "flag evaluation endpoint", "evaluate feature flags via HTTP", "track insights", "insight tracking API", "build custom SDK", "Kotlin SDK", "Android SDK", "iOS SDK", "Swift SDK", "Unity SDK", "embedded SDK", "mobile feature flags", "sendToExperiment", or needs to call FeatBit evaluation server directly. Do not use for management API operations (projects, environments, flag CRUD) — use featbit-rest-api for those. Do not use when an official SDK exists for the target language.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: api-reference
---

# FeatBit Flag Evaluation & Insights API

Direct HTTP access to FeatBit's evaluation server — the foundation for building custom SDKs on any platform (Kotlin, Swift, Android, iOS, Unity, embedded, etc.).

**Official documentation:**
- Flag Evaluation API: https://docs.featbit.co/api-docs/flag-evaluation-api
  (source: https://github.com/featbit/featbit-docs/blob/main/pages/api-docs/flag-evaluation-api.md)
- Track Insights API: https://docs.featbit.co/api-docs/track-insights-api
  (source: https://github.com/featbit/featbit-docs/blob/main/pages/api-docs/track-insights-api.md)

## Prerequisites

Before writing any code:
1. **Evaluation server URL** — `{evaluation-server-url}` (e.g., `https://eval.your-featbit.com`). Found in your FeatBit dashboard under environment settings.
2. **Environment secret key** — client-side or server-side key. Include in every request header:
   ```
   Authorization: your-environment-secret-key
   ```
   > See [How to get the environment secret](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)

## Execution Procedure

```python
# Recommended pattern for a custom mobile/frontend SDK

# Step 1 — App launch: evaluate ALL feature flags (or filtered by tags/keys)
flags = POST evaluate(user, filter?)
# Step 2 — Store locally: cache results in memory for fast synchronous reads
cache.update(flags)
# Step 3 — Track insights: fire-and-forget, send evaluation records to FeatBit
async POST track(variations=flags_with_sendToExperiment)
# Step 4 — Poll for updates: repeat Step 1 periodically using filter.timestamp
on_interval:
    changed = POST evaluate(user, filter={timestamp: last_poll_ms})
    cache.update(changed)
# Step 5 — Track metrics: send custom events (conversion, click, purchase) as they occur
on_user_action:
    async POST track(metrics=[{eventName, numericValue, ...}])
```

### Step 1: Evaluate Feature Flags

```
POST {evaluation-server-url}/api/public/featureflag/evaluate
Authorization: your-environment-secret-key
Content-Type: application/json
```

Minimal request body:

```json
{
  "user": {
    "keyId": "user-123",
    "name": "John Doe",
    "customizedProperties": [
      { "name": "country", "value": "US" },
      { "name": "plan", "value": "premium" }
    ]
  }
}
```

Response — array of feature flag evaluation results:

```json
[
  {
    "key": "new-checkout-flow",
    "variation": {
      "id": "08aceef3-5513-4b38-80ad-4b27bebe8871",
      "type": "boolean",
      "value": "true",
      "matchReason": "premium user rule",
      "sendToExperiment": true
    }
  }
]
```

Key fields:
- `variation.value` — always a **string**, cast to the needed type (`"true"` → boolean, `"104857600"` → number)
- `variation.sendToExperiment` — **save this**; pass it unchanged when tracking insights
- `variation.matchReason` — `"flag disabled"` | `"targeted"` | `"{rule name}"` | `"default"`

**Narrow the response with filters** (essential for reducing payload on mobile):

```json
{
  "user": { "keyId": "user-123" },
  "filter": {
    "keys": ["checkout-flow", "theme-color"],
    "tags": ["mobile"],
    "tagFilterMode": "and",
    "timestamp": 1704067200000
  }
}
```

> `filter.timestamp` — Unix ms. Returns only feature flags modified **after** this timestamp. Pass the timestamp from the previous call to implement efficient polling.

Read `references/flag-evaluation-api.md` for the complete schema, all filter options, and error responses.

### Step 2: Cache Locally

Store the response array in memory keyed by `flag.key`. Serve flag values synchronously from cache — never block UI on a network call.

### Step 3: Track Feature Flag Evaluation Insights (fire-and-forget)

After evaluating feature flags, send evaluation records asynchronously.

```
POST {evaluation-server-url}/api/public/insight/track
Authorization: your-environment-secret-key
Content-Type: application/json
```

Request body — array of insight objects:

```json
[
  {
    "user": {
      "keyId": "user-123",
      "name": "John Doe",
      "customizedProperties": [
        { "name": "plan", "value": "premium" }
      ]
    },
    "variations": [
      {
        "featureFlagKey": "new-checkout-flow",
        "variation": {
          "id": "08aceef3-5513-4b38-80ad-4b27bebe8871",
          "value": "true"
        },
        "sendToExperiment": true,
        "timestamp": 1704067200000
      }
    ],
    "metrics": []
  }
]
```

> Response: `200 OK` with empty body on success. Do **not** await this in the main UI thread.

**Critical**:
- Copy `variation.id`, `variation.value`, and `sendToExperiment` exactly from the evaluation response. Do **not** derive or guess these values.
- At least one of `variations` or `metrics` must be non-empty in each insight object. A payload where both are empty arrays is a no-op.

### Step 4: Poll for Feature Flag Updates

On a background timer, re-evaluate using `filter.timestamp` set to the time of the last successful poll. Only changed feature flags are returned — update your cache with the diff.

### Step 5: Track Custom Metric Events

When a user performs a meaningful action (purchase, conversion, click):

```json
[
  {
    "user": { "keyId": "user-123" },
    "variations": [],
    "metrics": [
      {
        "route": "/checkout/complete",
        "type": "CustomEvent",
        "eventName": "purchase-completed",
        "numericValue": 99.99,
        "appType": "Mobile",
        "timestamp": 1704067350000
      }
    ]
  }
]
```

> `appType` — use `"Mobile"` for Android/iOS apps, `"Web"` for browser-based apps.

Read `references/track-insights-api.md` for the complete schema, batch examples, and best practices.

## Decision Map

| Scenario | Action |
|---|---|
| Evaluate all feature flags on launch | POST evaluate with user, no filter |
| Evaluate only mobile-tagged feature flags | POST evaluate with `filter.tags: ["mobile"]` |
| Evaluate one specific feature flag | POST evaluate with `filter.keys: ["flag-key"]` |
| Detect feature flag changes since last poll | POST evaluate with `filter.timestamp: {lastPollMs}` |
| Record which variation user saw | POST track with `variations` array |
| Record A/B test conversion | POST track with `metrics` array |
| Record both at once | POST track with both `variations` and `metrics` in one payload |

## Mobile SDK Implementation Checklist

Copy and track progress:
- [ ] Step 1: Store evaluation server URL and secret key in app config (never hardcode in source)
- [ ] Step 2: At app startup, call evaluate endpoint and cache results in memory
- [ ] Step 3: Fire-and-forget: post feature flag variation insights immediately after evaluation
- [ ] Step 4: Implement background polling with `filter.timestamp` to pick up feature flag changes
- [ ] Step 5: Expose a `getFlag(key, defaultValue)` helper that reads from cache synchronously
- [ ] Step 6: On relevant user action, post metric insights asynchronously
- [ ] Step 7: On user context change (login, upgrade), re-evaluate immediately

## Reference Files

| File | Read when |
|---|---|
| `references/flag-evaluation-api.md` | User asks about exact request/response schema, filter parameters, error codes, or single-flag evaluation |
| `references/track-insights-api.md` | User asks about insights schema, batching multiple users, MetricInsight fields, or best practices |
