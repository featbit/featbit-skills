---
name: featbit-node-server-sdk
description: Expert guidance for integrating FeatBit Node.js Server-Side SDK in backend applications. Use for Express, Koa, NestJS, or any Node.js server application. Not for browser/client-side use.
appliesTo:
  - "**/server.js"
  - "**/app.js"
  - "**/index.js"
  - "**/*.server.js"
  - "**/api/**/*.js"
  - "**/backend/**/*.js"
---

# FeatBit Node.js Server SDK Expert

Expert knowledge for integrating FeatBit Server-Side SDK in Node.js applications.

**Source**: https://github.com/featbit/featbit-node-server-sdk

⚠️ **Important**: This is a **server-side SDK** for multi-user systems (web servers, APIs). Not for browser use - use `@featbit/js-client-sdk` for browsers.

## Data Synchronization

- **WebSocket** or **Polling** for real-time sync with FeatBit server
- Data stored in memory by default
- Changes pushed to SDK in <100ms average
- Auto-reconnects after internet outage

## Installation

```bash
npm install --save @featbit/node-server-sdk
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { FbClientBuilder, UserBuilder } from "@featbit/node-server-sdk";

// Setup SDK options
const fbClient = new FbClientBuilder()
    .sdkKey("your_sdk_key")
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri("https://app-eval.featbit.co")
    .build();

(async () => {
  // Wait for initialization
  try {
    await fbClient.waitForInitialization();
  } catch(err) {
    console.log("Failed to initialize SDK:", err);
  }

  const flagKey = "game-runner";
  
  // Create user
  const user = new UserBuilder('user-id-123')
    .name('John Doe')
    .custom('role', 'admin')
    .custom('age', 25)
    .build();

  // Evaluate boolean flag
  const boolValue = await fbClient.boolVariation(flagKey, user, false);
  console.log(`Flag '${flagKey}' returns ${boolValue} for user ${user.Key}`);

  // Evaluate with detail
  const detail = await fbClient.boolVariationDetail(flagKey, user, false);
  console.log(`Value: ${detail.value}, Reason: ${detail.reason}, Kind: ${detail.kind}`);

  // Flush events before exit
  await fbClient.close();
})();
```

## User Builder

```javascript
import { UserBuilder } from "@featbit/node-server-sdk";

// Basic user
const user = new UserBuilder('user-key-123')
    .name('Jane Smith')
    .build();

// User with custom properties
const userWithProps = new UserBuilder('user-key-456')
    .name('Bob Wilson')
    .custom('role', 'admin')
    .custom('country', 'US')
    .custom('subscription', 'premium')
    .custom('age', 30)
    .build();
```

## Express.js Integration

```javascript
const express = require('express');
const { FbClientBuilder, UserBuilder } = require('@featbit/node-server-sdk');

const app = express();

// Initialize FeatBit client
const fbClient = new FbClientBuilder()
    .sdkKey(process.env.FEATBIT_SDK_KEY)
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .build();

// Middleware to create user context
app.use(async (req, res, next) => {
  const userId = req.user?.id || req.sessionID || 'anonymous';
  req.fbUser = new UserBuilder(userId)
    .name(req.user?.name)
    .custom('role', req.user?.role)
    .custom('subscription', req.user?.subscription)
    .build();
  next();
});

// Route with feature flag
app.get('/api/feature', async (req, res) => {
  const isEnabled = await fbClient.boolVariation(
    'new-api-feature', 
    req.fbUser, 
    false
  );
  
  if (!isEnabled) {
    return res.status(403).json({ error: 'Feature not available' });
  }
  
  res.json({ data: 'feature data' });
});

// Gradual rollout example
app.get('/api/checkout', async (req, res) => {
  const useNewCheckout = await fbClient.boolVariation(
    'new-checkout-flow',
    req.fbUser,
    false
  );
  
  if (useNewCheckout) {
    return res.json(await processNewCheckout(req));
  } else {
    return res.json(await processOldCheckout(req));
  }
});

// A/B testing example
app.post('/api/purchase', async (req, res) => {
  const variant = await fbClient.variation(
    'pricing-strategy',
    req.fbUser,
    'standard'
  );
  
  const price = calculatePrice(req.body, variant);
  const result = await processPurchase(price);
  
  if (result.success) {
    // Track conversion
    await fbClient.track(req.fbUser, 'purchase-completed', price);
  }
  
  res.json(result);
});

// Start server after SDK initialization
fbClient.waitForInitialization().then(() => {
  app.listen(3000, () => {
    console.log('Server started on port 3000');
  });
}).catch(err => {
  console.error('Failed to initialize FeatBit:', err);
  process.exit(1);
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  await fbClient.close();
  process.exit(0);
});
```

## NestJS Integration

```typescript
// featbit.module.ts
import { Module, Global } from '@nestjs/common';
import { FbClientBuilder } from '@featbit/node-server-sdk';
import { ConfigService } from '@nestjs/config';

@Global()
@Module({
  providers: [
    {
      provide: 'FB_CLIENT',
      useFactory: async (configService: ConfigService) => {
        const client = new FbClientBuilder()
          .sdkKey(configService.get('FEATBIT_SDK_KEY'))
          .streamingUri(configService.get('FEATBIT_STREAMING_URI'))
          .eventsUri(configService.get('FEATBIT_EVENTS_URI'))
          .build();
        
        await client.waitForInitialization();
        return client;
      },
      inject: [ConfigService],
    },
  ],
  exports: ['FB_CLIENT'],
})
export class FeatBitModule {}

// feature.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { UserBuilder } from '@featbit/node-server-sdk';

@Injectable()
export class FeatureService {
  constructor(@Inject('FB_CLIENT') private fbClient: any) {}

  async checkFeature(userId: string, flagKey: string): Promise<boolean> {
    const user = new UserBuilder(userId).build();
    return await this.fbClient.boolVariation(flagKey, user, false);
  }
  
  async getFeatureVariant(userId: string, flagKey: string, defaultValue: string): Promise<string> {
    const user = new UserBuilder(userId).build();
    return await this.fbClient.variation(flagKey, user, defaultValue);
  }
}

// feature.controller.ts
import { Controller, Get, Req } from '@nestjs/common';
import { FeatureService } from './feature.service';

@Controller('api')
export class FeatureController {
  constructor(private featureService: FeatureService) {}

  @Get('check-feature')
  async checkFeature(@Req() req) {
    const userId = req.user?.id || 'anonymous';
    const isEnabled = await this.featureService.checkFeature(
      userId,
      'new-feature'
    );
    
    return { enabled: isEnabled };
  }
}
```

## Flag Evaluation

### Boolean Variation

```javascript
const isEnabled = await fbClient.boolVariation('flag-key', user, false);
```

### String Variation

```javascript
const theme = await fbClient.variation('theme', user, 'default');
```

### Number Variation

```javascript
const maxRetries = await fbClient.variation('max-retries', user, 3);
```

### JSON Variation

```javascript
const config = await fbClient.variation('config', user, '{}');
const configObj = JSON.parse(config);
```

### With Evaluation Detail

```javascript
const detail = await fbClient.boolVariationDetail('flag-key', user, false);
console.log({
  value: detail.value,
  reason: detail.reason,
  kind: detail.kind
});
```

### Get All Flags

```javascript
const allFlags = await fbClient.getAllLatestFlagsVariations(user);
console.log(allFlags);
```

## Event Tracking

```javascript
// Simple event
await fbClient.track(user, 'button-clicked');

// Event with numeric value
await fbClient.track(user, 'purchase-completed', 99.99);

// For A/B testing
await fbClient.track(user, 'conversion-event');

// Flush events manually
await fbClient.flush();
```

## Configuration Options

```javascript
const fbClient = new FbClientBuilder()
    // Required
    .sdkKey("your_sdk_key")
    
    // Streaming (recommended)
    .streamingUri('wss://app-eval.featbit.co')
    
    // Events
    .eventsUri('https://app-eval.featbit.co')
    
    // Optional settings
    .startWaitTime(5000)           // Wait time for initialization in ms
    .reconnectInterval(15000)      // Reconnect interval in ms
    .offline(false)                // Set to true for offline mode
    
    .build();
```

## Using with TypeScript

```typescript
import { FbClientBuilder, UserBuilder, IFbClient, IUser } from '@featbit/node-server-sdk';

const fbClient: IFbClient = new FbClientBuilder()
    .sdkKey(process.env.FEATBIT_SDK_KEY!)
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .build();

const user: IUser = new UserBuilder('user-123')
    .name('John Doe')
    .custom('role', 'admin')
    .build();

const isEnabled: boolean = await fbClient.boolVariation('feature-key', user, false);
```

## Koa Integration

```javascript
const Koa = require('koa');
const Router = require('@koa/router');
const { FbClientBuilder, UserBuilder } = require('@featbit/node-server-sdk');

const app = new Koa();
const router = new Router();

const fbClient = new FbClientBuilder()
    .sdkKey(process.env.FEATBIT_SDK_KEY)
    .streamingUri('wss://app-eval.featbit.co')
    .eventsUri('https://app-eval.featbit.co')
    .build();

// Middleware
app.use(async (ctx, next) => {
  const userId = ctx.state.user?.id || 'anonymous';
  ctx.fbUser = new UserBuilder(userId)
    .name(ctx.state.user?.name)
    .build();
  await next();
});

// Route
router.get('/api/feature', async (ctx) => {
  const isEnabled = await fbClient.boolVariation(
    'new-feature',
    ctx.fbUser,
    false
  );
  
  ctx.body = { enabled: isEnabled };
});

app.use(router.routes());

fbClient.waitForInitialization().then(() => {
  app.listen(3000);
});
```

## Best Practices

### 1. Single Client Instance

```javascript
// ✅ Good: Create once at app startup
const fbClient = createFbClient();

// ❌ Bad: Creating per request
app.get('/api/feature', async (req, res) => {
  const client = createFbClient(); // Don't do this!
});
```

### 2. Wait for Initialization

```javascript
// ✅ Good: Wait before starting server
await fbClient.waitForInitialization();
app.listen(3000);

// ❌ Bad: Start immediately
const fbClient = new FbClientBuilder().build();
app.listen(3000); // Flags may return defaults
```

### 3. Graceful Shutdown

```javascript
process.on('SIGTERM', async () => {
  console.log('Shutting down...');
  await fbClient.close();  // Flush events
  await server.close();     // Close server
  process.exit(0);
});
```

### 4. Error Handling

```javascript
try {
  const isEnabled = await fbClient.boolVariation('flag-key', user, false);
  // Use feature
} catch (error) {
  console.error('Flag evaluation failed:', error);
  // Fall back to default
  const isEnabled = false;
}
```

### 5. User Context

```javascript
// ✅ Good: Use stable identifiers
const userId = req.user.id; // From auth system
const user = new UserBuilder(userId).build();

// ✅ Good: Add relevant properties
const user = new UserBuilder(userId)
    .name(req.user.name)
    .custom('role', req.user.role)
    .custom('plan', req.user.subscription)
    .build();

// ❌ Bad: Random IDs
const user = new UserBuilder(Math.random().toString()).build();
```

## Common Use Cases

### Progressive Rollout

```javascript
app.get('/api/new-feature', async (req, res) => {
  const useNewVersion = await fbClient.boolVariation(
    'new-api-version',
    req.fbUser,
    false
  );
  
  if (useNewVersion) {
    return res.json(await newVersionHandler(req));
  } else {
    return res.json(await oldVersionHandler(req));
  }
});
```

### A/B Testing

```javascript
app.get('/api/recommendations', async (req, res) => {
  const algorithm = await fbClient.variation(
    'recommendation-algorithm',
    req.fbUser,
    'default'
  );
  
  const recommendations = await getRecommendations(req.user, algorithm);
  
  // Track which algorithm was used
  await fbClient.track(req.fbUser, `algorithm-${algorithm}`);
  
  res.json(recommendations);
});
```

### Remote Configuration

```javascript
const config = await fbClient.variation('api-config', user, JSON.stringify({
  timeout: 30000,
  retries: 3,
  rateLimit: 1000
}));

const { timeout, retries, rateLimit } = JSON.parse(config);

// Use configuration
axios.defaults.timeout = timeout;
```

### Feature Toggle

```javascript
const maintenanceMode = await fbClient.boolVariation(
  'maintenance-mode',
  systemUser,
  false
);

if (maintenanceMode) {
  return res.status(503).json({ 
    error: 'Service temporarily unavailable' 
  });
}
```

## Troubleshooting

### SDK Not Initializing

```javascript
fbClient.waitForInitialization()
  .then(() => console.log('SDK initialized'))
  .catch(err => {
    // Check:
    // 1. SDK key is correct
    // 2. Network connectivity
    // 3. Firewall rules for WebSocket
    console.error('Initialization failed:', err);
  });
```

### Connection Issues

```javascript
// Increase timeout
const fbClient = new FbClientBuilder()
    .startWaitTime(10000)  // 10 seconds
    .build();
```

### Events Not Sent

```javascript
// Manually flush before exit
process.on('exit', async () => {
  await fbClient.flush();
});
```

## Performance Optimization

### Caching User Context

```javascript
const userCache = new Map();

function getCachedUser(userId) {
  if (!userCache.has(userId)) {
    const user = new UserBuilder(userId).build();
    userCache.set(userId, user);
  }
  return userCache.get(userId);
}
```

### Batch Operations

```javascript
// Evaluate multiple flags efficiently
async function getFeatureConfig(user) {
  const [feature1, feature2, feature3] = await Promise.all([
    fbClient.boolVariation('feature-1', user, false),
    fbClient.boolVariation('feature-2', user, false),
    fbClient.variation('feature-3', user, 'default')
  ]);
  
  return { feature1, feature2, feature3 };
}
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-node-server-sdk
- **NPM**: https://www.npmjs.com/package/@featbit/node-server-sdk
- **Examples**: https://github.com/featbit/featbit-samples
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/node-js
