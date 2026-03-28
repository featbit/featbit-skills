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

Use for browser-based JavaScript and TypeScript applications that evaluate feature flags client-side — vanilla JS apps, SPAs, or any non-React framework. For React, use `featbit-sdks-react` instead. Designed for single-user browser contexts; stores feature flag data in localStorage and syncs via WebSocket streaming or HTTP polling. Do not use for Node.js server-side applications — those require `featbit-sdks-node`.

## Execution Procedure

```
1. Install @featbit/js-client-sdk via npm
2. Create FbClient
   a. Build a User with key + optional name/custom props
   b. Configure sdkKey, streamingUri, eventsUri
   c. Attach user and build the client
3. Initialize
   a. await fbClient.waitForInitialization()
   b. On failure → verify sdkKey, streamingUri, eventsUri, user → retry once
   c. If retry fails → report error and config to user, stop
4. Evaluate feature flags
   a. boolVariation / stringVariation / numberVariation / jsonVariation
   b. Use *Detail variants when evaluation reason is needed
5. (Optional) Listen for feature flag changes via event subscription
6. (Optional) Switch user at runtime via identify()
7. Close client when done
```

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
    .streamingUri('ws://localhost:5100')
    .eventsUri('http://localhost:5100')
    .user(user)
    .build();
```

**Step 3: Evaluate the first feature flag**

Use the official pattern:

```typescript
await fbClient.waitForInitialization();
const boolVariation = await fbClient.boolVariation('flag-key', false);
```

If initialization does not complete or the first value is the fallback unexpectedly, verify `sdkKey`, `streamingUri`, `eventsUri`, and the initial user, then retry once. If the retry also fails, report the error message and current configuration to the user and stop.

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

