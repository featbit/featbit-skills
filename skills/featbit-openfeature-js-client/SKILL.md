---
name: featbit-openfeature-js-client
description: Expert guidance for using FeatBit OpenFeature Provider for JavaScript client-side applications. Use when implementing OpenFeature standard in browser/client applications with FeatBit.
appliesTo:
  - "**/*.html"
  - "**/*.js"
  - "**/index.html"
  - "**/app.js"
  - "**/main.js"
  - "**/src/**/*.js"
---

# FeatBit OpenFeature Provider for JavaScript Client Expert

Expert knowledge for integrating FeatBit with OpenFeature in JavaScript client-side applications.

**Source**: https://github.com/featbit/openfeature-provider-js-client

⚠️ **Important**: This is a **client-side OpenFeature provider** for single-user browser applications. For server-side, use the Node.js OpenFeature provider.

## About OpenFeature

OpenFeature is an open standard for feature flag management. It provides a vendor-agnostic API for feature flagging, allowing you to easily switch between different feature flag providers without changing your code.

## Benefits of OpenFeature

### Vendor Neutrality
- Switch between different feature flag providers without changing application code
- Avoid vendor lock-in
- Standardized API across all providers

### Consistent API
- Same methods across all providers
- Predictable behavior
- Easy to learn and use

### Ecosystem
- Shared tools and libraries
- Community-driven standards
- Hooks and middleware support

## Installation

```bash
npm install @openfeature/web-sdk
npm install @featbit/js-client-sdk
npm install @featbit/openfeature-provider-js-client
```

Or using CDN:

```html
<script src="https://unpkg.com/@openfeature/web-sdk"></script>
<script src="https://unpkg.com/@featbit/js-client-sdk"></script>
<script src="https://unpkg.com/@featbit/openfeature-provider-js-client"></script>
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { OpenFeature } from '@openfeature/web-sdk';
import { FbProvider } from '@featbit/openfeature-provider-js-client';

// Initialize provider
const provider = new FbProvider({
    sdkKey: 'your-sdk-key',
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co',
    user: {
        keyId: 'user-123',
        name: 'John Doe',
        customizedProperties: [
            { name: 'role', value: 'user' },
            { name: 'country', value: 'US' }
        ]
    }
});

await OpenFeature.setProviderAndWait(provider);

// Get client and evaluate flags
const client = OpenFeature.getClient();
const isEnabled = await client.getBooleanValue('new-feature', false);

console.log('Feature enabled:', isEnabled);
```

## Vanilla JavaScript Example

```html
<!DOCTYPE html>
<html>
<head>
    <title>FeatBit OpenFeature Demo</title>
</head>
<body>
    <div id="feature-content"></div>
    <button id="toggle-button">Click Me</button>

    <script type="module">
        import { OpenFeature } from 'https://unpkg.com/@openfeature/web-sdk';
        import { FbProvider } from 'https://unpkg.com/@featbit/openfeature-provider-js-client';

        // Initialize provider
        const provider = new FbProvider({
            sdkKey: 'your-sdk-key',
            streamingUri: 'wss://app-eval.featbit.co',
            eventsUri: 'https://app-eval.featbit.co',
            user: {
                keyId: 'user-' + Date.now(),
                name: 'Anonymous User',
                customizedProperties: []
            }
        });

        await OpenFeature.setProviderAndWait(provider);
        const client = OpenFeature.getClient();

        // Evaluate feature flags
        const showNewUI = await client.getBooleanValue('new-ui', false);
        const theme = await client.getStringValue('theme', 'light');

        // Update UI based on flags
        const content = document.getElementById('feature-content');
        if (showNewUI) {
            content.innerHTML = '<h1>New UI Enabled!</h1>';
            content.className = `theme-${theme}`;
        } else {
            content.innerHTML = '<h1>Standard UI</h1>';
        }

        // Track button clicks
        document.getElementById('toggle-button').addEventListener('click', () => {
            // Custom tracking can be done through underlying FBClient
            const fbClient = provider.getClient();
            fbClient.sendCustomEvent('button-clicked');
        });
    </script>
</body>
</html>
```

## React Integration

```javascript
// FeatBitProvider.jsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { OpenFeature } from '@openfeature/web-sdk';
import { FbProvider } from '@featbit/openfeature-provider-js-client';

const FeatBitContext = createContext(null);

export const FeatBitProvider = ({ children, config }) => {
    const [client, setClient] = useState(null);
    const [isReady, setIsReady] = useState(false);

    useEffect(() => {
        const initializeProvider = async () => {
            const provider = new FbProvider(config);
            await OpenFeature.setProviderAndWait(provider);
            
            const ofClient = OpenFeature.getClient();
            setClient(ofClient);
            setIsReady(true);
        };

        initializeProvider();
    }, []);

    return (
        <FeatBitContext.Provider value={{ client, isReady }}>
            {children}
        </FeatBitContext.Provider>
    );
};

export const useFeatBit = () => useContext(FeatBitContext);

// useFeatureFlag.js
import { useState, useEffect } from 'react';
import { useFeatBit } from './FeatBitProvider';

export const useFeatureFlag = (flagKey, defaultValue) => {
    const { client, isReady } = useFeatBit();
    const [value, setValue] = useState(defaultValue);

    useEffect(() => {
        if (!isReady || !client) return;

        const evaluateFlag = async () => {
            let flagValue;
            if (typeof defaultValue === 'boolean') {
                flagValue = await client.getBooleanValue(flagKey, defaultValue);
            } else if (typeof defaultValue === 'string') {
                flagValue = await client.getStringValue(flagKey, defaultValue);
            } else if (typeof defaultValue === 'number') {
                flagValue = await client.getNumberValue(flagKey, defaultValue);
            } else {
                flagValue = await client.getObjectValue(flagKey, defaultValue);
            }
            setValue(flagValue);
        };

        evaluateFlag();
    }, [client, isReady, flagKey, defaultValue]);

    return value;
};

// App.jsx
import React from 'react';
import { FeatBitProvider } from './FeatBitProvider';
import { FeatureComponent } from './FeatureComponent';

function App() {
    const config = {
        sdkKey: 'your-sdk-key',
        streamingUri: 'wss://app-eval.featbit.co',
        eventsUri: 'https://app-eval.featbit.co',
        user: {
            keyId: 'user-123',
            name: 'John Doe',
            customizedProperties: [
                { name: 'role', value: 'user' }
            ]
        }
    };

    return (
        <FeatBitProvider config={config}>
            <FeatureComponent />
        </FeatBitProvider>
    );
}

// FeatureComponent.jsx
import React from 'react';
import { useFeatureFlag } from './useFeatureFlag';

export const FeatureComponent = () => {
    const showNewFeature = useFeatureFlag('new-feature', false);
    const theme = useFeatureFlag('theme', 'light');

    if (!showNewFeature) {
        return <div>Feature not available</div>;
    }

    return (
        <div className={`theme-${theme}`}>
            <h1>New Feature Enabled!</h1>
        </div>
    );
};
```

## Vue Integration

```javascript
// featbit-plugin.js
import { OpenFeature } from '@openfeature/web-sdk';
import { FbProvider } from '@featbit/openfeature-provider-js-client';

export default {
    install(app, options) {
        const provider = new FbProvider(options);
        
        app.config.globalProperties.$featbit = {
            client: null,
            async init() {
                await OpenFeature.setProviderAndWait(provider);
                this.client = OpenFeature.getClient();
            },
            async getBooleanValue(key, defaultValue) {
                return await this.client.getBooleanValue(key, defaultValue);
            },
            async getStringValue(key, defaultValue) {
                return await this.client.getStringValue(key, defaultValue);
            },
            async getNumberValue(key, defaultValue) {
                return await this.client.getNumberValue(key, defaultValue);
            }
        };
    }
};

// main.js
import { createApp } from 'vue';
import App from './App.vue';
import FeatBitPlugin from './featbit-plugin';

const app = createApp(App);

app.use(FeatBitPlugin, {
    sdkKey: 'your-sdk-key',
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co',
    user: {
        keyId: 'user-123',
        name: 'John Doe'
    }
});

await app.config.globalProperties.$featbit.init();

app.mount('#app');

// Component.vue
<template>
    <div v-if="showFeature">
        <h1>New Feature</h1>
    </div>
</template>

<script>
export default {
    data() {
        return {
            showFeature: false
        };
    },
    async mounted() {
        this.showFeature = await this.$featbit.getBooleanValue('new-feature', false);
    }
};
</script>
```

## Flag Evaluation

### Boolean Flags

```javascript
const isEnabled = await client.getBooleanValue('feature-flag', false);
```

### String Flags

```javascript
const theme = await client.getStringValue('theme', 'default');
```

### Number Flags

```javascript
const maxItems = await client.getNumberValue('max-items', 10);
```

### Object Flags

```javascript
const config = await client.getObjectValue('config', {});
```

### Evaluation Details

```javascript
const details = await client.getBooleanDetails('feature-flag', false);

console.log('Value:', details.value);
console.log('Reason:', details.reason);
console.log('Variant:', details.variant);
```

## User Identification

```javascript
// Update user context
const newProvider = new FbProvider({
    sdkKey: 'your-sdk-key',
    streamingUri: 'wss://app-eval.featbit.co',
    eventsUri: 'https://app-eval.featbit.co',
    user: {
        keyId: 'new-user-456',
        name: 'Jane Smith',
        customizedProperties: [
            { name: 'role', value: 'admin' },
            { name: 'plan', value: 'premium' }
        ]
    }
});

await OpenFeature.setProviderAndWait(newProvider);
```

## Access Underlying FBClient

```javascript
const provider = new FbProvider({...});
await OpenFeature.setProviderAndWait(provider);

// Access FeatBit-specific features
const fbClient = provider.getClient();

// Send custom events
fbClient.sendCustomEvent('custom-event-name');

// Track metrics
fbClient.trackMetric('metric-name', numericValue);

// Get all flags
const allFlags = fbClient.getAllLatestFlagsVariations();
```

## Event Tracking

```javascript
// Access underlying FBClient for tracking
const fbClient = provider.getClient();

// Track button click
document.getElementById('my-button').addEventListener('click', () => {
    fbClient.sendCustomEvent('button-clicked');
});

// Track with value
fbClient.trackMetric('purchase-amount', 99.99);
```

## Best Practices

### 1. Wait for Provider Ready

```javascript
// ✅ Good: Use setProviderAndWait
await OpenFeature.setProviderAndWait(provider);
const client = OpenFeature.getClient();

// ❌ Bad: Don't wait
OpenFeature.setProvider(provider); // Async, may not be ready
const client = OpenFeature.getClient();
```

### 2. Single Provider Instance

```javascript
// ✅ Good: Initialize once
const provider = new FbProvider({...});
await OpenFeature.setProviderAndWait(provider);

// ❌ Bad: Multiple providers
const provider1 = new FbProvider({...});
const provider2 = new FbProvider({...}); // Don't do this
```

### 3. Error Handling

```javascript
try {
    const value = await client.getBooleanValue('flag-key', false);
    // Use feature
} catch (error) {
    console.error('Flag evaluation failed:', error);
    // Fall back to default
    const value = false;
}
```

## Troubleshooting

### Provider Not Initializing

```javascript
try {
    await OpenFeature.setProviderAndWait(provider);
} catch (error) {
    console.error('Provider initialization failed:', error);
    // Check:
    // 1. SDK key is correct
    // 2. Network connectivity
    // 3. CORS configuration
}
```

### Flags Not Updating

```javascript
// Ensure WebSocket connection is active
const fbClient = provider.getClient();
fbClient.on('update', (changes) => {
    console.log('Flags updated:', changes);
});
```

## Advantages Over Direct SDK

### 1. Vendor Neutrality
```javascript
// Easy to switch providers
// Just change initialization, code stays the same
```

### 2. Standardized API
```javascript
// Consistent across all providers
const value = await client.getBooleanValue('flag', false);
```

### 3. Hooks and Middleware
```javascript
OpenFeature.addHooks({
    before: async (context) => {
        console.log('Evaluating:', context.flagKey);
    }
});
```

## Additional Resources

- **GitHub**: https://github.com/featbit/openfeature-provider-js-client
- **OpenFeature**: https://openfeature.dev/
- **OpenFeature Web SDK**: https://openfeature.dev/docs/reference/technologies/client/web
- **FeatBit JavaScript SDK**: https://github.com/featbit/featbit-js-client-sdk
