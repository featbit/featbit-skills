---
name: tracking-api-and-sdks
description: Wire format and per-language SDK helpers for recording FeatBit experiment events. Covers POST /api/track/event for both flag-evaluation exposures and metric events, the user.keyId + timestamp attribution rules, language helpers (Node.js, .NET, Java, Go, Python, Browser JS, React), and the three places event data can live (FeatBit flag-evaluation insights / FeatBit track-service / your own warehouse). Read when implementing experiment instrumentation, troubleshooting attribution drops, or choosing where to land experiment data.
---

# FeatBit Experiment APIs & SDKs

Two endpoints, one ingest service (`track-service`). Use this guide to wire flag-evaluation and metric events into a FeatBit experiment.

- **Base URL:** `https://track.featbit.ai`
- **Auth:** `Authorization: <env-secret>` (the `rat-env-v1`-shaped token from your environment settings)
- **Single event:** `POST /api/track/event`
- **Batch:** `POST /api/track` (array of the same event body)

## Contents

1. [Get Started — the four steps of an experiment](#1-get-started--the-four-steps-of-an-experiment)
2. [APIs — raw HTTP](#2-apis--raw-http) — flag-evaluation body, metric body (binary / continuous / duration), timestamp rule, ingest behavior
3. [SDKs — recording flag evaluations from code](#3-sdks--recording-flag-evaluations-from-code) — Node.js, .NET, Java, Go, Python, Browser JS, React
4. [Best Practice — where the `flag_evaluation` record should live](#4-best-practice--where-the-flag_evaluation-record-should-live) — FeatBit insights vs track-service vs your own warehouse

---

## 1. Get Started — the four steps of an experiment

1. **You have a hypothesis. You open an experiment.** Nothing to instrument yet.
2. **Split traffic with a feature flag, and record every flag evaluation back to the platform.**
   The experiment can't tell variants apart unless every evaluation is reported. One call per evaluation site. → see *APIs* (raw HTTP) or *SDKs* (one-liner per language).
3. **Record the metric data that decides success** — conversion rate, average duration, total revenue, etc.
   One event per moment that matters (checkout done, page loaded, purchase made). Metric events have their own shape, separate from flag evaluations. The SDKs section covers flag evaluation only — **metric events always go through the raw API**.
4. **Events flow into the experiment data pool. When the run ends, the analysis engine reads them and produces the result.** You don't touch the pool or the analyzer — both are handled.

```
  ┌──────────────────────┐
  │      your app        │
  │                      │
  │  ② flag evaluated ───┼──►  POST /api/track/event    ┐
  │                      │     { user, variations }      │
  │                      │                                ├──►  experiment data pool
  │  ③ user converted ───┼──►  POST /api/track/event    │
  │                      │     { user, metrics }         ┘          │
  └──────────────────────┘                                            │
                                                                      ▼
                                               ④ analysis engine  ──►  result
```

> ① happens in your head (hypothesis). ② and ③ are code you write. ④ is automatic.

---

## 2. APIs — raw HTTP

`POST /api/track/event` is the single entry point for both flag evaluations and metric events. Different body shape, same URL.

### Recording traffic events (flag evaluations)

One `variations[]` entry per flag evaluation. The same `user.keyId` on the metric event later is what lets the query layer attribute conversions back to the variant.

```bash
curl -X POST https://track.featbit.ai/api/track/event \
  -H "Authorization: rat-env-v1" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "keyId": "user-123",                  // required, stable user id — THE join key with metric events
      "properties": { "country": "US" }     // optional flat map; lets you slice results later (country, plan, platform…)
    },
    "variations": [{
      "flagKey":      "new-checkout",       // required — must match the flag key analyzed downstream
      "variant":      "treatment",          // required — usually "control" / "treatment"; any string your experiment configures
      "timestamp":    1776300000000,        // exposure time, epoch ms (Date.now()). Analysis uses the FIRST exposure; later ones are ignored
      "experimentId": "exp-checkout-q2",    // optional — set only when this exposure attributes to a specific run
      "layerId":      "checkout-layer"      // optional — for mutually-exclusive layers (a user is in at most one experiment per layer)
    }]
  }'
```

### Recording metric events

Same endpoint. Two contracts: `user.keyId` must match the one used at exposure, and `metric.timestamp` must be ≥ the exposure timestamp.

**Conversion rate (binary event)** — fire the event once, no `numericValue`. Denominator is users exposed; numerator is users who fired the event at least once post-exposure.

```bash
curl -X POST https://track.featbit.ai/api/track/event \
  -H "Authorization: rat-env-v1" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "keyId": "user-123"                   // MUST match the keyId used at exposure, or attribution drops the event
    },
    "metrics": [{
      "eventName": "checkout-completed",    // required — must match the metric configured on the experiment
      "timestamp": 1776300060000            // event time, epoch ms; must be ≥ exposure timestamp or it will not be attributed
    }]
  }'
```

**Revenue (continuous value)** — fire the event per transaction and put the amount in `numericValue`. Pick a consistent unit (cents, or primary currency unit) — stats-service sums as-is.

```bash
curl -X POST https://track.featbit.ai/api/track/event \
  -H "Authorization: rat-env-v1" \
  -H "Content-Type: application/json" \
  -d '{
    "user": { "keyId": "user-123" },
    "metrics": [{
      "eventName":    "purchase",
      "timestamp":    1776300060000,
      "numericValue": 42.50                 // quantity (consistent unit) — omit for binary conversion events
    }]
  }'
```

**Duration / latency** — same shape as revenue. Put the duration in `numericValue` (milliseconds is the convention across stats-service).

```bash
curl -X POST https://track.featbit.ai/api/track/event \
  -H "Authorization: rat-env-v1" \
  -H "Content-Type: application/json" \
  -d '{
    "user": { "keyId": "user-123" },
    "metrics": [{
      "eventName":    "page-load",
      "timestamp":    1776300060000,
      "numericValue": 842                   // duration in ms (the stats-service convention)
    }]
  }'
```

### Timestamp rule

All timestamps are epoch milliseconds (`Date.now()` in JS). The query layer attributes each user to their **first** exposure and only counts metric events where `metric.timestamp ≥ exposure.timestamp`. Events that arrive out of order are dropped from attribution — make sure your SDK does not backdate metric timestamps.

### What happens after ingest

Track-service writes into a bounded in-memory queue (`100k` events) and flushes to ClickHouse every `5s` or `1000` events. Flag exposures land in `featbit.flag_evaluations`; metric events land in `featbit.metric_events`. Both tables are queried by stats-service when you analyze a run.

---

## 3. SDKs — recording flag evaluations from code

How to record `flag_evaluation` correctly from code that already uses a FeatBit SDK.

### Two rules

1. **Wrap the track API in a project-internal helper.**
   Do not open-code a `fetch(...)` at each flag site. Wrap it once — name it `trackFlagForExpt(user, variant)` or similar — so every call site is a one-liner, the URL/envId are in one place, and you can swap the transport later (batch, fire-and-forget, queue) without touching business code.
2. **Call `trackFlagForExpt(user, variant)` immediately after `.boolVariation()` (or the SDK equivalent).**
   Exposure is the moment the variant influences behavior, not the moment the feature renders. Fire the track call in the same code path, with the same `user.keyId` the SDK evaluated against. For scenarios where FeatBit has no SDK, use the raw APIs above and still wrap them the same way.

Each language example below is two pieces: ① a helper you put in one shared file, ② how the call site looks right after the FeatBit SDK evaluation.

### Node.js

```ts
// lib/track.ts — wrap once, reuse everywhere
export async function trackFlagForExpt(
  user: { keyId: string; properties?: Record<string, string> },
  flagKey: string,
  variant: string,
  experimentId?: string,
) {
  await fetch(`${TRACK_URL}/api/track/event`, {
    method:  "POST",
    headers: { "Content-Type": "application/json", Authorization: ENV_ID },
    body: JSON.stringify({
      user,
      variations: [{ flagKey, variant, timestamp: Date.now(), experimentId }],
    }),
  });
}
```

```ts
import { UserBuilder } from "@featbit/node-server-sdk";
import { trackFlagForExpt } from "@/lib/track";

const user    = new UserBuilder("user-123").name("Alice").build();
const variant = (await fbClient.boolVariation("new-checkout", user, false))
  ? "treatment"
  : "control";

// ⬇ one-line report; same user.keyId flows through to metric events later
await trackFlagForExpt({ keyId: "user-123" }, "new-checkout", variant, "exp-checkout-q2");
```

### .NET

```csharp
// Services/ExperimentTracker.cs — wrap once, reuse everywhere
public sealed class ExperimentTracker(HttpClient http)
{
    public Task TrackFlagForExptAsync(
        string keyId, string flagKey, string variant, string? experimentId = null)
        => http.PostAsJsonAsync("http://track-service:8080/api/track/event", new {
            user       = new { keyId },
            variations = new[] { new {
                flagKey,
                variant,
                timestamp    = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
                experimentId,
            } },
        });
}
```

```csharp
var user    = FbUser.Builder("user-123").Name("Alice").Build();
var variant = _fbClient.BoolVariation("new-checkout", user, defaultValue: false)
    ? "treatment"
    : "control";

// ⬇ one-line report; same keyId flows through to metric events later
await _tracker.TrackFlagForExptAsync("user-123", "new-checkout", variant, "exp-checkout-q2");
```

### Java

```java
// ExperimentTracker.java — wrap once, reuse everywhere
public class ExperimentTracker {
    public void trackFlagForExpt(
            String keyId, String flagKey, String variant, String experimentId) throws Exception {
        var body = mapper.writeValueAsString(Map.of(
            "user",       Map.of("keyId", keyId),
            "variations", List.of(Map.of(
                "flagKey",      flagKey,
                "variant",      variant,
                "timestamp",    System.currentTimeMillis(),
                "experimentId", experimentId))));
        httpClient.send(HttpRequest.newBuilder()
            .uri(URI.create("http://track-service:8080/api/track/event"))
            .header("Authorization", "rat-env-v1")
            .header("Content-Type",  "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(body))
            .build(), HttpResponse.BodyHandlers.discarding());
    }
}
```

```java
FBUser user    = new FBUser.Builder("user-123").userName("Alice").build();
String variant = client.boolVariation("new-checkout", user, false)
    ? "treatment"
    : "control";

// ⬇ one-line report; same keyId flows through to metric events later
tracker.trackFlagForExpt("user-123", "new-checkout", variant, "exp-checkout-q2");
```

### Go

```go
// pkg/track/track.go — wrap once, reuse everywhere
func TrackFlagForExpt(keyId, flagKey, variant, experimentId string) error {
    body, _ := json.Marshal(map[string]any{
        "user": map[string]any{"keyId": keyId},
        "variations": []map[string]any{{
            "flagKey":      flagKey,
            "variant":      variant,
            "timestamp":    time.Now().UnixMilli(),
            "experimentId": experimentId,
        }},
    })
    req, _ := http.NewRequest("POST", "http://track-service:8080/api/track/event", bytes.NewReader(body))
    req.Header.Set("Authorization", "rat-env-v1")
    req.Header.Set("Content-Type",  "application/json")
    _, err := http.DefaultClient.Do(req)
    return err
}
```

```go
user, _ := featbit.NewUserBuilder("user-123").UserName("Alice").Build()
enabled, _, _ := client.BoolVariation("new-checkout", user, false)
variant := "control"
if enabled {
    variant = "treatment"
}

// ⬇ one-line report; same keyId flows through to metric events later
track.TrackFlagForExpt("user-123", "new-checkout", variant, "exp-checkout-q2")
```

### Python

```python
# track.py — wrap once, reuse everywhere
def track_flag_for_expt(key_id, flag_key, variant, experiment_id=None):
    requests.post(
        "http://track-service:8080/api/track/event",
        headers={"Authorization": "rat-env-v1"},
        json={
            "user":       {"keyId": key_id},
            "variations": [{
                "flagKey":      flag_key,
                "variant":      variant,
                "timestamp":    int(time.time() * 1000),
                "experimentId": experiment_id,
            }],
        },
    )
```

```python
from track import track_flag_for_expt

user    = {"key": "user-123", "name": "Alice"}
variant = "treatment" if client.variation("new-checkout", user, default=False) else "control"

# ⬇ one-line report; same key_id flows through to metric events later
track_flag_for_expt("user-123", "new-checkout", variant, "exp-checkout-q2")
```

### Browser JS

```js
// lib/track.js — wrap once, reuse everywhere
export async function trackFlagForExpt(keyId, flagKey, variant, experimentId) {
  await fetch("/api/track/event", {
    method:  "POST",
    headers: { "Content-Type": "application/json", Authorization: ENV_ID },
    body: JSON.stringify({
      user: { keyId },
      variations: [{ flagKey, variant, timestamp: Date.now(), experimentId }],
    }),
  });
}
```

```js
import { trackFlagForExpt } from "./lib/track.js";

// User is bound at fbClient.init(); no per-call user arg
const variant = (await fbClient.boolVariation("new-checkout", false))
  ? "treatment"
  : "control";

// ⬇ one-line report; same keyId flows through to metric events later
await trackFlagForExpt(currentUserId, "new-checkout", variant, "exp-checkout-q2");
```

### React

```jsx
// hooks/useFlagForExpt.ts — wrap once, reuse everywhere
export function useFlagForExpt(flagKey: string, experimentId?: string) {
  const flags   = useFlags();
  const { keyId } = useUser();
  const variant = flags[flagKey] ? "treatment" : "control";

  useEffect(() => {
    fetch("/api/track/event", {
      method:  "POST",
      headers: { "Content-Type": "application/json", Authorization: ENV_ID },
      body: JSON.stringify({
        user: { keyId },
        variations: [{ flagKey, variant, timestamp: Date.now(), experimentId }],
      }),
    });
  }, [keyId, flagKey, variant, experimentId]);

  return variant;
}
```

```jsx
function Checkout() {
  // ⬇ hook returns the variant AND records the exposure in one call
  const variant = useFlagForExpt("new-checkout", "exp-checkout-q2");

  return variant === "treatment" ? <NewCheckout /> : <LegacyCheckout />;
}
```

---

## 4. Best Practice — where the `flag_evaluation` record should live

Three places it can live. Pick based on where your **analysis** is going to run, not where your flag service is.

### Decision matrix

| You have… | You want… | Record to |
|---|---|---|
| FeatBit flags only | Variant distribution, flag health, nothing about business metrics | **FeatBit flag-evaluation insights** |
| FeatBit flags + want managed analysis | Full experiment analysis without building your own warehouse | **FeatBit data warehouse (track-service)** |
| Your own data warehouse + FeatBit flags | Experiment analysis sitting next to your other product data | **Your own warehouse** |

### Option A — FeatBit flag-evaluation insights

Zero app instrumentation. FeatBit records every flag evaluation server-side; insights dashboards show variant distribution, flag reach, rule hits.

- **Use when:** you need to answer "is the flag firing and who's getting which variant" — not "did the variant change business metrics". No `flag_evaluation` event needs to leave the FeatBit stack; no metric events exist in this world.
- **Trade-off:** you cannot do cross-metric analysis (conversion / revenue / duration by variant) because business metrics don't live here. The moment you need that, switch to option B or C.
- **Pointer:** the `featbit-opentelemetry` and `featbit-deployment-*` agent skills, plus the docs.featbit.co pages on flag-evaluation insights.

### Option B — FeatBit data warehouse (track-service)

What this whole guide documents. Record both `flag_evaluation` and metric events to track-service; stats-service runs the analysis for you.

- **Use when:** you want experiment analysis but don't want to stand up a warehouse. Follow the SDKs section for `flag_evaluation` and the APIs section for metric events. Both land in ClickHouse, joined by `user.keyId`, queryable from the run-analysis UI.
- **Trade-off:** the raw event data lives in FeatBit, not in your warehouse. Cross-system analysis (joining experiment events against your CRM, billing, support data) requires an export.

### Option C — your own data warehouse

You already have Snowflake / BigQuery / Redshift / ClickHouse and want experiment events to land there so they sit next to every other product signal.

- **Use when:** your analytics team already lives in a warehouse and FeatBit is only the flag-decision service. Skip track-service entirely.
- **How:** keep the same two-rule pattern from the SDKs section, but point `trackFlagForExpt` at your own collector (Segment, RudderStack, Kafka topic, warehouse-ingest endpoint) instead of `/api/track/event`. The payload fields are yours to define; the invariant that survives is *same `user.keyId` on exposure and metric*.
- **Trade-off:** you own the schema, the ingestion pipeline, and the analysis layer. You lose stats-service — your analysts write the SQL.
