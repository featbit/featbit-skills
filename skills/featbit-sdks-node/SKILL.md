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

Server-side Node.js and TypeScript SDK for real-time feature flag evaluation in web servers, REST APIs, and background workers. Maintains one persistent WebSocket connection per process, synchronizes all feature flag data locally in under 100 ms, and evaluates every feature flag locally without a remote call. Do not use for browser JavaScript (use `featbit-sdks-javascript`), React (`featbit-sdks-react`), or React Native (`featbit-sdks-react-native`).

## Execution Procedure

```
1. Install package
   npm install --save @featbit/node-server-sdk

2. Build FbClient
   new FbClientBuilder()
     .sdkKey(envSecret)
     .streamingUri(wsUri)
     .eventsUri(httpUri)
     .build()

3. Evaluate first feature flag
   await fbClient.waitForInitialization()
   user = new UserBuilder(key).name(name).build()
   value = await fbClient.boolVariation(flagKey, user, fallback)

4. Integrate into application
   Wire FbClient into request handlers / worker loops.
   Use boolVariation / stringVariation / numberVariation / jsonVariation as needed.

5. Close client on shutdown
   await fbClient.close()
```

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

Use `boolVariation` when only the feature flag value is needed. Use `boolVariationDetail` when the evaluation reason is also needed.

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

## Custom Event Tracking

Use `track` to send a custom event for experiments or conversion metrics after the related feature flag has already been evaluated for the same user:

```typescript
const flagKey = 'checkout-redesign';
const user = new UserBuilder('user-123').name('Jane').build();

await fbClient.waitForInitialization();

const enabled = await fbClient.boolVariation(flagKey, user, false);

if (enabled) {
    fbClient.track(user, 'purchase-completed', 99.5);
}

await fbClient.close();
```

Use `fbClient.track(user, eventName)` when the event is a simple counter. Use `fbClient.track(user, eventName, numericValue)` when the event also carries a numeric metric such as revenue, score, or quantity. `numericValue` is optional and defaults to `1.0`.

Call `track` only after the related feature flag evaluation call. If `track` runs before the feature flag is evaluated for that user, the custom event will not be included in experiment results.

## OpenFeature Integration

Install three packages:

```bash
npm install @openfeature/server-sdk
npm install @featbit/node-server-sdk
npm install @featbit/openfeature-provider-node-server
```

Register the FeatBit provider and evaluate feature flags via the OpenFeature client:

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

See [openfeature-provider-node-server](https://github.com/featbit/openfeature-provider-node-server) for `ProviderEvents.ConfigurationChanged` and feature flag change subscription examples.

