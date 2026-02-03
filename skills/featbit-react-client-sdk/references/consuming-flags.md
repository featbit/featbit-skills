# Consuming Feature Flags

## Consuming flags in class component

There are two ways to consume the flags.

### Using contextType property
```javascript
import React from 'react';
import { context } from '@featbit/react-client-sdk';

class MyComponent extends React.Component {
  static contextType = context;

  constructor(props) {
    super(props);
  }

  render() {
    const { flags, fbClient } = this.context;

    return (
      <div>{flags['dev-test-flag'] ? 'Flag on' : 'Flag off'}</div>
    );
  }
}

export default MyComponent;
```

###  Using withFbConsumer

The return value of withFbConsumer is a wrapper function that takes your component and returns a React component injected with flags & fbClient as props.

```javascript
import { withFbConsumer } from '@featbit/react-client-sdk';

class MyComponent extends React.Component {
  render() {
    const { flags } = this.props;

    return (
      <div>
        <div>{flags.flag1}</div>
      </div>
    );
  }
}

export default withFbConsumer()(MyComponent)
```

## Consuming flags in function component
There are two ways to consume the flags.

### Using withFbConsumer
The return value of withFbConsumer is a wrapper function that takes your component and returns a React component injected with flags & fbClient as props.

```javascript
import { withFbConsumer } from '@featbit/react-client-sdk';

const Home = ({ flags, fbClient /*, ...otherProps */ }) => {
  // You can call any of the methods from the JavaScript SDK
  // fbClient.identify({...})

  return flags['dev-test-flag'] ? <div>Flag on</div> : <div>Flag off</div>;
};

export default withFbConsumer()(Home);
```

### Using Hooks

The React SDK offers two custom hooks which you can use as an alternative to `withFbConsumer`: 
- `useFlags`
- `useFbClient`

**Note:** These functions require React 16.8.0 or later. useFlags and useFbClient use the Hooks API, which requires React version 16.8.0 or later.

useFlags is a custom hook which returns all feature flags. It uses the useContext primitive to access the FeatBit context set up by asyncWithFbProvider or withFbProvider. You still must use the asyncWithFbProvider or the withFbProvider higher-order component at the root of your application to initialize the React SDK and populate the context with fbClient and your flags.

useFbClient is the second custom hook which returns the underlying FeatBit's JavaScript SDK client object. Like the useFlags custom hook, useFbClient also uses the useContext primitive to access the context set up by asyncWithFbProvider or withFbProvider. You still must use the asyncWithFbProvider or the withFbProvider higher-order component to initialize the React SDK to use this custom hook.

Here is an example of how to use those two hooks:

```javascript
import { useFlags, useFbClient } from '@featbit/react-client-sdk';

const MyComponent = props => {
  const fbClient = useFbClient();

  const { flag1, flag2 } = useFlags();
  // or use
  const flags = useFlags();
  // then use flags.flag1 or flags['flag1'] to reference the flag1 feature flag
  
  return (
    <div>
      <div>1: {flag1}</div>
      <div>2: {flag2}</div>
    </div>
  );
};

export default MyComponent;
```
