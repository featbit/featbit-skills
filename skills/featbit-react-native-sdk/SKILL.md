---
name: featbit-react-native-sdk
description: Expert guidance for integrating FeatBit in React Native mobile applications (iOS & Android). Use when developing React Native apps with feature flags.
appliesTo:
  - "**/App.tsx"
  - "**/App.jsx"
  - "**/app/**/*.tsx"
  - "**/app/**/*.jsx"
  - "**/src/**/*.tsx"
  - "**/src/**/*.jsx"
  - "**/*.native.js"
  - "**/*.ios.js"
  - "**/*.android.js"
---

# FeatBit React Native SDK Expert

Expert knowledge for integrating FeatBit in React Native mobile applications.

**Source**: https://github.com/featbit/featbit-react-native-sdk

✨ **Built on**: FeatBit React Client SDK (which uses JavaScript client SDK)

⚠️ **Client-Side SDK**: For single-user mobile contexts (iOS & Android apps)

## Installation

```bash
npm install @featbit/react-native-sdk
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```javascript
// App.tsx
import React from 'react';
import { SafeAreaView, Text, View, useColorScheme } from 'react-native';
import { Colors } from 'react-native/Libraries/NewAppScreen';
import { buildConfig, withFbProvider } from '@featbit/react-native-sdk';
import TestComponent from './src/TestComponent';

function App(): React.JSX.Element {
  const isDarkMode = useColorScheme() === 'dark';

  const backgroundStyle = {
    backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
  };

  return (
    <SafeAreaView style={backgroundStyle}>
      <TestComponent isDarkMode={isDarkMode} />
    </SafeAreaView>
  );
}

const options = {
  user: {
    name: 'Mobile User',
    keyId: 'user-unique-id',
    customizedProperties: [
      { name: 'platform', value: Platform.OS },
      { name: 'version', value: Platform.Version }
    ]
  },
  sdkKey: 'YOUR_ENVIRONMENT_SECRET',
  streamingUri: 'wss://app-eval.featbit.co',
  eventsUrl: 'https://app-eval.featbit.co'
};

const config = buildConfig({ options });
export default withFbProvider(config)(App);
```

```javascript
// src/TestComponent.tsx
import React from 'react';
import { Text, View } from 'react-native';
import { useFlags } from '@featbit/react-native-sdk';

export default function TestComponent({ isDarkMode }) {
  const { robot, theme } = useFlags();
  
  return (
    <View style={{ marginTop: 32, paddingHorizontal: 24 }}>
      {robot === 'AlphaGo' && (
        <Text style={{
          color: isDarkMode ? '#FFFFFF' : '#000000',
          textAlign: 'center',
          fontSize: 30,
          fontWeight: '700'
        }}>
          AlphaGo 🤖
        </Text>
      )}
      <Text>Current theme: {theme || 'default'}</Text>
    </View>
  );
}
```

## Provider Setup

### Method 1: withFbProvider (Recommended)

```javascript
import { buildConfig, withFbProvider } from '@featbit/react-native-sdk';
import { Platform } from 'react-native';

const options = {
  user: {
    name: 'John Doe',
    keyId: 'user-123',
    customizedProperties: [
      { name: 'platform', value: Platform.OS },
      { name: 'deviceId', value: DeviceInfo.getUniqueId() },
      { name: 'appVersion', value: '1.0.0' }
    ]
  },
  sdkKey: process.env.FEATBIT_SDK_KEY,
  streamingUri: 'wss://app-eval.featbit.co',
  eventsUrl: 'https://app-eval.featbit.co'
};

// IMPORTANT: Must call buildConfig before withFbProvider
const config = buildConfig({ options });

export default withFbProvider(config)(App);
```

### Method 2: asyncWithFbProvider

```javascript
import { asyncWithFbProvider, buildConfig } from '@featbit/react-native-sdk';

const options = {
  user: {
    keyId: 'user-id',
    name: 'User Name',
    customizedProperties: []
  },
  sdkKey: 'your-sdk-key',
  streamingUri: 'wss://app-eval.featbit.co',
  eventsUrl: 'https://app-eval.featbit.co'
};

const config = buildConfig({ options });

(async () => {
  const Provider = await asyncWithFbProvider(config);
  
  // Render app with provider
  AppRegistry.registerComponent(appName, () => () => (
    <Provider>
      <App />
    </Provider>
  ));
})();
```

## Using Hooks

### useFlags - Get All Flags

```javascript
import { useFlags } from '@featbit/react-native-sdk';
import { View, Text } from 'react-native';

function FeatureComponent() {
  const flags = useFlags();
  
  const newUI = flags['new-ui'] || false;
  const theme = flags['theme'] || 'light';
  const maxItems = flags['max-items'] || 10;

  return (
    <View>
      {newUI && <Text>New UI Enabled</Text>}
      <Text>Theme: {theme}</Text>
      <Text>Max Items: {maxItems}</Text>
    </View>
  );
}
```

### useFbClient - Access Client

```javascript
import { useFbClient } from '@featbit/react-native-sdk';
import { Button } from 'react-native';

function TrackingComponent() {
  const { flags, client } = useFbClient();
  
  const handlePress = () => {
    client.sendCustomEvent('button-pressed');
  };

  return (
    <Button 
      title={flags['button-text'] || 'Click Me'} 
      onPress={handlePress}
    />
  );
}
```

## Complete Mobile App Example

```javascript
// App.tsx
import React from 'react';
import {
  SafeAreaView,
  View,
  Text,
  Button,
  StyleSheet,
  Platform
} from 'react-native';
import { buildConfig, withFbProvider, useFlags, useFbClient } from '@featbit/react-native-sdk';

function FeatureScreen() {
  const flags = useFlags();
  const { client } = useFbClient();
  
  const darkModeEnabled = flags['dark-mode'] || false;
  const premiumFeatures = flags['premium-features'] || false;
  
  const handleAction = () => {
    client.sendCustomEvent('action-performed');
  };

  return (
    <View style={[
      styles.container,
      darkModeEnabled && styles.darkContainer
    ]}>
      <Text style={darkModeEnabled && styles.darkText}>
        Feature Flags Demo
      </Text>
      
      {premiumFeatures && (
        <View style={styles.premiumSection}>
          <Text>🌟 Premium Features Enabled</Text>
        </View>
      )}
      
      <Button 
        title="Perform Action" 
        onPress={handleAction}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#FFFFFF'
  },
  darkContainer: {
    backgroundColor: '#000000'
  },
  darkText: {
    color: '#FFFFFF'
  },
  premiumSection: {
    padding: 15,
    backgroundColor: '#FFD700',
    borderRadius: 8,
    marginVertical: 10
  }
});

const options = {
  user: {
    name: Platform.OS === 'ios' ? 'iOS User' : 'Android User',
    keyId: 'mobile-user-123',
    customizedProperties: [
      { name: 'platform', value: Platform.OS },
      { name: 'version', value: Platform.Version.toString() }
    ]
  },
  sdkKey: 'YOUR_SDK_KEY',
  streamingUri: 'wss://app-eval.featbit.co',
  eventsUrl: 'https://app-eval.featbit.co'
};

const config = buildConfig({ options });

export default withFbProvider(config)(FeatureScreen);
```

## User Identification

### Identify New User

```javascript
import { useFbClient } from '@featbit/react-native-sdk';

function UserSwitcher() {
  const { client } = useFbClient();

  const switchUser = async (userId, userName) => {
    const newUser = {
      keyId: userId,
      name: userName,
      customizedProperties: [
        { name: 'role', value: 'user' }
      ]
    };
    
    await client.identify(newUser);
  };

  return (
    <Button 
      title="Switch User" 
      onPress={() => switchUser('user-2', 'Jane')}
    />
  );
}
```

### Anonymous Users

```javascript
import DeviceInfo from 'react-native-device-info';

const deviceId = await DeviceInfo.getUniqueId();

const options = {
  user: {
    keyId: `anonymous-${deviceId}`,
    name: 'Anonymous User',
    customizedProperties: [
      { name: 'anonymous', value: 'true' },
      { name: 'platform', value: Platform.OS }
    ]
  },
  // ... other options
};
```

## Device Information Integration

```javascript
import DeviceInfo from 'react-native-device-info';
import { Platform } from 'react-native';

async function getUserContext() {
  const deviceId = await DeviceInfo.getUniqueId();
  const appVersion = DeviceInfo.getVersion();
  const systemVersion = DeviceInfo.getSystemVersion();
  
  return {
    keyId: deviceId,
    name: 'Mobile User',
    customizedProperties: [
      { name: 'platform', value: Platform.OS },
      { name: 'systemVersion', value: systemVersion },
      { name: 'appVersion', value: appVersion },
      { name: 'deviceBrand', value: await DeviceInfo.getBrand() },
      { name: 'deviceModel', value: await DeviceInfo.getModel() }
    ]
  };
}

const user = await getUserContext();
const options = { user, sdkKey, streamingUri, eventsUrl };
```

## Event Tracking

```javascript
import { useFbClient } from '@featbit/react-native-sdk';

function PurchaseButton() {
  const { client } = useFbClient();

  const handlePurchase = async () => {
    try {
      const result = await processPurchase();
      
      if (result.success) {
        // Track conversion
        client.sendCustomEvent('purchase-completed');
        
        // Track revenue
        client.trackMetric('revenue', result.amount);
      }
    } catch (error) {
      console.error('Purchase failed:', error);
    }
  };

  return (
    <Button title="Buy Now" onPress={handlePurchase} />
  );
}
```

## Navigation Integration

### React Navigation

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useFbClient } from '@featbit/react-native-sdk';

const Stack = createNativeStackNavigator();

function AppNavigator() {
  const { flags } = useFbClient();
  const showNewScreen = flags['new-screen-enabled'] || false;

  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        {showNewScreen && (
          <Stack.Screen name="New" component={NewScreen} />
        )}
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

## Platform-Specific Flags

```javascript
import { Platform } from 'react-native';
import { useFlags } from '@featbit/react-native-sdk';

function PlatformFeature() {
  const flags = useFlags();
  
  const iosFeature = flags['ios-feature'] || false;
  const androidFeature = flags['android-feature'] || false;
  
  const isFeatureEnabled = Platform.select({
    ios: iosFeature,
    android: androidFeature,
    default: false
  });

  if (!isFeatureEnabled) return null;

  return <FeatureComponent />;
}
```

## Best Practices

### 1. Single Provider Instance

```javascript
// ✅ Good: Wrap entire app once
export default withFbProvider(config)(App);

// ❌ Bad: Multiple providers
<FbProvider>
  <FbProvider> // Don't nest providers
    <App />
  </FbProvider>
</FbProvider>
```

### 2. Environment Variables

```javascript
// .env
FEATBIT_SDK_KEY=your-sdk-key
FEATBIT_STREAMING_URL=wss://app-eval.featbit.co
FEATBIT_EVENTS_URL=https://app-eval.featbit.co

// Config
import Config from 'react-native-config';

const options = {
  sdkKey: Config.FEATBIT_SDK_KEY,
  streamingUri: Config.FEATBIT_STREAMING_URL,
  eventsUrl: Config.FEATBIT_EVENTS_URL,
  // ...
};
```

### 3. Loading States

```javascript
import { useState, useEffect } from 'react';
import { useFbClient } from '@featbit/react-native-sdk';
import { ActivityIndicator } from 'react-native';

function App() {
  const [isReady, setIsReady] = useState(false);
  const { client } = useFbClient();

  useEffect(() => {
    if (client) {
      client.waitForInitialization().then(() => {
        setIsReady(true);
      });
    }
  }, [client]);

  if (!isReady) {
    return <ActivityIndicator size="large" />;
  }

  return <MainApp />;
}
```

### 4. Error Handling

```javascript
import { useFbClient } from '@featbit/react-native-sdk';

function SafeFeature() {
  const { flags } = useFbClient();
  
  try {
    const isEnabled = flags['feature-key'] ?? false;
    
    if (isEnabled) {
      return <NewFeature />;
    }
  } catch (error) {
    console.error('Feature flag error:', error);
  }
  
  return <DefaultFeature />;
}
```

## Offline Support

```javascript
import NetInfo from '@react-native-community/netinfo';
import { useFbClient } from '@featbit/react-native-sdk';

function App() {
  const { client } = useFbClient();

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      if (state.isConnected) {
        // Reconnect when back online
        client.identify(currentUser);
      }
    });

    return () => unsubscribe();
  }, []);

  return <MainApp />;
}
```

## Testing

```javascript
// __tests__/App.test.js
import React from 'react';
import { render } from '@testing-library/react-native';
import { buildConfig, withFbProvider } from '@featbit/react-native-sdk';
import App from '../App';

describe('App with Feature Flags', () => {
  it('renders correctly', () => {
    const mockOptions = {
      user: { keyId: 'test-user', name: 'Test' },
      sdkKey: 'test-key',
      streamingUri: 'wss://test',
      eventsUrl: 'https://test'
    };

    const config = buildConfig({ options: mockOptions });
    const AppWithProvider = withFbProvider(config)(App);

    const { getByText } = render(<AppWithProvider />);
    expect(getByText('Feature Flags Demo')).toBeTruthy();
  });
});
```

## Troubleshooting

### SDK Not Initializing

```javascript
// Check configuration
console.log('SDK Config:', {
  sdkKey: options.sdkKey.substring(0, 10) + '...',
  streamingUri: options.streamingUri,
  user: options.user.keyId
});

// Wait for initialization
client.waitForInitialization()
  .then(() => console.log('SDK ready'))
  .catch(err => console.error('SDK failed:', err));
```

### Network Issues

```javascript
// Check network connectivity
import NetInfo from '@react-native-community/netinfo';

NetInfo.fetch().then(state => {
  console.log('Connection type:', state.type);
  console.log('Is connected?', state.isConnected);
});
```

### Flags Not Updating

```javascript
// Listen for updates
client.on('ff_update', (changes) => {
  console.log('Flags updated:', changes);
});
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-react-native-sdk
- **NPM**: https://www.npmjs.com/package/@featbit/react-native-sdk
- **Base SDK**: https://github.com/featbit/featbit-react-client-sdk
- **Examples**: https://github.com/featbit/featbit-samples
