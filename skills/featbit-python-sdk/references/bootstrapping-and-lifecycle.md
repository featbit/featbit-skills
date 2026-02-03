# Bootstrapping and Client Lifecycle

Applications should create a single `FBClient` instance per environment and keep it for the lifetime of the application. Use one client for each FeatBit environment when necessary.

## Singleton Pattern (Recommended)

```python
from fbclient.config import Config
from fbclient import get, set_config

set_config(Config(env_secret, event_url, streaming_url))
client = get()

if client.initialize:
    # the client is ready
    pass
```

## Direct Construction

The constructor initializes the SDK and attempts to connect. It returns when the connection succeeds or the timeout is reached (default: 15 seconds). If it times out before connecting, the client is uninitialized and returns default values, but it keeps trying to connect in the background unless a network error occurs or you call `stop()`.

```python
from fbclient.config import Config
from fbclient.client import FBClient

client = FBClient(Config(env_secret, event_url, streaming_url), start_wait=15)

if client.initialize:
    # the client is ready
    pass
```

You can detect readiness with `initialize` (property) or by waiting for the status provider.

## Asynchronous Readiness Check

If you want the constructor to return immediately, set `start_wait=0` and wait on the status provider:

```python
from fbclient.config import Config
from fbclient.client import FBClient

client = FBClient(Config(env_secret), start_wait=0)
if client.update_status_provider.wait_for_OKState():
    # the client is ready
    pass
```

`wait_for_OKState()` accepts an optional timeout in seconds. If it times out, it returns `False` and the client remains uninitialized while it continues trying to connect in the background.
