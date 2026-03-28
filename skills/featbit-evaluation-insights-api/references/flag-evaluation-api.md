---
name: flag-evaluation-api
description: FeatBit feature flag evaluation REST API endpoint reference with request/response schemas, filtering, polling, and error handling
---

# Flag Evaluation API Reference

**Endpoint**: `POST {evaluation-server-url}/api/public/featureflag/evaluate`
**Official docs**: https://docs.featbit.co/api-docs/flag-evaluation-api
**Source**: https://github.com/featbit/featbit-docs/blob/main/pages/api-docs/flag-evaluation-api.md

## Table of Contents

1. [Authentication](#authentication)
2. [Request Schema](#request-schema)
3. [Response Schema](#response-schema)
4. [Filter Options](#filter-options)
5. [Single-Flag Evaluation](#single-flag-evaluation)
6. [Polling Pattern with Timestamp](#polling-pattern-with-timestamp)
7. [Error Responses](#error-responses)

---

## Authentication

Include the environment secret key in the `Authorization` header on every request:

```http
Authorization: your-environment-secret-key
```

Use either the **client-side** or **server-side** key — both are accepted.

> How to find it: FeatBit dashboard → Project → Environment → Secret Keys.
> See also: https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret

---

## Request Schema

### Top-Level Object

| Parameter | Type | Required | Description |
|---|---|---|---|
| `user` | `EndUser` | Yes | The end user to evaluate flags for |
| `filter` | `FeatureFlagFilter` | No | Optional filter to limit which flags are evaluated |

### EndUser

| Field | Type | Required | Description |
|---|---|---|---|
| `keyId` | `string` | Yes | Unique, stable identifier for the user in your environment |
| `name` | `string` | No | Display name |
| `customizedProperties` | `Array<CustomProperty>` | No | Custom attributes for targeting rules |

### CustomProperty

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | Yes | Attribute name (e.g., `"country"`, `"plan"`) |
| `value` | `string` | Yes | Attribute value (always string) |

### FeatureFlagFilter

| Field | Type | Default | Description |
|---|---|---|---|
| `keys` | `string[]` | `[]` | Evaluate only these specific flag keys. Empty means all flags. |
| `tags` | `string[]` | `[]` | Filter flags by tags |
| `tagFilterMode` | `string` | `"and"` | `"and"` — flag must have ALL specified tags. `"or"` — flag must have ANY tag. |
| `timestamp` | `number` | `0` | Unix ms. Returns only flags modified **after** this timestamp. Use for polling. |

### Full Request Example

```json
{
  "user": {
    "keyId": "user-123",
    "name": "John Doe",
    "customizedProperties": [
      { "name": "country", "value": "US" },
      { "name": "plan", "value": "premium" },
      { "name": "appVersion", "value": "3.2.1" }
    ]
  },
  "filter": {
    "tagFilterMode": "and",
    "tags": ["frontend", "mobile"],
    "keys": [],
    "timestamp": 0
  }
}
```

---

## Response Schema

The response is a **JSON array** of evaluation results — one item per matching flag.

### EvalResult (array element)

| Field | Type | Description |
|---|---|---|
| `key` | `string` | Feature flag key |
| `variation` | `EvalResultVariation` | Evaluated variation for the user |

### EvalResultVariation

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Variation UUID — save for use in insights tracking |
| `type` | `string` | Value type: `"boolean"`, `"string"`, `"number"`, `"json"` |
| `value` | `string` | Variation value — **always a string**, cast as needed |
| `matchReason` | `string` | Why this variation was selected (see below) |
| `sendToExperiment` | `boolean` | Whether to include this evaluation in experiment analysis. Pass unchanged to the Track Insights API. |

### matchReason Values

| Value | Meaning |
|---|---|
| `"flag disabled"` | The flag is turned off; default variation was served |
| `"targeted"` | User matched an individual target |
| `"{rule name}"` | User matched a named targeting rule |
| `"default"` | User matched the default (fallthrough) rule |

### Type Casting

`variation.value` is always a `string`. Cast by `variation.type`:

| `type` | Cast example (Kotlin) |
|---|---|
| `"boolean"` | `value.toBoolean()` |
| `"number"` | `value.toDouble()` or `value.toInt()` |
| `"json"` | `gson.fromJson(value, MyClass::class.java)` |
| `"string"` | Use as-is |

### Full Response Example

```json
[
  {
    "key": "new-dashboard-design",
    "variation": {
      "id": "08aceef3-5513-4b38-80ad-4b27bebe8871",
      "type": "boolean",
      "value": "true",
      "matchReason": "premium user rule",
      "sendToExperiment": true
    }
  },
  {
    "key": "theme-color",
    "variation": {
      "id": "c5364cfd-292a-4e06-8e8e-b0ae7387f791",
      "type": "string",
      "value": "dark",
      "matchReason": "default",
      "sendToExperiment": false
    }
  },
  {
    "key": "max-file-upload-size",
    "variation": {
      "id": "34683e86-b436-4694-b335-29628c0896e3",
      "type": "number",
      "value": "104857600",
      "matchReason": "premium user rule",
      "sendToExperiment": true
    }
  }
]
```

---

## Filter Options

### Evaluate All Flags (No Filter)

Omit the `filter` field or pass an empty object. Every flag in the environment is evaluated. Best for initial SDK bootstrap on startup.

```json
{ "user": { "keyId": "user-123" } }
```

### Evaluate Flags by Tags

Returns only flags that carry the specified tags:

```json
{
  "user": { "keyId": "user-123" },
  "filter": {
    "tags": ["mobile"],
    "tagFilterMode": "and"
  }
}
```

Use `"tagFilterMode": "or"` when you want flags that have **any** of the tags.

### Evaluate Specified Flags by Key

```json
{
  "user": { "keyId": "user-123" },
  "filter": {
    "keys": ["checkout-flow", "dark-mode"]
  }
}
```

---

## Single-Flag Evaluation

Pass exactly one key in `filter.keys`. The response is still an array with one element.

### Request

```json
{
  "user": {
    "keyId": "user-123",
    "customizedProperties": [
      { "name": "country", "value": "US" }
    ]
  },
  "filter": {
    "keys": ["new-dashboard-design"]
  }
}
```

### Response

```json
[
  {
    "key": "new-dashboard-design",
    "variation": {
      "id": "ac727073-8cc5-4ec5-9405-59737ea737bb",
      "type": "boolean",
      "value": "false",
      "matchReason": "flag disabled",
      "sendToExperiment": false
    }
  }
]
```

---

## Polling Pattern with Timestamp

Use `filter.timestamp` to implement efficient delta polling — only receive flags changed since the last successful poll.

1. First call: `timestamp: 0` — receive all flags; record current time as `lastPollTime`.
2. Subsequent calls: `timestamp: lastPollTime` — receive only modified flags; update `lastPollTime`.
3. Merge response with local cache: update changed flags, leave unchanged flags intact.

```json
{
  "user": { "keyId": "user-123" },
  "filter": {
    "timestamp": 1704067200000
  }
}
```

If no flags changed after the timestamp, the response is an empty array `[]`.

---

## Error Responses

### 401 Unauthorized

The `Authorization` header is missing or the secret key is invalid.

### 400 Bad Request

**Missing or invalid user:**
```json
{ "message": "A valid user is required." }
```

**Invalid tagFilterMode:**
```json
{ "message": "Invalid tag filter mode: invalid. Valid values are 'and' or 'or'." }
```
