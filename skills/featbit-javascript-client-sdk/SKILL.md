---
name: featbit-javascript-client-sdk
description: Expert guidance for integrating FeatBit JavaScript Client SDK in web applications (browser-only). Use when users work with vanilla JavaScript in browsers, need client-side feature flags, or ask about JS client SDK integration.
appliesTo:
  - "**/*.html"
  - "**/*.js"
  - "**/index.js"
  - "**/app.js"
---

# FeatBit JavaScript Client SDK Expert

Expert knowledge for integrating FeatBit JavaScript Client SDK in web browsers.

**Source**: https://github.com/featbit/featbit-js-client-sdk

⚠️ **Important**: This is a **client-side SDK** for single-user browser environments only. Not suitable for Node.js server applications (use `@featbit/node-server-sdk` instead).

## Installation

```bash
npm install --save @featbit/js-client-sdk
```

## Prerequisites

Before using, obtain:
- **Environment Secret (SDK Key)**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { FbClientBuilder, UserBuilder } from "@featbit/js-client-sdk";

// Create user
const bob = new UserBuilder('a-unique-key-of-user')
    .name('bob')
    .custom('age', '18')
    .custom('country', 'FR')
    .build();

// Setup SDK client with WebSocket streaming
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .user(bob)
    .build();

(async () => {
  // Wait for initialization
  try {
    await fbClient.waitForInitialization();
  } catch(err) {
    console.log("Failed to initialize SDK:", err);
  }

  // Evaluate feature flag
  const flagKey = "game-runner";
  const isEnabled = await fbClient.boolVariation(flagKey, false);
  
  console.log(`Feature ${flagKey}: ${isEnabled}`);
  
  // Switch user
  const alice = new UserBuilder('another-unique-key-of-user')
      .name('alice')
      .custom('country', 'UK')
      .custom('age', 36)
      .build();
  
  await fbClient.identify(alice);
})();
```

## Building User Context

```javascript
import { UserBuilder } from "@featbit/js-client-sdk";

// Basic user
const user = new UserBuilder('user-key-123')
    .name('John Doe')
    .build();

// User with custom properties
const userWithProps = new UserBuilder('user-key-456')
    .name('Jane Smith')
    .custom('age', 25)
    .custom('country', 'US')
    .custom('subscription', 'premium')
    .custom('role', 'admin')
    .build();
```

## Client Configuration

### Using WebSocket Streaming (Recommended)

```javascript
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .user(user)
    .build();
```

### Using HTTP Polling

```javascript
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .eventsUri("https://app-eval.featbit.co")
    .pollingUri("https://app-eval.featbit.co")
    .pollingInterval(3000) // 3 seconds
    .user(user)
    .build();
```

## Flag Evaluation

### Boolean Flags

```javascript
const isEnabled = await fbClient.boolVariation('feature-key', false);
if (isEnabled) {
  // Feature is enabled
}
```

### String Flags

```javascript
const theme = await fbClient.variation('theme-key', 'default');
console.log(`Current theme: ${theme}`);
```

### Numeric Flags

```javascript
const maxItems = await fbClient.variation('max-items', 10);
```

### JSON Flags

```javascript
const config = await fbClient.variation('config-key', '{}');
const configObject = JSON.parse(config);
```

### With Evaluation Details

```javascript
const detail = await fbClient.boolVariationDetail('feature-key', false);
console.log(`Value: ${detail.value}`);
console.log(`Reason: ${detail.reason}`);
console.log(`Kind: ${detail.kind}`);
```

## User Identification & Switching

### Identify New User

```javascript
const newUser = new UserBuilder('new-user-key')
    .name('New User')
    .custom('role', 'viewer')
    .build();

await fbClient.identify(newUser);
```

### Anonymous Users

```javascript
// Use session ID or generate unique ID
const sessionId = sessionStorage.getItem('sessionId') || crypto.randomUUID();
const anonymousUser = new UserBuilder(sessionId)
    .custom('anonymous', true)
    .build();
```

## Event Tracking

### Track Custom Events

```javascript
// Simple event
fbClient.trackMetric('button-clicked');

// Event with numeric value
fbClient.trackMetric('purchase-completed', 99.99);

// For A/B testing
fbClient.trackMetric('conversion-event');
```

### Flush Events Manually

```javascript
await fbClient.flush();
```

## Real-Time Flag Updates

### Listen for Flag Changes

```javascript
fbClient.on('ff_update', (changes) => {
  console.log('Flags updated:', changes);
  
  // Re-evaluate flags
  const isEnabled = fbClient.variation('feature-key', false);
  updateUI(isEnabled);
});
```

### Listen for Specific Flag

```javascript
fbClient.on('ff_update', (changes) => {
  if (changes.includes('feature-key')) {
    const newValue = fbClient.variation('feature-key', false);
    console.log(`Feature flag updated: ${newValue}`);
  }
});
```

## Complete Web App Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>FeatBit Demo</title>
</head>
<body>
  <div id="app">
    <h1>Feature Flag Demo</h1>
    <div id="feature-content"></div>
    <button id="switch-user">Switch User</button>
  </div>

  <script type="module">
    import { FbClientBuilder, UserBuilder } from '@featbit/js-client-sdk';

    const users = [
      new UserBuilder('user-1').name('Alice').custom('role', 'admin').build(),
      new UserBuilder('user-2').name('Bob').custom('role', 'user').build()
    ];

    let currentUserIndex = 0;

    const fbClient = new FbClientBuilder()
      .sdkKey('your_sdk_key')
      .streamingUri('wss://app-eval.featbit.co')
      .eventsUri('https://app-eval.featbit.co')
      .user(users[0])
      .build();

    async function init() {
      try {
        await fbClient.waitForInitialization();
        updateFeatureDisplay();
        
        fbClient.on('ff_update', () => {
          updateFeatureDisplay();
        });
      } catch (err) {
        console.error('Failed to initialize:', err);
      }
    }

    function updateFeatureDisplay() {
      const isEnabled = fbClient.variation('new-feature', false);
      const content = document.getElementById('feature-content');
      
      if (isEnabled) {
        content.innerHTML = '<p>✅ New Feature Enabled</p>';
      } else {
        content.innerHTML = '<p>❌ New Feature Disabled</p>';
      }
    }

    document.getElementById('switch-user').onclick = async () => {
      currentUserIndex = (currentUserIndex + 1) % users.length;
      await fbClient.identify(users[currentUserIndex]);
      updateFeatureDisplay();
    };

    init();
  </script>
</body>
</html>
```

## Configuration Options

```javascript
const fbClient = new FbClientBuilder()
    // Required
    .sdkKey("your_sdk_key")
    .user(user)
    
    // Streaming (recommended)
    .streamingUri('wss://app-eval.featbit.co')
    
    // Or Polling
    .pollingUri('https://app-eval.featbit.co')
    .pollingInterval(3000)
    
    // Events
    .eventsUri('https://app-eval.featbit.co')
    
    // Optional
    .startWaitTime(3000) // Wait time for initialization in ms
    .enableDataSync(true) // Enable data synchronization
    
    .build();
```

## Best Practices

### 1. Single Client Instance

```javascript
// ✅ Good: Create once, reuse
const fbClient = createFbClient();

// ❌ Bad: Creating multiple instances
function checkFeature() {
  const client = createFbClient(); // Don't do this
}
```

### 2. Wait for Initialization

```javascript
// ✅ Good: Wait for initialization
await fbClient.waitForInitialization();
const value = fbClient.variation('key', default);

// ❌ Bad: Don't wait
const value = fbClient.variation('key', default); // May return default
```

### 3. Error Handling

```javascript
try {
  await fbClient.waitForInitialization();
  const isEnabled = fbClient.variation('feature', false);
  // Use feature
} catch (error) {
  console.error('SDK initialization failed:', error);
  // Fall back to default behavior
  const isEnabled = false;
}
```

### 4. User Context Best Practices

```javascript
// ✅ Good: Use stable user IDs
const userId = getUserId(); // From auth system
const user = new UserBuilder(userId).build();

// ✅ Good: Add relevant properties for targeting
const user = new UserBuilder(userId)
    .name(userName)
    .custom('subscription', subscription)
    .custom('country', country)
    .build();

// ❌ Bad: Random IDs (user won't get consistent experience)
const user = new UserBuilder(Math.random().toString()).build();
```

### 5. Clean Up

```javascript
// Close client when page unloads
window.addEventListener('beforeunload', async () => {
  await fbClient.close();
});
```

## Common Use Cases

### Progressive Rollout

```javascript
// Enable feature for percentage of users
const isEnabled = await fbClient.boolVariation('new-checkout', false);

if (isEnabled) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

### A/B Testing

```javascript
const variant = await fbClient.variation('landing-page', 'A');

switch(variant) {
  case 'A':
    renderVariantA();
    break;
  case 'B':
    renderVariantB();
    break;
}

// Track conversion
if (userConverted) {
  fbClient.trackMetric('landing-page-conversion');
}
```

### Feature Toggle

```javascript
const showBetaFeature = await fbClient.boolVariation('beta-feature', false);

if (showBetaFeature) {
  document.getElementById('beta-section').style.display = 'block';
}
```

### Remote Configuration

```javascript
const config = await fbClient.variation('app-config', '{}');
const { apiUrl, timeout, retries } = JSON.parse(config);

// Use configuration
fetch(apiUrl, { timeout });
```

## Troubleshooting

### SDK Not Initializing

```javascript
try {
  await fbClient.waitForInitialization();
} catch (err) {
  // Check:
  // 1. SDK key is correct
  // 2. Network connectivity
  // 3. CORS settings
  // 4. WebSocket/HTTP connection
  console.error('Initialization error:', err);
}
```

### Flags Not Updating

```javascript
// Check event listeners
fbClient.on('ff_update', (changes) => {
  console.log('Update received:', changes);
});

// Check WebSocket connection status
fbClient.on('ready', () => console.log('Connected'));
fbClient.on('error', (err) => console.error('Connection error:', err));
```

### CORS Issues

Ensure FeatBit server allows your origin. Contact your FeatBit administrator to configure CORS settings.

## Migration from LaunchDarkly

```javascript
// LaunchDarkly
const ldClient = LDClient.initialize('sdk-key', user);
const flag = ldClient.variation('flag-key', false);

// FeatBit equivalent
const fbClient = new FbClientBuilder()
    .sdkKey('sdk-key')
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .user(user)
    .build();

await fbClient.waitForInitialization();
const flag = fbClient.variation('flag-key', false);
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-js-client-sdk
- **NPM**: https://www.npmjs.com/package/@featbit/js-client-sdk
- **Documentation**: https://docs.featbit.co/sdk-docs/client-side-sdks/javascript
- **Examples**: https://github.com/featbit/featbit-js-client-sdk/tree/main/examples
- **FAQ**: https://docs.featbit.co/sdk/faq
