---
name: featbit-react-client-sdk
description: Expert guidance for integrating FeatBit React Client SDK in React and Next.js applications. Use when users work with React, need React hooks, or ask about React/Next.js feature flag integration.
appliesTo:
  - "**/*.jsx"
  - "**/*.tsx"
  - "**/App.jsx"
  - "**/App.tsx"
  - "**/_app.js"
  - "**/_app.tsx"
---

# FeatBit React Client SDK Expert

Expert knowledge for integrating FeatBit in React and Next.js applications with hooks and components.

**Source**: https://github.com/featbit/featbit-react-client-sdk

✨ **Built on top of**: [@featbit/js-client-sdk](https://github.com/featbit/featbit-js-client-sdk)

## Compatibility

- **React**: 16.3.0+ (16.8.0+ for hooks)
- **Next.js**: Client-side only (not SSR compatible)

⚠️ **Next.js Note**: This SDK is **not compatible with server-side rendering**. Use [@featbit/node-server-sdk](https://github.com/featbit/featbit-node-server-sdk) for SSR.

## Installation

```bash
npm install @featbit/react-client-sdk
```

## Prerequisites

Obtain these values first:
- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
import { createRoot } from 'react-dom/client';
import { asyncWithFbProvider, useFlags } from '@featbit/react-client-sdk';

function Game() {
  return <div>This is a game</div>;
}

function App() {
  const flags = useFlags();
  const gameEnabled = flags['game-enabled'];

  return (
    <div>
      {gameEnabled ? <Game /> : <div>The game is not enabled</div>}
    </div>
  );
}

(async () => {
  const config = {
    options: {
      user: {
        name: 'the user name',
        keyId: 'fb-demo-user-key',
        customizedProperties: []
      },
      sdkKey: 'YOUR_ENVIRONMENT_SECRET',
      streamingUrl: 'wss://app-eval.featbit.co',
      eventsUrl: 'https://app-eval.featbit.co'
    }
  };
  
  const root = createRoot(document.getElementById('root'));
  const Provider = await asyncWithFbProvider(config);
  
  root.render(
    <Provider>
      <App />
    </Provider>
  );
})();
```

## Provider Setup

### Method 1: asyncWithFbProvider (Async Initialization)

```javascript
import { asyncWithFbProvider } from '@featbit/react-client-sdk';

const config = {
  options: {
    user: {
      name: 'User Name',
      keyId: 'user-unique-id',
      customizedProperties: [
        { name: 'country', value: 'US' },
        { name: 'age', value: '25' }
      ]
    },
    sdkKey: 'your-sdk-key',
    streamingUrl: 'wss://app-eval.featbit.co',
    eventsUrl: 'https://app-eval.featbit.co'
  }
};

const Provider = await asyncWithFbProvider(config);

root.render(
  <Provider>
    <App />
  </Provider>
);
```

### Method 2: FbProvider (Synchronous)

```javascript
import { FbProvider } from '@featbit/react-client-sdk';

function Root() {
  const config = {
    user: {
      name: 'User Name',
      keyId: 'user-unique-id',
      customizedProperties: []
    },
    sdkKey: 'your-sdk-key',
    streamingUrl: 'wss://app-eval.featbit.co',
    eventsUrl: 'https://app-eval.featbit.co'
  };

  return (
    <FbProvider config={config}>
      <App />
    </FbProvider>
  );
}
```

## Using Hooks

### useFlags - Get All Flags

```javascript
import { useFlags } from '@featbit/react-client-sdk';

function MyComponent() {
  const flags = useFlags();
  
  const newFeatureEnabled = flags['new-feature'] || false;
  const theme = flags['theme'] || 'default';
  const maxItems = flags['max-items'] || 10;

  return (
    <div>
      {newFeatureEnabled && <NewFeature />}
      <div className={theme}>Content</div>
    </div>
  );
}
```

### useFbClient - Access Client and Flags

```javascript
import { useFbClient } from '@featbit/react-client-sdk';

function MyComponent() {
  const { flags, client } = useFbClient();
  
  const handleAction = () => {
    // Track custom event
    client.sendCustomEvent('button-clicked');
  };

  return (
    <div>
      <button onClick={handleAction}>
        {flags['button-text'] || 'Click Me'}
      </button>
    </div>
  );
}
```

### Custom Hook for Specific Flag

```javascript
import { useFlags } from '@featbit/react-client-sdk';

function useFeatureFlag(flagKey, defaultValue = false) {
  const flags = useFlags();
  return flags[flagKey] ?? defaultValue;
}

// Usage
function Component() {
  const isEnabled = useFeatureFlag('new-feature', false);
  const theme = useFeatureFlag('theme', 'light');
  
  return (
    <div className={theme}>
      {isEnabled && <NewFeature />}
    </div>
  );
}
```

## Identify User (Switch User)

```javascript
import { useFbClient } from '@featbit/react-client-sdk';

function UserSwitcher() {
  const { client } = useFbClient();

  const switchToUser = async (userId, userName) => {
    const newUser = {
      keyId: userId,
      name: userName,
      customizedProperties: [
        { name: 'role', value: 'admin' }
      ]
    };
    
    await client.identify(newUser);
  };

  return (
    <button onClick={() => switchToUser('user-2', 'Jane')}>
      Switch User
    </button>
  );
}
```

## Track Custom Events

```javascript
import { useFbClient } from '@featbit/react-client-sdk';

function PurchaseButton() {
  const { client } = useFbClient();

  const handlePurchase = async () => {
    // Process purchase...
    
    // Track conversion event
    client.sendCustomEvent('purchase-completed');
    
    // Track with value
    client.trackMetric('revenue', 99.99);
  };

  return <button onClick={handlePurchase}>Buy Now</button>;
}
```

## Complete React App Example

```javascript
import React from 'react';
import { createRoot } from 'react-dom/client';
import { asyncWithFbProvider, useFlags, useFbClient } from '@featbit/react-client-sdk';

// Feature Component
function NewDashboard() {
  return (
    <div className="new-dashboard">
      <h2>New Dashboard Design</h2>
      <p>This is the new and improved dashboard!</p>
    </div>
  );
}

function OldDashboard() {
  return (
    <div className="old-dashboard">
      <h2>Dashboard</h2>
      <p>This is the current dashboard</p>
    </div>
  );
}

// Main App
function App() {
  const flags = useFlags();
  const { client } = useFbClient();
  
  const useNewDashboard = flags['new-dashboard'] || false;
  const theme = flags['theme'] || 'light';
  
  const handleFeedback = () => {
    client.sendCustomEvent('feedback-clicked');
  };

  return (
    <div className={`app theme-${theme}`}>
      <header>
        <h1>My Application</h1>
        <button onClick={handleFeedback}>Give Feedback</button>
      </header>
      <main>
        {useNewDashboard ? <NewDashboard /> : <OldDashboard />}
      </main>
    </div>
  );
}

// Initialize and Render
(async () => {
  const config = {
    options: {
      user: {
        name: 'John Doe',
        keyId: 'user-123',
        customizedProperties: [
          { name: 'role', value: 'user' },
          { name: 'plan', value: 'premium' }
        ]
      },
      sdkKey: process.env.REACT_APP_FEATBIT_SDK_KEY,
      streamingUrl: 'wss://app-eval.featbit.co',
      eventsUrl: 'https://app-eval.featbit.co'
    }
  };

  const root = createRoot(document.getElementById('root'));
  const Provider = await asyncWithFbProvider(config);
  
  root.render(
    <React.StrictMode>
      <Provider>
        <App />
      </Provider>
    </React.StrictMode>
  );
})();
```

## Next.js Integration (Client-Side Only)

```javascript
// pages/_app.js
import { useEffect, useState } from 'react';
import { FbProvider } from '@featbit/react-client-sdk';

function MyApp({ Component, pageProps }) {
  const [fbConfig, setFbConfig] = useState(null);

  useEffect(() => {
    // Only initialize on client-side
    if (typeof window !== 'undefined') {
      setFbConfig({
        user: {
          keyId: 'user-id',
          name: 'User Name',
          customizedProperties: []
        },
        sdkKey: process.env.NEXT_PUBLIC_FEATBIT_SDK_KEY,
        streamingUrl: 'wss://app-eval.featbit.co',
        eventsUrl: 'https://app-eval.featbit.co'
      });
    }
  }, []);

  if (!fbConfig) {
    return <Component {...pageProps} />;
  }

  return (
    <FbProvider config={fbConfig}>
      <Component {...pageProps} />
    </FbProvider>
  );
}

export default MyApp;

// pages/index.js
import { useFlags } from '@featbit/react-client-sdk';

export default function Home() {
  const flags = useFlags();
  const newFeature = flags['new-feature'] || false;

  return (
    <div>
      <h1>Home Page</h1>
      {newFeature && <div>New Feature!</div>}
    </div>
  );
}
```

## Advanced Patterns

### Feature Flag HOC

```javascript
import { useFlags } from '@featbit/react-client-sdk';

function withFeatureFlag(Component, flagKey, defaultValue = false) {
  return function FeatureFlagWrapper(props) {
    const flags = useFlags();
    const isEnabled = flags[flagKey] ?? defaultValue;

    if (!isEnabled) {
      return null;
    }

    return <Component {...props} />;
  };
}

// Usage
const BetaFeature = withFeatureFlag(MyBetaComponent, 'beta-feature');

function App() {
  return (
    <div>
      <BetaFeature /> {/* Only renders if flag is enabled */}
    </div>
  );
}
```

### Conditional Rendering Component

```javascript
import { useFlags } from '@featbit/react-client-sdk';

function FeatureFlag({ flag, fallback = null, children }) {
  const flags = useFlags();
  const isEnabled = flags[flag] || false;

  return isEnabled ? children : fallback;
}

// Usage
function App() {
  return (
    <div>
      <FeatureFlag flag="new-ui" fallback={<OldUI />}>
        <NewUI />
      </FeatureFlag>
    </div>
  );
}
```

### A/B Test Component

```javascript
import { useFlags, useFbClient } from '@featbit/react-client-sdk';
import { useEffect } from 'react';

function ABTest({ flagKey, variants, defaultVariant = 'A' }) {
  const flags = useFlags();
  const { client } = useFbClient();
  const variant = flags[flagKey] || defaultVariant;

  useEffect(() => {
    // Track variant shown
    client.sendCustomEvent(`${flagKey}-variant-${variant}`);
  }, [variant]);

  const Component = variants[variant] || variants[defaultVariant];
  return <Component />;
}

// Usage
function App() {
  return (
    <ABTest
      flagKey="landing-page"
      variants={{
        A: LandingPageA,
        B: LandingPageB,
        C: LandingPageC
      }}
      defaultVariant="A"
    />
  );
}
```

## TypeScript Support

```typescript
import { useFlags, useFbClient, FbClient } from '@featbit/react-client-sdk';

interface Flags {
  'new-feature': boolean;
  'theme': string;
  'max-items': number;
}

function MyComponent() {
  const flags = useFlags() as Flags;
  const { client } = useFbClient();

  const isEnabled: boolean = flags['new-feature'];
  const theme: string = flags['theme'] || 'light';
  const maxItems: number = flags['max-items'] || 10;

  return <div className={theme}>Content</div>;
}
```

## Best Practices

### 1. Environment Variables

```javascript
// .env.local
REACT_APP_FEATBIT_SDK_KEY=your-sdk-key
REACT_APP_FEATBIT_STREAMING_URL=wss://app-eval.featbit.co
REACT_APP_FEATBIT_EVENTS_URL=https://app-eval.featbit.co

// Usage
const config = {
  sdkKey: process.env.REACT_APP_FEATBIT_SDK_KEY,
  streamingUrl: process.env.REACT_APP_FEATBIT_STREAMING_URL,
  eventsUrl: process.env.REACT_APP_FEATBIT_EVENTS_URL
};
```

### 2. Loading States

```javascript
import { useState, useEffect } from 'react';
import { useFbClient } from '@featbit/react-client-sdk';

function App() {
  const [isReady, setIsReady] = useState(false);
  const { flags, client } = useFbClient();

  useEffect(() => {
    if (client) {
      client.waitForInitialization().then(() => {
        setIsReady(true);
      });
    }
  }, [client]);

  if (!isReady) {
    return <div>Loading...</div>;
  }

  return <div>App Content</div>;
}
```

### 3. Error Boundaries

```javascript
class FeatureFlagErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Feature flag error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong with feature flags</div>;
    }

    return this.props.children;
  }
}

// Usage
<FeatureFlagErrorBoundary>
  <App />
</FeatureFlagErrorBoundary>
```

## Troubleshooting

### Hooks Not Working

Ensure React version is 16.8.0+:
```bash
npm list react
```

### Next.js SSR Issues

```javascript
// Use dynamic import with ssr: false
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(
  () => import('../components/FeatureComponent'),
  { ssr: false }
);
```

### Flags Not Updating

```javascript
// Listen for updates
const { client } = useFbClient();

useEffect(() => {
  const handleUpdate = () => {
    console.log('Flags updated');
  };
  
  client.on('ff_update', handleUpdate);
  
  return () => {
    client.off('ff_update', handleUpdate);
  };
}, [client]);
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-react-client-sdk
- **NPM**: https://www.npmjs.com/package/@featbit/react-client-sdk
- **Examples**: https://github.com/featbit/featbit-react-client-sdk/tree/main/examples
- **Base SDK**: https://github.com/featbit/featbit-js-client-sdk
