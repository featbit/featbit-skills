---
name: track-insights-api
description: FeatBit insights tracking API for recording flag evaluations, custom metric events, and experiment data
---

# Track Insights API Reference

**Endpoint**: `POST {evaluation-server-url}/api/public/insight/track`
**Official docs**: https://docs.featbit.co/api-docs/track-insights-api
**Source**: https://github.com/featbit/featbit-docs/blob/main/pages/api-docs/track-insights-api.md

## Table of Contents

1. [Purpose](#purpose)
2. [Authentication](#authentication)
3. [Request Schema](#request-schema)
4. [Use Cases](#use-cases)
   - [Track Flag Evaluations Only](#1-track-flag-evaluations-only)
   - [Track Custom Metric Events Only](#2-track-custom-metric-events-only)
   - [Track Both Together](#3-track-both-together)
   - [Batch Multiple Users](#4-batch-multiple-users)
5. [Response](#response)
6. [Error Responses](#error-responses)
7. [Best Practices](#best-practices)

---

## Purpose

The Track Insights API is the data pipeline for:

- **Experiment / A/B test analysis** — records which variation each user received, enabling statistical comparison between control and treatment groups
- **Disaster recovery & rollback audit** — provides a time-stamped log of every flag evaluation so you can trace when a bad variation was served and to whom
- **Custom metric conversion tracking** — correlates user actions (purchase, click, signup) with the flag variation they were served at the time

Always track asynchronously — never block the main UI thread or critical application paths for this call.

---

## Authentication

```http
Authorization: your-environment-secret-key
```

Same environment secret key used for the Evaluation API. Client-side or server-side key both work.

---

## Request Schema

The request body is a **JSON array** of `Insight` objects. Send multiple users or multiple events in one request.

### Insight (array element)

| Field | Type | Required | Description |
|---|---|---|---|
| `user` | `EndUser` | Yes | The user who received the evaluation or triggered the event |
| `variations` | `Array<VariationInsight>` | No | Flag evaluations to record |
| `metrics` | `Array<MetricInsight>` | No | Custom conversion events to record |

> At least one of `variations` or `metrics` should be non-empty in each insight object.

### EndUser

| Field | Type | Required | Description |
|---|---|---|---|
| `keyId` | `string` | Yes | Unique, stable user identifier (must match the `keyId` used in evaluation) |
| `name` | `string` | No | Display name |
| `customizedProperties` | `Array<CustomProperty>` | No | Custom attributes (should match those used during evaluation) |

### CustomProperty

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | Yes | Attribute name |
| `value` | `string` | Yes | Attribute value |

### VariationInsight

| Field | Type | Required | Description |
|---|---|---|---|
| `featureFlagKey` | `string` | Yes | Key of the evaluated flag |
| `variation` | `Variation` | Yes | The variation that was served |
| `sendToExperiment` | `boolean` | Yes | Copy this value exactly from the evaluation response's `variation.sendToExperiment` |
| `timestamp` | `number` | Yes | Unix ms when the evaluation occurred (not when you send the request) |

### Variation

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | `string` | Yes | Variation UUID — copy from evaluation response's `variation.id` |
| `value` | `string` | Yes | Variation value — copy from evaluation response's `variation.value` |

### MetricInsight

| Field | Type | Required | Description |
|---|---|---|---|
| `route` | `string` | Yes | Screen or route where the event occurred (e.g., `/checkout/complete`, `OrderConfirmationScreen`) |
| `type` | `string` | Yes | Always `"CustomEvent"` |
| `eventName` | `string` | Yes | Name of the event (e.g., `"purchase-completed"`, `"item-added"`) |
| `numericValue` | `number` | Yes | Numeric value associated with the event (use `1.0` for simple occurrence counts) |
| `appType` | `string` | Yes | `"Mobile"` for Android/iOS apps, `"Web"` for browser-based apps |
| `timestamp` | `number` | Yes | Unix ms when the event occurred |

---

## Use Cases

### 1. Track Flag Evaluations Only

Record which variations were served, with no metrics:

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
          "id": "3c9f8e2a-5b1d-4f7c-a3e6-8d4b2c1a9f5e",
          "value": "true"
        },
        "sendToExperiment": true,
        "timestamp": 1704067200000
      },
      {
        "featureFlagKey": "theme-color",
        "variation": {
          "id": "c5364cfd-292a-4e06-8e8e-b0ae7387f791",
          "value": "dark"
        },
        "sendToExperiment": false,
        "timestamp": 1704067200000
      }
    ],
    "metrics": []
  }
]
```

All `variation` field values are copied directly from the evaluation response. Never construct them independently.

---

### 2. Track Custom Metric Events Only

Record a conversion event without associating it with a specific flag evaluation. Useful when a meaningful user action happens after the evaluation context is gone:

```json
[
  {
    "user": {
      "keyId": "user-123",
      "name": "John Doe"
    },
    "variations": [],
    "metrics": [
      {
        "route": "/api/checkout/complete",
        "type": "CustomEvent",
        "eventName": "purchase-completed",
        "numericValue": 299.99,
        "appType": "Mobile",
        "timestamp": 1704067350000
      }
    ]
  }
]
```

---

### 3. Track Both Together

Most efficient approach — combine flag evaluation records and conversion events in a single request:

```json
[
  {
    "user": {
      "keyId": "user-456",
      "name": "Jane Smith",
      "customizedProperties": [
        { "name": "country", "value": "USA" },
        { "name": "plan", "value": "enterprise" }
      ]
    },
    "variations": [
      {
        "featureFlagKey": "recommendation-algorithm",
        "variation": {
          "id": "7d2e4f3b-8c6a-4d9e-b2f7-1a5c3e8b6d4f",
          "value": "algorithm-v2"
        },
        "sendToExperiment": true,
        "timestamp": 1704067300000
      }
    ],
    "metrics": [
      {
        "route": "/api/checkout/complete",
        "type": "CustomEvent",
        "eventName": "purchase-completed",
        "numericValue": 299.99,
        "appType": "Web",
        "timestamp": 1704067350000
      },
      {
        "route": "/api/items/add-to-cart",
        "type": "CustomEvent",
        "eventName": "item-added",
        "numericValue": 1.0,
        "appType": "Web",
        "timestamp": 1704067250000
      }
    ]
  }
]
```

---

### 4. Batch Multiple Users

Send insights for multiple users in one HTTP request — reduces overhead for server-side or batch processing scenarios:

```json
[
  {
    "user": { "keyId": "user-001", "name": "Alice" },
    "variations": [
      {
        "featureFlagKey": "feature-x",
        "variation": {
          "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
          "value": "control"
        },
        "sendToExperiment": true,
        "timestamp": 1704067200000
      }
    ],
    "metrics": []
  },
  {
    "user": { "keyId": "user-002", "name": "Bob" },
    "variations": [
      {
        "featureFlagKey": "feature-x",
        "variation": {
          "id": "f9e8d7c6-b5a4-4938-8271-6a5b4c3d2e1f",
          "value": "treatment"
        },
        "sendToExperiment": true,
        "timestamp": 1704067300000
      }
    ],
    "metrics": []
  }
]
```

---

## Response

### 200 OK

Empty body. Insights have been accepted and queued for processing. This is a fire-and-forget endpoint — do not wait for processing confirmation.

---

## Error Responses

### 401 Unauthorized

- `Authorization` header is missing, or
- The environment secret key is invalid.

### 400 Bad Request

- Request body is not valid JSON
- Required fields are missing (e.g., `user.keyId` is absent)
- Data types don't match the schema

---

## Best Practices

1. **Fire and forget** — Post insights on a background thread / async coroutine. Never block UI or business logic waiting for this response.

2. **Use batching** — Accumulate multiple insights in a local buffer and flush in one request every few seconds rather than one request per evaluation.

3. **Copy variation fields exactly** — `variation.id`, `variation.value`, and `sendToExperiment` must be copied verbatim from the evaluation response. Do not reconstruct or transform them.

4. **Accurate timestamps** — Record `Date.now()` (Unix ms) at the moment of evaluation or event, not at the moment you flush the buffer.

5. **Consistent `keyId`** — Use the same stable user identifier across evaluation and insight calls. If the `keyId` differs, FeatBit cannot correlate evaluations with metrics.

6. **Secure your secret key** — Never expose your environment secret key in client-side code.

7. **Handle errors silently** — If the insight track call fails, log the error but do not surface it to the user and do not retry indefinitely. Lost insight records are preferable to degraded user experience.

8. **Include `customizedProperties`** — Rich user attributes enable better segmentation and retroactive analysis in FeatBit's analytics dashboard.
