---
name: featbit-sdks-node
description: Expert guidance for integrating FeatBit Node.js Server SDK. Use when user asks about "Node.js SDK", "Node server SDK", "FeatBit Node", or mentions server-side TypeScript/JavaScript with FeatBit. Do not use for browser JavaScript, React, React Native, Python, Java, Go, or .NET questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit Node.js Server SDK

## When to Use This Skill

Use for server-side Node.js and TypeScript applications — web servers, REST APIs, background workers — that need real-time feature flag evaluation.

Why server-side SDK: maintains one persistent WebSocket connection per process, synchronizes all flag data locally in under 100 ms, and evaluates every flag locally without a remote call. Do not use for browser JavaScript (use `featbit-sdks-javascript`), React (`featbit-sdks-react`), or React Native (`featbit-sdks-react-native`).

## Source

https://github.com/featbit/featbit-node-server-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Build the client with `sdkKey`, `streamingUri`, and `eventsUri`
- [ ] Step 3: Wait for initialization and evaluate the first feature flag
- [ ] Step 4: If initialization fails, validate configuration and retry
- [ ] Step 5: Read advanced README sections only when needed

**Step 1: Install the package**

Run:

```bash
npm install --save @featbit/node-server-sdk
```

**Step 2: Build the client**

Use this minimal setup:

```typescript
import { FbClientBuilder, UserBuilder } from "@featbit/node-server-sdk";

const fbClient = new FbClientBuilder()
    .sdkKey('<your-env-secret>')
    .streamingUri('ws://localhost:5100')
    .eventsUri('http://localhost:5100')
    .build();
```

**Step 3: Evaluate the first feature flag**

Use the official pattern:

```typescript
await fbClient.waitForInitialization();
const user = new UserBuilder('<unique-user-key>').name('Jane').build();
const boolVariation = await fbClient.boolVariation('flag-key', user, false);
await fbClient.close();
```

**Step 4: Validate the integration**

If initialization hangs or evaluation returns the fallback unexpectedly, verify `sdkKey`, `streamingUri`, and `eventsUri`, then retry.

## Feature Flag Evaluation

After the client is initialized, evaluate a feature flag with a user and a fallback value:

```typescript
const flagKey = "game-runner";
const user = new UserBuilder('<unique-user-key>').name('Jane').build();

// value only
const boolVariation = await fbClient.boolVariation(flagKey, user, false);

// value with evaluation detail
const boolVariationDetail = await fbClient.boolVariationDetail(flagKey, user, false);
console.log(`Returns ${boolVariationDetail.value}, Kind: ${boolVariationDetail.kind}, Reason: ${boolVariationDetail.reason}`);
```

Use `boolVariation` when only the flag value is needed. Use `boolVariationDetail` when the evaluation reason is also needed.

Also available: `stringVariation`/`stringVariationDetail`, `numberVariation`/`numberVariationDetail`, `jsonVariation`/`jsonVariationDetail`.

## User Custom Properties

Add custom properties to a user when targeting rules depend on attributes beyond `key` and `name`:

```typescript
import { UserBuilder } from "@featbit/node-server-sdk";

const bob = new UserBuilder('unique_key_for_bob')
    .name('Bob')
    .custom('age', 18)
    .custom('country', 'FR')
    .build();
```

Use `.custom(key, value)` for any attribute that must be referenced in feature flag targeting rules. Values can be strings or numbers.

## OpenFeature Integration

Install three packages:

```bash
npm install @openfeature/server-sdk
npm install @featbit/node-server-sdk
npm install @featbit/openfeature-provider-node-server
```

Register the FeatBit provider and evaluate flags via the OpenFeature client:

```typescript
import { OpenFeature, ProviderEvents } from '@openfeature/server-sdk';
import { FbProvider } from '@featbit/openfeature-provider-node-server';

const provider = new FbProvider({
    sdkKey: '<your-sdk-key>',
    streamingUri: '<your-streaming-uri>',
    eventsUri: '<your-events-uri>'
});

OpenFeature.setProvider(provider);

// Evaluations before the provider is ready may return default values.
OpenFeature.addHandler(ProviderEvents.Ready, async () => {
    const client = OpenFeature.getClient();
    const value = await client.getBooleanValue('flag-key', false, { targetingKey: 'user-key' });
});
```

See [openfeature-provider-node-server](https://github.com/featbit/openfeature-provider-node-server) for `ProviderEvents.ConfigurationChanged` and flag change subscription examples.

## Read Next Only When Needed

- Read the official README section for [evaluating flags](https://github.com/featbit/featbit-node-server-sdk#evaluating-flags) when the user asks how to inspect evaluation detail or use non-boolean typed variants (`stringVariation`, `numberVariation`, `jsonVariation`).
- Read the official README section for [IUser](https://github.com/featbit/featbit-node-server-sdk#iuser) when the user asks about user attributes, custom properties, targeting fields, or user construction patterns.
- Read the official README sections for [FbClientNode](https://github.com/featbit/featbit-node-server-sdk#fbclientnode), [polling mode](https://github.com/featbit/featbit-node-server-sdk#fbclient-using-polling), [logger](https://github.com/featbit/featbit-node-server-sdk#logger), [offline mode](https://github.com/featbit/featbit-node-server-sdk#offline-mode), [disable events collection](https://github.com/featbit/featbit-node-server-sdk#disable-events-collection), and [experiments](https://github.com/featbit/featbit-node-server-sdk#experiments-abn-testing) only when those topics are requested.
