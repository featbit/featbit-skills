---
name: featbit-sdks-react
description: Expert guidance for integrating FeatBit React Client SDK in React web applications. Use when user asks about "React SDK", "React feature flags", "FeatBit React", "useFlags hook", "asyncWithFbProvider", "withFbProvider", or mentions .tsx/.jsx files with FeatBit. Do not use for React Native, Node.js server-side rendering, Python, Java, Go, .NET, or Next.js server-side questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit React Client SDK

## Execution Procedure

```
1. Install package: npm install @featbit/react-client-sdk
2. Choose provider:
   - withFbProvider      → synchronous, renders immediately, feature flags arrive after
   - asyncWithFbProvider → asynchronous, awaits initialization, feature flags ready at first render
3. Wrap app root with the chosen provider, passing sdkKey + streamingUrl + eventsUrl + user
4. In child components, call useFlags() to read feature flag values
5. Branch UI logic on feature flag values (boolean, string, or JSON)
```

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Wrap the app root with the provider
- [ ] Step 3: Consume feature flags in a component

**Step 1: Install the package**

Run:

```bash
npm install @featbit/react-client-sdk
```

**Step 2: Wrap the app root with the provider**

Use `asyncWithFbProvider` at the app entry point before rendering so feature flags are ready at startup:

```tsx
import { createRoot } from 'react-dom/client';
import { asyncWithFbProvider } from '@featbit/react-client-sdk';

(async () => {
  const config = {
    options: {
      sdkKey: 'YOUR ENVIRONMENT SECRET',
      streamingUrl: 'THE STREAMING URL',
      eventsUrl: 'THE EVENTS URL',
      user: {
        id: 'user-key-123',
        userName: 'Jane',
        customizedProperties: []
      }
    }
  };

  const FbProvider = await asyncWithFbProvider(config);
  const root = createRoot(document.getElementById('root'));
  root.render(
    <FbProvider>
      <App />
    </FbProvider>
  );
})();
```

**Why `asyncWithFbProvider`**: Awaiting initialization prevents feature flag flicker — the provider resolves before the first render. Use `withFbProvider` only if you prefer to render first and apply feature flag updates after.

**Step 3: Consume feature flags in a component**

```tsx
import { useFlags } from '@featbit/react-client-sdk';

function MyComponent() {
  const { 'game-enabled': gameEnabled } = useFlags();
  return gameEnabled ? <Game /> : <div>Feature not available</div>;
}
```

If feature flags return fallback values unexpectedly, verify `sdkKey`, `streamingUrl`, and `eventsUrl`.

## Feature Flag Evaluation

```tsx
import { useFlags, useFbClient } from '@featbit/react-client-sdk';

const MyComponent = props => {
  const fbClient = useFbClient();
  const { flag1, flag2 } = useFlags();

  // Or access all flags by key:
  // const flags = useFlags();
  // flags['game-enabled']  // bracket notation required for keys with dashes
  // flags.flag1            // dot notation works for simple alphanumeric keys

  return (
    <div>
      <div>flag1: {flag1}</div>
      <div>flag2: {flag2}</div>
    </div>
  );
};
```

`useFlags()` returns all feature flags. Destructure for known keys or access by bracket/dot notation. Use `useFbClient()` to get the underlying JavaScript SDK client (e.g., to call `await fbClient.identify(user)` after login).

**Class components** — two options:
- `withFbConsumer`: wraps the component and injects `flags` and `fbClient` as props
- `static contextType = context`: assign `context` from the SDK and access `this.context.flags`

## User Custom Properties

React SDK users pass custom properties as a `customizedProperties` array of `{name, value}` objects. Add an entry for each attribute referenced in targeting rules:

```tsx
const config = {
  options: {
    sdkKey: 'YOUR ENVIRONMENT SECRET',
    streamingUrl: 'THE STREAMING URL',
    eventsUrl: 'THE EVENTS URL',
    user: {
      id: 'user-key-123',
      userName: 'Jane',
      customizedProperties: [
        { name: 'age', value: '25' },
        { name: 'country', value: 'FR' },
        { name: 'plan', value: 'premium' }
      ]
    }
  }
};
```

To update the user after login, call `await fbClient.identify(user)` with the same shape. See [Switch user after initialization](https://github.com/featbit/featbit-react-client-sdk#switch-user-after-initialization).

