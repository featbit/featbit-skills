---
name: featbit-openfeature-node-server
description: Expert guidance for using FeatBit OpenFeature Provider for Node.js server-side applications. Use when implementing OpenFeature standard in Node.js backends with FeatBit.
appliesTo:
  - "**/server.js"
  - "**/app.js"
  - "**/index.js"
  - "**/*.server.js"
  - "**/api/**/*.js"
  - "**/backend/**/*.js"
---

# FeatBit OpenFeature Provider for Node.js Expert

Expert knowledge for integrating FeatBit with OpenFeature in Node.js server applications.

**Source**: https://github.com/featbit/openfeature-provider-node-server

⚠️ **Important**: This is a **server-side OpenFeature provider** for multi-user systems. Not for browser use.

## About OpenFeature

OpenFeature is an open standard for feature flag management. It provides a vendor-agnostic API for feature flagging, allowing you to easily switch between different feature flag providers without changing your code.

## Supported Node Versions

Compatible with Node.js versions 16 and above.

## Installation

```bash
npm install @openfeature/server-sdk
npm install @featbit/node-server-sdk
npm install @featbit/openfeature-provider-node-server
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { OpenFeature, ProviderEvents } from '@openfeature/server-sdk';
import { FbProvider } from '@featbit/openfeature-provider-node-server';

const provider = new FbProvider({
    sdkKey: '<your-sdk-key>',
    streamingUri: '<your-streaming-uri>',
    eventsUri: '<your-events-uri>'
});

OpenFeature.setProvider(provider);

// Wait for provider to be ready
OpenFeature.addHandler(ProviderEvents.Ready, (eventDetails) => {
    console.log('Provider is ready');
});

// Get client and evaluate flags
const client = OpenFeature.getClient();
const value = await client.getBooleanValue('my-flag', false, {
    targetingKey: 'user-123'
});

console.log('Flag value:', value);
```

## Express.js Integration

```javascript
import express from 'express';
import { OpenFeature, ProviderEvents } from '@openfeature/server-sdk';
import { FbProvider } from '@featbit/openfeature-provider-node-server';

const app = express();

// Initialize FeatBit provider
const provider = new FbProvider({
    sdkKey: process.env.FEATBIT_SDK_KEY,
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co'
});

OpenFeature.setProvider(provider);

// Wait for initialization
let isReady = false;
OpenFeature.addHandler(ProviderEvents.Ready, () => {
    isReady = true;
    console.log('FeatBit provider is ready');
});

// Middleware to create evaluation context
app.use((req, res, next) => {
    const userId = req.user?.id || req.sessionID || 'anonymous';
    req.evaluationContext = {
        targetingKey: userId,
        name: req.user?.name,
        role: req.user?.role,
        subscription: req.user?.subscription
    };
    next();
});

// Route with feature flag
app.get('/api/feature', async (req, res) => {
    const client = OpenFeature.getClient();
    
    const isEnabled = await client.getBooleanValue(
        'new-feature',
        false,
        req.evaluationContext
    );
    
    if (!isEnabled) {
        return res.status(403).json({ error: 'Feature not available' });
    }
    
    res.json({ data: 'feature data' });
});

// A/B testing example
app.post('/api/purchase', async (req, res) => {
    const client = OpenFeature.getClient();
    
    const variant = await client.getStringValue(
        'pricing-strategy',
        'standard',
        req.evaluationContext
    );
    
    const price = calculatePrice(req.body, variant);
    const result = await processPurchase(price);
    
    res.json({ ...result, variant, price });
});

// Start server after provider is ready
const startServer = () => {
    if (isReady) {
        app.listen(3000, () => {
            console.log('Server started on port 3000');
        });
    } else {
        setTimeout(startServer, 100);
    }
};

startServer();
```

## Evaluation Context

The evaluation context is used to pass user information for flag evaluation:

```javascript
const client = OpenFeature.getClient();

const context = {
    targetingKey: 'user-123',      // Required: unique user identifier
    name: 'John Doe',              // Optional: user name
    email: 'john@example.com',     // Optional: custom property
    role: 'admin',                 // Optional: custom property
    subscription: 'premium'        // Optional: custom property
};

const value = await client.getBooleanValue('feature-flag', false, context);
```

## Flag Evaluation

### Boolean Flags

```javascript
const isEnabled = await client.getBooleanValue('flag-key', false, context);
```

### String Flags

```javascript
const theme = await client.getStringValue('theme', 'default', context);
```

### Number Flags

```javascript
const maxRetries = await client.getNumberValue('max-retries', 3, context);
```

### Object Flags

```javascript
const config = await client.getObjectValue('config', {}, context);
```

### Evaluation Details

```javascript
const details = await client.getBooleanDetails('flag-key', false, context);

console.log('Value:', details.value);
console.log('Reason:', details.reason);
console.log('Variant:', details.variant);
console.log('Flag key:', details.flagKey);
```

## Event Handling

### Provider Ready Event

```javascript
OpenFeature.addHandler(ProviderEvents.Ready, (eventDetails) => {
    console.log('Provider is ready');
});
```

### Configuration Changed Event

```javascript
// Listen for flag changes
OpenFeature.addHandler(ProviderEvents.ConfigurationChanged, async (eventDetails) => {
    console.log('Flags changed:', eventDetails.flagsChanged);
    
    // Re-evaluate flags
    const client = OpenFeature.getClient();
    const value = await client.getBooleanValue(
        eventDetails.flagsChanged[0],
        false,
        { targetingKey: 'my-key' }
    );
    
    console.log('New value:', value);
});
```

### Error Event

```javascript
OpenFeature.addHandler(ProviderEvents.Error, (eventDetails) => {
    console.error('Provider error:', eventDetails);
});
```

## Access Underlying FBClient

If you need access to FeatBit-specific features:

```javascript
import { FbProvider } from '@featbit/openfeature-provider-node-server';

const provider = new FbProvider({
    sdkKey: 'your-sdk-key',
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co'
});

OpenFeature.setProvider(provider);

// Access the underlying FeatBit client
const fbClient = provider.getClient();

// Use FeatBit-specific methods
fbClient.track(user, 'custom-event');
fbClient.flush();
```

## NestJS Integration

```typescript
// featbit.module.ts
import { Module, Global, OnModuleInit } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { OpenFeature, ProviderEvents } from '@openfeature/server-sdk';
import { FbProvider } from '@featbit/openfeature-provider-node-server';

@Global()
@Module({})
export class FeatBitModule implements OnModuleInit {
    constructor(private configService: ConfigService) {}
    
    async onModuleInit() {
        const provider = new FbProvider({
            sdkKey: this.configService.get('FEATBIT_SDK_KEY'),
            streamingUri: this.configService.get('FEATBIT_STREAMING_URI'),
            eventsUri: this.configService.get('FEATBIT_EVENTS_URI')
        });
        
        OpenFeature.setProvider(provider);
        
        return new Promise((resolve) => {
            OpenFeature.addHandler(ProviderEvents.Ready, () => {
                console.log('FeatBit provider is ready');
                resolve(undefined);
            });
        });
    }
}

// feature.service.ts
import { Injectable } from '@nestjs/common';
import { OpenFeature } from '@openfeature/server-sdk';

@Injectable()
export class FeatureService {
    private client = OpenFeature.getClient();
    
    async isFeatureEnabled(userId: string, flagKey: string): Promise<boolean> {
        return await this.client.getBooleanValue(
            flagKey,
            false,
            { targetingKey: userId }
        );
    }
    
    async getFeatureValue<T>(
        userId: string,
        flagKey: string,
        defaultValue: T
    ): Promise<T> {
        if (typeof defaultValue === 'boolean') {
            return await this.client.getBooleanValue(
                flagKey,
                defaultValue,
                { targetingKey: userId }
            ) as T;
        } else if (typeof defaultValue === 'string') {
            return await this.client.getStringValue(
                flagKey,
                defaultValue as string,
                { targetingKey: userId }
            ) as T;
        } else if (typeof defaultValue === 'number') {
            return await this.client.getNumberValue(
                flagKey,
                defaultValue as number,
                { targetingKey: userId }
            ) as T;
        } else {
            return await this.client.getObjectValue(
                flagKey,
                defaultValue as object,
                { targetingKey: userId }
            ) as T;
        }
    }
}
```

## Graceful Shutdown

```javascript
// Flush events before exit
process.on('SIGTERM', async () => {
    console.log('Shutting down...');
    await OpenFeature.close();  // This will flush events
    process.exit(0);
});

// Or in async function
async function shutdown() {
    await OpenFeature.close();
}
```

## Best Practices

### 1. Wait for Provider Ready

```javascript
// ✅ Good: Wait for ready state
await new Promise((resolve) => {
    OpenFeature.addHandler(ProviderEvents.Ready, resolve);
});

// ❌ Bad: Use immediately
const client = OpenFeature.getClient();
const value = await client.getBooleanValue(...); // May return defaults
```

### 2. Reuse Client Instance

```javascript
// ✅ Good: Reuse client
const client = OpenFeature.getClient();

app.get('/api/feature1', async (req, res) => {
    const value = await client.getBooleanValue(...);
});

app.get('/api/feature2', async (req, res) => {
    const value = await client.getBooleanValue(...);
});

// ❌ Bad: Create new client per request
app.get('/api/feature', async (req, res) => {
    const client = OpenFeature.getClient(); // Unnecessary
});
```

### 3. Handle Errors

```javascript
try {
    const value = await client.getBooleanValue('flag-key', false, context);
    // Use feature
} catch (error) {
    console.error('Flag evaluation failed:', error);
    // Fall back to default
    const value = false;
}
```

### 4. Named Clients

```javascript
// Use named clients for different domains
const authClient = OpenFeature.getClient('auth');
const paymentClient = OpenFeature.getClient('payment');

const authEnabled = await authClient.getBooleanValue('new-auth', false, context);
const paymentEnabled = await paymentClient.getBooleanValue('new-payment', false, context);
```

## Benefits of OpenFeature

### Vendor Neutrality

```javascript
// Easy to switch providers without code changes
// Just change the provider initialization

// FeatBit
const provider = new FbProvider({...});

// Can easily switch to another provider
// const provider = new AnotherProvider({...});

OpenFeature.setProvider(provider);

// Application code remains the same
const value = await client.getBooleanValue('flag', false, context);
```

### Standardized API

```javascript
// Consistent API across all providers
// - getBooleanValue
// - getStringValue
// - getNumberValue
// - getObjectValue
// - getBooleanDetails (with evaluation reason)
```

### Hooks Support

```javascript
// Add hooks for logging, metrics, etc.
OpenFeature.addHooks({
    before: async (context, hints) => {
        console.log('Evaluating flag:', context.flagKey);
    },
    after: async (context, details, hints) => {
        console.log('Flag result:', details.value);
    },
    error: async (context, error, hints) => {
        console.error('Flag evaluation error:', error);
    }
});
```

## Troubleshooting

### Provider Not Initializing

```javascript
OpenFeature.addHandler(ProviderEvents.Error, (eventDetails) => {
    console.error('Provider error:', eventDetails);
    // Check:
    // 1. SDK key is correct
    // 2. Network connectivity
    // 3. Firewall rules for WebSocket
});
```

### Evaluation Returning Defaults

```javascript
const details = await client.getBooleanDetails('flag-key', false, context);
console.log('Reason:', details.reason);
// Reasons:
// - TARGETING_MATCH: Flag evaluated successfully
// - DEFAULT: Using default value
// - ERROR: Evaluation failed
// - DISABLED: Flag is disabled
```

## Additional Resources

- **GitHub**: https://github.com/featbit/openfeature-provider-node-server
- **OpenFeature**: https://openfeature.dev/
- **OpenFeature Node.js SDK**: https://openfeature.dev/docs/reference/technologies/server/javascript
- **FeatBit Node.js SDK**: https://github.com/featbit/featbit-node-server-sdk
