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

# FeatBit React Client SDK

Expert guidance for integrating the FeatBit React Client SDK in React and Next.js applications. This is a browser-only SDK built on top of FeatBit's JavaScript SDK, optimized for React 16.3.0+.

## Activates When

- The user asks about FeatBit React SDK setup, initialization, or usage.
- The user mentions `asyncWithFbProvider`, `withFbProvider`, `useFlags`, `useFbClient`, or `withFbConsumer`.
- The user needs client-side feature flag integration in React or Next.js applications.
- The user is working with React components (class or function) and needs feature flag evaluation.

## Overview

This is a **client-side SDK** for browser environments only. It wraps the JavaScript SDK to provide React-specific features like Context API integration, custom hooks, and automatic flag subscription. **Not suitable for React Native** (use the dedicated React Native SDK instead).

**Key capabilities:**
- Initialize via `asyncWithFbProvider` (async, pre-rendered) or `withFbProvider` (render-first)
- Access flags via `useFlags` hook or `withFbConsumer` HOC
- Automatic subscription to flag changes (no manual opt-in needed)
- Support for both class and function components
- Optional camelCase flag keys
- Bootstrap with default flag values

**Next.js compatibility:** Client-side only. Use `@featbit/node-server-sdk` for server-side rendering.

## Core Knowledge Areas

### 1. Prerequisites

- Install: `npm install @featbit/react-client-sdk`
- Required values: `sdkKey` (environment secret), `streamingUrl`, `eventsUrl`
- Official FAQ: environment secret + SDK URLs

### 2. Initialization Methods

Two approaches with different timing:
- **asyncWithFbProvider**: Async initialization before render (React 16.8.0+, Hooks required). Ensures flags ready at app start, may delay initial render up to 100ms.
- **withFbProvider**: Wraps root component, renders first, processes flag updates after. No Hooks requirement.

Both methods use React Context API and accept a `ProviderConfig` object.

### 3. Consuming Flags

**Class components:**
- `contextType = context` to access `flags` and `fbClient`
- `withFbConsumer()` HOC to inject props

**Function components:**
- `withFbConsumer()` HOC to inject props
- `useFlags()` hook for all flags (React 16.8.0+)
- `useFbClient()` hook for client instance (React 16.8.0+)

### 4. User Management

- Set user during initialization in `ProviderConfig.options.user`
- Switch user after initialization: call `fbClient.identify(user)` (returns Promise)

### 5. Configuration Options

- `options`: Initialization config for JS SDK (mandatory)
- `reactOptions.useCamelCaseFlagKeys`: Auto-convert flag keys to camelCase (default: false)
- `deferInitialization`: Delay SDK init until client defined (not supported by `asyncWithFbProvider`)
- `options.bootstrap`: Populate SDK with fallback flag values

### 6. Flag Keys and Naming

- FeatBit flag keys: alphanumeric, dots, underscores, dashes
- Access via bracket notation: `flags['dev-flag-test']`
- Optional camelCase: enable `reactOptions.useCamelCaseFlagKeys` → access as `flags.devFlagTest`
- ⚠️ CamelCase caveats: key collisions, 3+ capitals conversion, JS SDK uses original keys

## Quick Start (Concise)

```javascript
import { createRoot } from 'react-dom/client';
import { asyncWithFbProvider, useFlags } from '@featbit/react-client-sdk';

function App() {
  const flags = useFlags();
  const gameEnabled = flags['game-enabled'];

  return (
    <div>
      {gameEnabled ? <Game /> : <div>Game disabled</div>}
    </div>
  );
}

(async () => {
  const config = {
    options: {
      sdkKey: 'YOUR ENVIRONMENT SECRET',
      streamingUrl: 'THE STREAMING URL',
      eventsUrl: 'THE EVENTS URL',
      user: {
        name: 'the user name',
        keyId: 'fb-demo-user-key',
        customizedProperties: []
      }
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

For complete examples and all initialization/consumption patterns, see the reference guides below.

## Reference Guides

- [references/initialization.md](references/initialization.md) — asyncWithFbProvider, withFbProvider, ProviderConfig details, flag change subscription
- [references/consuming-flags.md](references/consuming-flags.md) — Class components (contextType, withFbConsumer), function components (hooks, HOC)
- [references/user-management.md](references/user-management.md) — Switching users after initialization
- [references/configuration-and-advanced.md](references/configuration-and-advanced.md) — Flag keys, camelCase, bootstrap, types

## Best Practices

1. **Choose the right initialization**: Use `asyncWithFbProvider` for flag-ready apps, `withFbProvider` for render-first apps.
2. **Use hooks when possible**: `useFlags()` and `useFbClient()` are simpler than HOCs for function components (React 16.8.0+).
3. **Avoid camelCase unless necessary**: Bracket notation `flags['flag-key']` avoids key collision issues.
4. **Bootstrap for offline/fallback**: Use `options.bootstrap` to provide default flag values before SDK connects.
5. **Client-side only in Next.js**: Never use this SDK in server-side rendering; use `@featbit/node-server-sdk` instead.

## Documentation Reference

- **Complete Guide** (full README and latest updates): https://github.com/featbit/featbit-react-client-sdk
- Official Docs: https://docs.featbit.co/sdk/overview#react
- Live Examples: [React App](https://github.com/featbit/featbit-react-client-sdk/tree/main/examples/react-app), [Next.js App](https://github.com/featbit/featbit-react-client-sdk/tree/main/examples/nextjs-app)

## Related Topics

- FeatBit JavaScript Client SDK (underlying SDK)
- FeatBit React Native SDK (for mobile apps)
- FeatBit Node.js Server SDK (for Next.js server-side rendering)
- Next.js integration patterns
