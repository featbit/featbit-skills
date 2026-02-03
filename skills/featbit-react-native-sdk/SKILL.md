---
name: featbit-react-native-sdk
description: Provides integration guidance for FeatBit React Native client SDK. Use when adding FeatBit feature flags to React Native apps (iOS/Android), initializing the SDK, or evaluating flags in components.
appliesTo:
  - "**/App.tsx"
  - "**/App.jsx"
  - "**/App.ts"
  - "**/App.js"
  - "**/app/**/*.tsx"
  - "**/app/**/*.jsx"
  - "**/app/**/*.ts"
  - "**/app/**/*.js"
  - "**/src/**/*.tsx"
  - "**/src/**/*.jsx"
  - "**/src/**/*.ts"
  - "**/src/**/*.js"
  - "**/*.native.js"
  - "**/*.ios.js"
  - "**/*.android.js"
---

# FeatBit React Native SDK

Guidance for integrating FeatBit feature flags into React Native apps. This SDK is client-side and intended for single-user contexts (mobile, desktop, or embedded apps).

## Activates When

- The user asks about FeatBit React Native SDK setup or initialization.
- The user is integrating feature flags in React Native apps.
- The user needs to evaluate flags or wire up `withFbProvider`/`asyncWithFbProvider`.

## Overview

The React Native SDK builds on FeatBit’s React Client SDK (which depends on the JavaScript client SDK). Most React Client SDK functionality is available; the key difference is how initialization is done in React Native.

## Core Knowledge Areas

### 1) Installation and prerequisites

- Install: `npm install @featbit/react-native-sdk`
- Required values: environment secret (`sdkKey`), streaming URL, and events URL
- Docs for values:
  - https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret
  - https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls
  - `customizedProperties` is an array of `{ name, value }` pairs for user attributes.

### 2) SDK initialization

- Build the config with `buildConfig({ options })`
- Initialize using one of:
  - `withFbProvider`
  - `asyncWithFbProvider`

### 3) Flag evaluation in components

- Use `useFlags()` to read flag values in components
- Example: show UI based on a flag value (e.g., `robot === 'AlphaGo'`)

### 4) Identifying users after initialization

- Use `useFbClient()` from `@featbit/react-client-sdk`, then call `fbClient.identify(user)`
- This pattern comes from the React Client SDK and applies to React Native when using the same hooks.

## Best Practices

1. **Create a single config instance**: Build config once with `buildConfig({ options })` and reuse it at app entry.
2. **Provide required user fields**: Ensure `user.keyId` and `user.name` are set in `options`.
3. **Keep secrets out of source**: Use environment variables or secure storage for `sdkKey` and URLs.
4. **Match option field names to your SDK version**: Use `eventsUrl` or `eventsUri` per your SDK version.

## Standard Usage (Recommended)

### Initialize the SDK at app entry

```tsx
import { buildConfig, withFbProvider } from '@featbit/react-native-sdk';

const options = {
  user: {
    name: 'the user name',
    keyId: 'fb-demo-user-key',
    customizedProperties: [],
  },
  sdkKey: 'YOUR ENVIRONMENT SECRET',
  streamingUri: 'THE STREAMING URL',
  eventsUri: 'THE EVENTS URL',
};

export default withFbProvider(buildConfig({ options }))(App);
```

### Provide `customizedProperties`

```tsx
const options = {
  user: {
    name: 'the user name',
    keyId: 'fb-demo-user-key',
    customizedProperties: [
      { name: 'plan', value: 'pro' },
      { name: 'age', value: '18' },
    ],
  },
  sdkKey: 'YOUR ENVIRONMENT SECRET',
  streamingUri: 'THE STREAMING URL',
  eventsUri: 'THE EVENTS URL',
};
```

### Initialize with `asyncWithFbProvider`

Use this when you want flags ready before initial render.

```tsx
import { asyncWithFbProvider, buildConfig } from '@featbit/react-native-sdk';

const options = { /* same shape as above */ };

export const createRootProvider = async () => {
  const Provider = await asyncWithFbProvider(buildConfig({ options }));
  return Provider;
};
```

### Read flags in components (`useFlags`)

```tsx
import { useFlags } from '@featbit/react-client-sdk';

export const FeatureSection = () => {
  const { robot } = useFlags();
  return robot === 'AlphaGo' ? <Text>AlphaGo 🤖</Text> : null;
};
```

### Identify a user after initialization (`identify`)

```tsx
import { useFbClient } from '@featbit/react-client-sdk';

export const LoginButton = () => {
  const fbClient = useFbClient();

  const handleLogin = async () => {
    const user = { name: 'the user name', keyId: 'fb-demo-user-key' };
    await fbClient.identify(user);
  };

  return <Button title="Login" onPress={handleLogin} />;
};
```

## Documentation Reference

- Official SDK guide (complete reference): https://github.com/featbit/featbit-react-native-sdk
- Examples (React Native + Expo): https://github.com/featbit/featbit-react-native-sdk/tree/main/examples
- React Client SDK (dependency): https://github.com/featbit/featbit-react-client-sdk
- FeatBit SDK FAQ: https://docs.featbit.co/sdk/faq

> If this SKILL.md does not fully cover your scenario, refer to the complete guide at https://github.com/featbit/featbit-react-native-sdk for full usage details and examples.

## Related Topics

- FeatBit React Client SDK
- FeatBit JavaScript Client SDK
- Feature flag evaluation patterns
