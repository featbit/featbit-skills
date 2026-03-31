---
name: featbit-sdks-javascript
description: Expert guidance for integrating FeatBit JavaScript Client SDK in browser environments. Use when user asks about "JavaScript client SDK", "browser feature flags", "FeatBit JS", "vanilla JS SDK", or mentions browser-side HTML/JS with FeatBit. Do not use for Node.js server-side, React, React Native, .NET, Python, Java, or Go questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit JavaScript Client SDK

## When to Use This Skill

Use for browser-based JavaScript and TypeScript applications that evaluate feature flags client-side — vanilla JS apps, SPAs, or any non-React framework. For React, use `featbit-sdks-react` instead.

Why client-side SDK: designed for single-user browser contexts, stores feature flag data in localStorage, and syncs via WebSocket streaming or HTTP polling. Do not use for Node.js server-side applications — those require `featbit-sdks-node`.

## Source

https://github.com/featbit/featbit-js-client-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Build the browser client with a user
- [ ] Step 3: Wait for readiness and evaluate the first feature flag
- [ ] Step 4: If events or user switching are needed, read the specific README section

**Step 1: Install the package**

Run:

```bash
npm install --save @featbit/js-client-sdk
```

**Step 2: Build the browser client**

Use this minimal setup:

```typescript
import { FbClientBuilder, UserBuilder } from "@featbit/js-client-sdk";

const user = new UserBuilder('<unique-user-key>').name('Jane').build();

const fbClient = new FbClientBuilder()
    .sdkKey('<your-env-secret>')
    .streamingUri('ws://localhost:5100')   // replace with your FeatBit server URL
    .eventsUri('http://localhost:5100')    // replace with your FeatBit server URL
    .user(user)
    .build();
```

**Step 3: Evaluate the first feature flag**

Use the official pattern:

```typescript
await fbClient.waitForInitialization();
const boolVariation = await fbClient.boolVariation('flag-key', false);
```

If initialization does not complete or the first value is the fallback unexpectedly, verify `sdkKey`, `streamingUri`, `eventsUri`, and the initial user, then retry.

## Feature Flag Evaluation

After the client is ready, evaluate a feature flag with a fallback value:

```typescript
const flagKey = "game-runner";

// value only
const boolVariation = await fbClient.boolVariation(flagKey, false);

// value with evaluation detail
const boolVariationDetail = await fbClient.boolVariationDetail(flagKey, false);
```

All variation calls return a `Promise` — always use `await`. Use `boolVariation` when only the feature flag value is needed. Use `boolVariationDetail` when the evaluation reason is also needed.

Also available: `stringVariation`/`stringVariationDetail`, `numberVariation`/`numberVariationDetail`, `jsonVariation`/`jsonVariationDetail`.

## User Custom Properties

Add custom properties to a user when targeting rules depend on attributes beyond `key` and `name`:

```typescript
import { UserBuilder } from "@featbit/js-client-sdk";

const user = new UserBuilder('a-unique-key-of-user')
    .name('bob')
    .custom('age', 18)
    .custom('country', 'FR')
    .build();
```

Use `.custom(key, value)` for any attribute that must be referenced in feature flag targeting rules. Values can be strings or numbers.

## OpenFeature Integration

To use FeatBit with the OpenFeature standard, install three packages:

```bash
npm install @openfeature/web-sdk featbit-js-client-sdk @featbit/openfeature-provider-js-client
```

See the [openfeature-provider-js-client](https://github.com/featbit/openfeature-provider-js-client) repository for usage examples.

## Read Next Only When Needed

- Read the official README section for [evaluation](https://github.com/featbit/featbit-js-client-sdk#evaluation) when the user asks how to inspect evaluation detail or use non-boolean typed variants (`stringVariation`, `numberVariation`, `jsonVariation`).
- Read the official README section for [IUser](https://github.com/featbit/featbit-js-client-sdk#iuser) when the user asks about user attributes, custom properties, targeting fields, or user construction patterns.
- Read the official README section for [events](https://github.com/featbit/featbit-js-client-sdk#events) when the user asks how to subscribe to feature flag changes at runtime.
- Read the official README sections for [bootstrap](https://github.com/featbit/featbit-js-client-sdk#bootstrap), [offline mode](https://github.com/featbit/featbit-js-client-sdk#offline-mode), [switch user after initialization](https://github.com/featbit/featbit-js-client-sdk#switch-user-after-initialization), [polling mode](https://github.com/featbit/featbit-js-client-sdk#fbclient-using-polling), [disable events collection](https://github.com/featbit/featbit-js-client-sdk#disable-events-collection), and [experiments](https://github.com/featbit/featbit-js-client-sdk#experiments-abn-testing) only when those topics are requested.
