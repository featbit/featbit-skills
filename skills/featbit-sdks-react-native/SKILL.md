---
name: featbit-sdks-react-native
description: >
  Expert guidance for integrating FeatBit React Native SDK in mobile React Native
  and Expo applications. Use when user asks about "React Native SDK", "React Native feature flags",
  "FeatBit mobile", "FeatBit Expo", "buildConfig", "withFbProvider React Native", or
  mentions .tsx/.jsx files in a React Native or Expo project with FeatBit.
  Do not use for React web browser apps, Node.js server-side, Python, Java, Go, or .NET questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit React Native SDK

## When to Use This Skill

Use for React Native mobile applications (iOS/Android) and Expo projects that evaluate feature flags on the client side.

**Why client SDK**: Connects from the mobile device via WebSocket, syncs flag data for the current user, and evaluates flags locally. Supports both React Native CLI and Expo Go.

**Key difference from the React web SDK**: initialization requires a mandatory `buildConfig()` call before the provider, and the initialization package (`@featbit/react-native-sdk`) is separate from the hooks package (`@featbit/react-client-sdk`). User fields are also different: `keyId` and `name` (not `id` and `userName`).

Do not use for React web browser apps (use `featbit-sdks-react`) or any server-side context.

## Source

https://github.com/featbit/featbit-react-native-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Build config and wrap the app with the provider
- [ ] Step 3: Consume feature flags in a component

**Step 1: Install the package**

Run:

```bash
npm install @featbit/react-native-sdk
```

**Step 2: Build config and wrap the app**

Call `buildConfig()` with the options object, then pass the result to `withFbProvider`:

```tsx
// App.tsx
import {buildConfig, withFbProvider} from '@featbit/react-native-sdk';

function App(): React.JSX.Element {
  // App component code
}

const options = {
  user: {
    name: 'the user name',
    keyId: 'fb-demo-user-key',
    customizedProperties: []
  },
  sdkKey: 'YOUR ENVIRONMENT SECRET',
  streamingUri: 'THE STREAMING URL',
  eventsUrl: 'THE EVENTS URL'
};

export default withFbProvider(buildConfig({options}))(App);
```

**Why `buildConfig`**: React Native requires this adapter step to normalize the config before passing it to the provider. Without it, the provider will not initialize correctly.

**Step 3: Consume flags in a component**

```tsx
// src/TestComponent.tsx
import {useFlags} from '@featbit/react-client-sdk';

export default function TestComponent({isDarkMode}: {isDarkMode: boolean}) {
  const {robot} = useFlags();
  return robot === 'AlphaGo' ? <Text>AlphaGo</Text> : null;
}
```

**Important**: `useFlags` is imported from `@featbit/react-client-sdk`, not from `@featbit/react-native-sdk`.

If flags return fallback values unexpectedly, verify `sdkKey`, `streamingUri`, and `eventsUrl`.

## Feature Flag Evaluation

```tsx
import {useFlags, useFbClient} from '@featbit/react-client-sdk';

const MyComponent = () => {
  const {robot, flag1} = useFlags();

  // Or access all flags at once:
  // const flags = useFlags();
  // flags['my-flag-key']  // bracket notation for keys with dashes

  const fbClient = useFbClient();  // underlying JS client for advanced operations

  return <Text>{robot}</Text>;
};
```

`useFlags()` returns all flags — destructure for known keys or use bracket notation. Both hooks are from `@featbit/react-client-sdk`. Use `useFbClient()` when you need the underlying client (e.g., `await fbClient.identify(user)` to switch users after login).

## User Custom Properties

React Native SDK uses `keyId` and `name` (not `id` and `userName` like the React web SDK). Add custom attributes in the `customizedProperties` array:

```tsx
const options = {
  user: {
    keyId: 'user-key-123',
    name: 'Jane',
    customizedProperties: [
      { name: 'age', value: '25' },
      { name: 'country', value: 'US' }
    ]
  },
  sdkKey: 'YOUR ENVIRONMENT SECRET',
  streamingUri: 'THE STREAMING URL',
  eventsUrl: 'THE EVENTS URL'
};
```

To switch users after initialization, call `await fbClient.identify(user)` with an updated user object (via `useFbClient()`).

## Read Next Only When Needed

- Read [Initializing the SDK](https://github.com/featbit/featbit-react-native-sdk#initializing-the-sdk) when the user asks about `asyncWithFbProvider` vs `withFbProvider` or how `buildConfig` works in more detail.
- Read the [React Client SDK README — Consuming flags](https://github.com/featbit/featbit-react-client-sdk#consuming-flags) for full documentation on hooks, class components (`withFbConsumer`, `contextType`), `reactOptions.useCamelCaseFlagKeys`, and bootstrap values — the React Native SDK inherits all of this behavior.
