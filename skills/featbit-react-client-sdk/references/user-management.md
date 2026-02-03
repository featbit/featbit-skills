# Switch User After Initialization

If the user is not available during the initialization, you can call the **identity()** method on **fbClient** to set the user after initialization.

## Switch user in class component

```javascript
import { withFbConsumer } from '@featbit/react-client-sdk';

class LoginComponent extends React.Component {
    
  handleLogin = async () => {
    const { fbClient } = this.props;
    const user = {}; // use your user object
    await fbClient.identify(user);
  }
    
  render() {
    return (
      <div>
        <button onClick={this.handleLogin}>Login</button>
      </div>
    );
  }
}

export default withFbConsumer()(LoginComponent)
```

## Switch user in function component

```javascript
import { useFbClient } from '@featbit/react-client-sdk';

const LoginComponent = () => {
  const fbClient = useFbClient();

  const handleLogin = async () => {
    const user = {}; // use your user object
    await fbClient.identify(user);
  }
    
  return (
    <div>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}
```
