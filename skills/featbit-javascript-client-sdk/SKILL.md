---
name: featbit-javascript-client-sdk
description: Integrates FeatBit JavaScript Client SDK in browser applications. Use when working with client-side feature flags, browser-based feature toggles, or when user asks about "JavaScript SDK", "JS client SDK", "browser feature flags".
appliesTo:
  - "**/*.html"
  - "**/*.js"
  - "**/index.js"
  - "**/app.js"
---

# FeatBit JavaScript Client SDK

Integration guide for FeatBit JavaScript Client SDK in web browsers.

> **📖 Complete Documentation**: [GitHub Repository](https://github.com/featbit/featbit-js-client-sdk)  
> If this skill doesn't cover your needs, refer to the full SDK documentation above.

⚠️ **Client-Side SDK Only**: For single-user browser environments. Use `@featbit/node-server-sdk` for Node.js server applications.

## Installation & Setup

```bash
npm install --save @featbit/js-client-sdk
```

**Prerequisites**:
- SDK Key: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- SDK URLs: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { FbClientBuilder, UserBuilder } from "@featbit/js-client-sdk";

// 1. Build user context
const user = new UserBuilder('user-unique-key')
    .name('bob')
    .custom('age', '18')
    .custom('country', 'FR')
    .build();

// 2. Initialize client (WebSocket streaming)
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('ws://localhost:5100')
    .eventsUri("http://localhost:5100")
    .user(user)
    .build();

// 3. Wait for initialization & evaluate
(async () => {
  try {
    await fbClient.waitForInitialization();
    const isEnabled = await fbClient.boolVariation("game-runner", false);
    console.log(`Feature enabled: ${isEnabled}`);
  } catch(err) {
    console.error("SDK initialization failed:", err);
  }
})();
```

## Core Concepts

### 1. User Context (IUser)

Build users with `UserBuilder`:

```javascript
const user = new UserBuilder('user-key')  // Required: unique key
    .name('Alice')                        // Optional: display name
    .custom('role', 'admin')              // Optional: custom properties
    .custom('subscription', 'premium')
    .build();
```

**Key Requirements**:
- `key` is mandatory and must uniquely identify each user
- Custom properties are used for targeting rules
- Properties are included in analytics

### 2. Client Configuration

**Option A: WebSocket Streaming (Recommended)**
```javascript
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('ws://localhost:5100')
    .eventsUri("http://localhost:5100")
    .user(user)
    .build();
```

**Option B: HTTP Polling**
```javascript
import { DataSyncModeEnum } from "@featbit/js-client-sdk";

const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .dataSyncMode(DataSyncModeEnum.POLLING)
    .pollingUri('http://localhost:5100')
    .pollingInterval(10000)  // 10 seconds
    .eventsUri("http://localhost:5100")
    .user(user)
    .build();
```

### 3. Flag Evaluation

```javascript
// Boolean flags
const isEnabled = await fbClient.boolVariation('feature-key', false);

// String flags
const theme = await fbClient.variation('theme', 'default');

// With evaluation details
const detail = await fbClient.boolVariationDetail('feature-key', false);
console.log(detail.value, detail.reason, detail.kind);
```

### 4. User Switching

```javascript
const newUser = new UserBuilder('new-user-key')
    .name('Bob')
    .build();

await fbClient.identify(newUser);
```

### 5. Event Tracking (A/B Testing)

```javascript
// Track custom events
fbClient.track('button-clicked');
fbClient.track('purchase-completed', 99.99);  // with numeric value

// Manual flush
await fbClient.flush();
```

### 6. Real-Time Updates

```javascript
// Listen to any flag changes
fbClient.on('update', (flagKeys) => {
  console.log('Flags changed:', flagKeys);
});

// Listen to specific flag
fbClient.on('update:feature-key', (key) => {
  const newValue = fbClient.variation('feature-key', false);
  console.log(`${key} updated:`, newValue);
});

// Ready event
fbClient.on('ready', () => console.log('SDK ready'));
```

## Advanced Features

### Bootstrap (Pre-loaded Flags)

Provide initial flags to avoid waiting for remote fetch:

```javascript
const options = {
  bootstrap: [
    { id: 'flag-key', variation: 'true', variationType: 'boolean' },
    { id: 'theme', variation: 'dark', variationType: 'string' }
  ]
};

const fbClient = new FbClientBuilder(options).build();
```

> Bootstrapped flags are overridden by remote flags once fetched.

### Logging Configuration

**Method 1: Set log level**
```javascript
const fbClient = new FbClientBuilder()
    .logLevel('debug')  // 'debug', 'info', 'warn', 'error', 'none'
    .build();
```

**Method 2: Custom logger**
```javascript
import { BasicLogger } from "@featbit/js-client-sdk";

const logger = new BasicLogger({
    level: 'debug',
    destination: console.log
});

const fbClient = new FbClientBuilder()
    .logger(logger)  // Takes precedence over logLevel
    .build();
```

### Offline Mode

Disable remote calls (use with bootstrap):

```javascript
const fbClient = new FbClientBuilder()
    .offline(true)
    .bootstrap([/* flags */])
    .build();
```

### Disable Event Collection

```javascript
const fbClient = new FbClientBuilder()
    .disableEvents(true)
    .build();
```

## Best Practices

1. **Single Client Instance**: Create once, reuse throughout app lifetime
2. **Wait for Initialization**: Always `await fbClient.waitForInitialization()`
3. **Stable User IDs**: Use IDs from authentication system, not random values
4. **Error Handling**: Wrap initialization in try-catch and provide fallback
5. **Cleanup**: Close client on page unload

```javascript
window.addEventListener('beforeunload', () => fbClient.close());
```

## Common Patterns

**Progressive Rollout**:
```javascript
if (await fbClient.boolVariation('new-checkout', false)) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

**A/B Testing**:
```javascript
const variant = await fbClient.variation('landing-page', 'A');
if (variant === 'B') renderVariantB();
if (userConverted) fbClient.track('conversion');
```

**Remote Configuration**:
```javascript
const config = JSON.parse(await fbClient.variation('app-config', '{}'));
fetch(config.apiUrl, { timeout: config.timeout });
```

## Troubleshooting

**SDK not initializing**: Check SDK key, network, CORS, WebSocket connection  
**Flags not updating**: Verify event listeners and WebSocket status  
**CORS errors**: Configure allowed origins on FeatBit server

## Examples & Resources

- **Full Examples**: [GitHub examples folder](https://github.com/featbit/featbit-js-client-sdk/tree/main/examples)
- **React Integration**: Use [@featbit/react-client-sdk](https://github.com/featbit/featbit-react-client-sdk)
- **Documentation**: https://docs.featbit.co/sdk-docs/client-side-sdks/javascript
- **Getting Support**: [FeatBit Slack](https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ)

---

> 💡 **Version Notice**: For v1/v2 SDK, see [legacy documentation](https://github.com/featbit/featbit-js-client-sdk/tree/v2)
