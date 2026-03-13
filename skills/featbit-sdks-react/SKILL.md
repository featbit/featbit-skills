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

## When to Use This Skill

Use for client-side React applications (React 16.3+) that evaluate feature flags in the browser.

**Why client SDK**: Connects directly from the browser via WebSocket, syncs flag data for the current user, and evaluates flags locally. No backend proxy is required.

Do not use for React Native (use the React Native SDK), Next.js server-side rendering, or any server-side context — use `@featbit/node-server-sdk` for server-side Next.js instead.

## Source

https://github.com/featbit/featbit-react-client-sdk

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

Use `asyncWithFbProvider` at the app entry point before rendering so flags are ready at startup:

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

**Why `asyncWithFbProvider`**: Awaiting initialization prevents flag flicker — the provider resolves before the first render. Use `withFbProvider` only if you prefer to render first and apply flag updates after.

**Step 3: Consume flags in a component**

```tsx
import { useFlags } from '@featbit/react-client-sdk';

function MyComponent() {
  const { 'game-enabled': gameEnabled } = useFlags();
  return gameEnabled ? <Game /> : <div>Feature not available</div>;
}
```

If flags return fallback values unexpectedly, verify `sdkKey`, `streamingUrl`, and `eventsUrl`.

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

`useFlags()` returns all flags. Destructure for known keys or access by bracket/dot notation. Use `useFbClient()` to get the underlying JavaScript SDK client (e.g., to call `await fbClient.identify(user)` after login).

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

## Read Next Only When Needed

- [Initializing the SDK](https://github.com/featbit/featbit-react-client-sdk#initializing-the-sdk) — only when the user asks about `asyncWithFbProvider` vs `withFbProvider` or deferred initialization.
- [Consuming flags](https://github.com/featbit/featbit-react-client-sdk#consuming-flags) — only when the user asks about `withFbConsumer`, `contextType`, or class component patterns.
- [Switch user after initialization](https://github.com/featbit/featbit-react-client-sdk#switch-user-after-initialization) — only when the user asks about calling `fbClient.identify()` post-login.
- [Fallback flag values](https://github.com/featbit/featbit-react-client-sdk#populating-the-sdk-with-fallback-flag-values) — only when the user asks about `options.bootstrap` for pre-render default values.
- [Flag keys](https://github.com/featbit/featbit-react-client-sdk#flag-keys) — only when the user asks about `useCamelCaseFlagKeys` or key collision issues.
