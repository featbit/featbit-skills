# React SDK Initialization

## Install

```bash
npm install @featbit/react-client-sdk
```

## Prerequisite

Before using the SDK, you need to obtain the environment secret and SDK URLs.

Follow the documentation below to retrieve these values:

- [How to get the environment secret](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- [How to get the SDK URLs](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

---

## Initializing using asyncWithFbProvider

The `asyncWithFbProvider` function initializes the React SDK and returns a provider which is a React component. It is an async function. It accepts a `ProviderConfig` object.

`asyncWithFbProvider` cannot be deferred. You must initialize `asyncWithFbProvider` at the app entry point prior to rendering to ensure flags and the client are ready at the beginning of the app.

**Usage:**

```javascript
import { createRoot } from 'react-dom/client';
import { asyncWithFbProvider } from '@featbit/react-client-sdk';

(async () => {
  const config = {
    options: {
      sdkKey: 'YOUR ENVIRONMENT SECRET',
      streamingUrl: 'THE STREAMING URL',
      eventsUrl: 'THE EVENTS URL',
      user: {
        name: 'USER NAME',
        keyId: 'USER ID',
        customizedProperties: [
          {
            name: 'age',
            value: '18'
          }
        ]
      }
    }
  }

  const root = createRoot(document.getElementById('root'));
  const FbProvider = await asyncWithFbProvider(config);
  root.render(
    <FbProvider>
      <YourApp />,
    </FbProvider>
  );
})();
```

After initialization is complete, your flags and the client become available at the start of your React app lifecycle. This ensures your app does not flicker due to flag changes at startup time.

**Note:** Because the asyncWithFbProvider function is asynchronous, the rendering of your React app is delayed until initialization is completed. This can take up to 100 milliseconds, but often completes sooner. Alternatively, you can use the withFbProvider function if you prefer to render your app first and then process flag updates after rendering is complete.

**Requires React 16.8.0 or later:** The asyncWithFbProvider function uses the Hooks API.

## Initializing using withFbProvider

The `withFbProvider` function initializes the React SDK and wraps your root component in a Context.Provider. It accepts a `ProviderConfig` object.

**Usage:**

```javascript
import { withFbProvider } from '@featbit/react-client-sdk';

const config = {
  options: {
    sdkKey: 'YOUR ENVIRONMENT SECRET',
    streamingUrl: 'THE STREAMING URL',
    eventsUrl: 'THE EVENTS URL',
    user: {
      name: 'USER NAME',
      keyId: 'USER ID',
      customizedProperties: [
        {
          name: 'age',
          value: '18'
        }
      ]
    }
  }
};

export default withFbProvider(config)(YourApp);
```

## Subscribing to flag changes

The React SDK automatically subscribes to flag change events. This is different from the JavaScript SDK, where customers need to opt in to event listening.

## Configuring the React SDK

The `ProviderConfig` object provides configuration to both `withFbProvider` and `asyncWithFbProvider` function.

The only mandatory property is the **options**, it is the config needed to initialize the `@featbit/js-client-sdk`. To know more details about the **options**, please refer to [@featbit/js-client-sdk](https://github.com/featbit/featbit-js-client-sdk). All other properties are React SDK related.

The complete liste of the available properties:

- **options**: the initialization config for `@featbit/js-client-sdk`. **mandatory**

  You can use the **[options.bootstrap](https://github.com/featbit/featbit-js-client-sdk?tab=readme-ov-file#bootstrap)** option to populate the SDK with default values.
- **reactOptions**: You can use this option to enable automatic camel casing of flag keys when using the React SDK. the default value is false  **not mandatory**
- **deferInitialization**: This property allows SDK initialization to be deferred until you define the fbClient property. the default value is false **not mandatory**

  asyncWithFbProvider does not support deferInitialization. You must initialize asyncWithFbProvider at the app entry point prior to rendering to ensure flags and the client are ready at the beginning of your app.

  By deferring SDK initialization, you defer all steps which take place as part of SDK initialization, including reading flag values from local storage and sending the SDK's ready event.

  The one exception to this rule is that the SDK continues to load bootstrapped flag values as long as the bootstrapped values are provided as a map of flag keys and values. If you indicate that the SDK should bootstrap flags from local storage, this will not happen until the SDK initializes.

The following is an example ProviderConfig object including each of the above properties:
```javascript
{
  options: {
    sdkKey: 'YOUR ENVIRONMENT SECRET',
    streamingUrl: 'THE STREAMING URL',
    eventsUrl: 'THE EVENTS URL',
    user: {
      userName: 'USER NAME',
      id: 'USER ID',
      customizedProperties: [
        {
          "name": "age",
          "value": '18'
        }
      ]
    }
    ...
  },
  reactOptions: {
    useCamelCaseFlagKeys: false
  },
  deferInitialization: false
}
```
