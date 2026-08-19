---
name: Tool Adapter: FeatBit SDK
description: FeatBit SDK track() usage, experiment event association via sendToExperiment, and instrumentation integration patterns.
---

# Tool Adapter: FeatBit SDK

**Vendor:** FeatBit  
**Tool type:** SDK (server-side and client-side)  
**Default stage:** measurement design (CF-05)

This file documents how to use the FeatBit SDK to emit custom events that support experiment analysis.

## TOC

- [Core Concept](#core-concept)
- [Track a Custom Event](#track-a-custom-event)
- [Linking Events to Experiments](#linking-events-to-experiments)
- [sendToExperiment](#sendtoexperiment)
- [Tracking Guardrail Events](#tracking-guardrail-events)
- [SDK Reference](#sdk-reference)

---

## Core Concept

FeatBit experiments tie flag evaluations to custom events via a shared `user_key`. When `track()` is called after a flag evaluation — using the same user — FeatBit computes per-variant metric aggregates across the experiment.

The linkage is: **same user_key in evaluation + same user_key in track() = event attributed to that user's variant**.

---

## Track a Custom Event

### Server-side: Node.js

```js
fbClient.track('user_completed_onboarding', user, {
  sessionId: session.id,
  durationSeconds: session.duration
});
```

### Server-side: .NET

```csharp
fbClient.Track("user_completed_onboarding", user, new {
  sessionId = session.Id,
  durationSeconds = session.Duration
});
```

### Server-side: Python

```python
fb_client.track('user_completed_onboarding', user, {
  'session_id': session.id,
  'duration_seconds': session.duration
})
```

### Client-side: React

```js
const { track } = useFbClient();

track('ai_skill_session_started', {
  source: 'top_nav',
  sessionId: sessionId
});
```

For other languages (Java, Go, React Native), see the [FeatBit SDK documentation](https://docs.featbit.co/sdk/overview).

---

## Linking Events to Experiments

Evaluation order matters:

```js
// 1. Evaluate the flag — user is assigned a variant
const variant = fbClient.variation('new-checkout-flow', user, 'control');

// 2. User takes the action (checkout, onboarding step, etc.)

// 3. Track the event WITH THE SAME USER
//    This attributes the event to the user's variant assignment
fbClient.track('checkout_completed', user, { orderId: order.id });
```

If the `user` object in step 3 has a different `user_key` than step 1, the event will not be attributed to the correct variant.

---

## sendToExperiment

Some FeatBit SDK versions require explicitly marking an event as experiment-eligible:

```js
fbClient.track('checkout_completed', user, {
  orderId: order.id,
  $sendToExperiment: true
});
```

Consult your SDK version's documentation to confirm whether this parameter is required.

---

## Tracking Guardrail Events

Track guardrail events the same way as primary events. FeatBit can display multiple per-variant metrics side-by-side in the experiment dashboard.

```js
// Guardrail: page error rate
fbClient.track('page_error_encountered', user, {
  errorCode: err.code,
  path: window.location.pathname
});
```

---

## SDK Reference

Full documentation for all supported languages: [FeatBit SDK documentation](https://docs.featbit.co/sdk/overview)

Supported SDKs: Node.js, .NET, Python, Java, Go, React, React Native, JavaScript (browser)  
For platforms without an official SDK, see `featbit-evaluation-insights-api` for the HTTP-based evaluation and track API.
