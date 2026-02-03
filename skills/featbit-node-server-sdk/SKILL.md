---
name: featbit-node-server-sdk
description: Expert guidance for integrating FeatBit Node.js Server-Side SDK in backend applications. Use when building Node.js server apps with feature flags, A/B testing, gradual rollouts, or OpenFeature integration. Covers SDK setup, flag evaluation, and event tracking.
appliesTo:
  - "**/server.js"
  - "**/app.js"
  - "**/index.js"
  - "**/*.server.js"
  - "**/api/**/*.js"
  - "**/backend/**/*.js"
---

# FeatBit Node.js Server SDK

Expert guidance for integrating FeatBit Server-Side SDK in Node.js backend applications.

> **📦 Official SDK**: [@featbit/node-server-sdk](https://www.npmjs.com/package/@featbit/node-server-sdk)  
> **📖 Full Documentation**: [GitHub Repository](https://github.com/featbit/featbit-node-server-sdk)

⚠️ **Important**: This is a **server-side SDK** for multi-user systems (web servers, APIs). Not for browser/client-side applications.

## Quick Start

### Installation

```bash
npm install --save @featbit/node-server-sdk
```

### Prerequisites

Before using the SDK, obtain:
- **Environment Secret (SDK Key)**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)
  - FeatBit SaaS: `wss://app-eval.featbit.co` and `https://app-eval.featbit.co`
  - Self-hosted: `ws://localhost:5100` and `http://localhost:5100`

### Basic Usage

```javascript
import { FbClientBuilder, UserBuilder } from "@featbit/node-server-sdk";

// Setup SDK options
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .build();

(async () => {
  // Wait for the SDK to be initialized
  try {
    await fbClient.waitForInitialization();
  } catch(err) {
    // failed to initialize the SDK
    console.log(err);
  }

  // flag to be evaluated
  const flagKey = "game-runner";
  
  // create a user
  const user = new UserBuilder('a-unique-key-of-user')
    .name('bob')
    .custom('sex', 'female')
    .custom('age', 18)
    .build();

  // evaluate a feature flag for a given user
  const boolVariation = await fbClient.boolVariation(flagKey, user, false);
  console.log(`flag '${flagKey}' returns ${boolVariation} for user ${user.Key}`);

  // evaluate a boolean flag for a given user with evaluation detail
  const boolVariationDetail = await fbClient.boolVariationDetail(flagKey, user, false);
  console.log(`flag '${flagKey}' returns ${boolVariationDetail.value} for user ${user.Key}` +
    `Reason Kind: ${boolVariationDetail.kind}, Reason Description: ${boolVariationDetail.reason}`);

  // make sure the events are flushed before exit
  await fbClient.close();
})();
```

## Core Concepts

### 1. SDK Client (FbClient)

Applications should instantiate a **single instance** for the lifetime of the application.

**FbClient Using Streaming** (recommended):

```javascript
import { FbClientBuilder } from "@featbit/node-server-sdk";

const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .build();
```

**FbClient Using Polling**:

```javascript
import { FbClientBuilder, DateSyncMode } from "@featbit/node-server-sdk";

const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .dataSyncMode(DateSyncMode.POLLING)
    .pollingUri('https://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .pollingInterval(10000)
    .build();
```

### 2. User Context (IUser)

IUser defines the attributes of a user for whom you are evaluating feature flags. The **key** is mandatory and must uniquely identify each user.

```javascript
import { UserBuilder } from "@featbit/node-server-sdk";

const bob = new UserBuilder("unique_key_for_bob")
    .name("Bob")
    .custom('age', 18)
    .custom('country', 'FR')
    .build();
```

Alternatively, create a user object directly:

```typescript
import { IUser } from "@featbit/node-server-sdk";

const bob: IUser = {
  key: "unique_key_for_bob",
  name: "Bob",
  customizedProperties: [
    { name: "age", value: "18" },
    { name: "country", value: "FR" },
  ],
};
```

### 3. Flag Evaluation

The SDK **locally calculates** flag values using data it has already received.

**Variation methods** (return flag value):
- `boolVariation` / `boolVariationDetail`
- `stringVariation` / `stringVariationDetail`
- `numberVariation` / `numberVariationDetail`
- `jsonVariation` / `jsonVariationDetail`

```javascript
// flag to be evaluated
const flagKey = "game-runner";

// create a user
const user = new UserBuilder('a-unique-key-of-user')
    .name('bob')
    .custom('sex', 'female')
    .custom('age', 18)
    .build();

// evaluate a feature flag for a given user
const boolVariation = await fbClient.boolVariation(flagKey, user, false);
console.log(`flag '${flagKey}' returns ${boolVariation} for user ${user.Key}`);

// evaluate a boolean flag for a given user with evaluation detail
const boolVariationDetail = await fbClient.boolVariationDetail(flagKey, user, false);
console.log(`flag '${flagKey}' returns ${boolVariationDetail.value} for user ${user.Key}` +
    `Reason Kind: ${boolVariationDetail.kind}, Reason Description: ${boolVariationDetail.reason}`);
```

### 4. Experiments (A/B/n Testing)

Track custom events for experiments:

```javascript
fbClient.track(user, eventName, numericValue);
```

**numericValue** is optional (default: **1.0**).

**Important**: Call `track` **after** the related feature flag is evaluated.

## OpenFeature Integration

FeatBit provides an OpenFeature provider for Node.js server applications.

> **📦 OpenFeature Provider**: [@featbit/openfeature-provider-node-server](https://www.npmjs.com/package/@featbit/openfeature-provider-node-server)  
> **📖 Repository**: [GitHub](https://github.com/featbit/openfeature-provider-node-server)

### Installation

```bash
npm install @openfeature/server-sdk
npm install @featbit/node-server-sdk
npm install @featbit/openfeature-provider-node-server
```

### Quick Start

```javascript
import { OpenFeature, ProviderEvents } from '@openfeature/server-sdk';
import { FbProvider } from '@featbit/openfeature-provider-node-server';

const provider = new FbProvider({
    sdkKey: '<your-sdk-key>',
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co'
});

OpenFeature.setProvider(provider);

// Access FbClient if needed: provider.getClient()

// Wait for provider ready
OpenFeature.addHandler(ProviderEvents.Ready, (eventDetails) => {
    console.log(`Provider ready. Flags changed: ${eventDetails.flagsChanged}`);
});

// Listen for configuration changes
OpenFeature.addHandler(ProviderEvents.ConfigurationChanged, async (eventDetails) => {
    const client = OpenFeature.getClient();
    const value = await client.getBooleanValue('ff1', false, {targetingKey: 'my-key'});
    console.log({...eventDetails, value});
});

// Evaluate flags
const client = OpenFeature.getClient();
const value = await client.getBooleanValue('my-flag', false, {targetingKey: 'user-123'});
console.log(`Flag value: ${value}`);

// For short-lived processes, close to flush events
// await OpenFeature.close();
```

### Supported Node.js Versions

Compatible with Node.js versions **16 and above**.

## Configuration Options

### Streaming Mode (Recommended)

Real-time updates via WebSocket (default):

```javascript
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .startWaitTime(5000)           // Initialization timeout (ms)
    .reconnectInterval(15000)      // Reconnect interval (ms)
    .build();
```

### Polling Mode

```javascript
import { FbClientBuilder, DateSyncMode } from "@featbit/node-server-sdk";

const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .dataSyncMode(DateSyncMode.POLLING)
    .pollingUri('https://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .pollingInterval(10000)        // Poll every 10 seconds
    .build();
```

### Offline Mode

Use with bootstrap JSON for testing or development:

```javascript
import fs from 'fs';

// Get bootstrap data:
// curl -H "Authorization: <your-env-secret>" \
//   https://app-eval.featbit.co/api/public/sdk/server/latest-all > featbit-bootstrap.json

let data = '';
try {
  data = fs.readFileSync('path_to_the_json_file', 'utf8');
} catch (err) {
  console.error(err);
}

const fbClient = new FbClientBuilder()
    .offline(true)
    .useJsonBootstrapProvider(data)
    .build();
```

### Disable Events Collection

```javascript
const fbClient = new FbClientBuilder()
    .disableEvents(true)  // No automatic event collection
    .build();
```

### Logger Configuration

**Default**: Log level is **none** (no output).

**Use different log level**:

```javascript
const fbClient = new FbClientBuilder()
    .logLevel('debug')
    .build();

// or
const options = {
  logLevel: 'debug'
};

const fbClient = new FbClientBuilder(options).build();
```

**Define custom logger**:

```javascript
import { BasicLogger } from "@featbit/node-server-sdk";

const logger = new BasicLogger({
    level: 'debug',
    destination: console.log
});

const fbClient = new FbClientBuilder()
    .logger(logger)
    .build();
```

**Note**: `logger` option has higher priority than `logLevel`.

## Data Synchronization

- **WebSocket** or **Polling** keeps local data synchronized with FeatBit server
- Data stored in **memory** by default
- Changes pushed to SDK in **<100ms average** (WebSocket)
- Auto-reconnects after internet outage

## Best Practices

### Single Client Instance

Create one FbClient instance at application startup and reuse it throughout the application lifetime.

### Graceful Shutdown

Flush pending events before application exits:

```javascript
process.on('SIGTERM', async () => {
  await fbClient.close();
  process.exit(0);
});
```

## Supported Node.js Versions

This SDK should work for recent versions of Node.js. If you encounter issues with a specific version, please [create an issue](https://github.com/featbit/featbit-node-server-sdk/issues/new).

## Getting Support

- **Questions**: [Join our Slack](https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ)
- **Bug Reports / Feature Requests**: [Submit an issue](https://github.com/featbit/featbit-node-server-sdk/issues/new)

## Additional Resources

For complete documentation, examples, and detailed guides, visit:

- **📦 NPM Package**: https://www.npmjs.com/package/@featbit/node-server-sdk
- **📖 GitHub Repository**: https://github.com/featbit/featbit-node-server-sdk
- **💡 Code Examples**: https://github.com/featbit/featbit-samples
- ** Getting Started**: https://docs.featbit.co/getting-started/connect-an-sdk#node.js
- **❓ SDK FAQ**: https://docs.featbit.co/sdk/faq
