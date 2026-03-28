---
name: featbit-sdks-python
description: Expert guidance for integrating FeatBit Python Server SDK in backend applications. Use when user asks about "Python SDK", "Python feature flags", "FeatBit Python", fbclient usage, or mentions .py files, Flask, Django, FastAPI, or scripts with FeatBit. Do not use for Node.js, Java, Go, .NET, React, or any browser/client-side questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit Python Server SDK

Integrate FeatBit feature flags into server-side Python applications — Flask, Django, FastAPI, background workers, or scripts. Flags sync via WebSocket on startup (<100ms) and evaluate locally with no remote call per request. Do not use for browser JavaScript, React, React Native, or any frontend context.

## Execution Procedure

```
1. INSTALL package
   pip install fb-python-sdk

2. CONFIGURE singleton client
   Call set_config() once at application startup
   Call get() to obtain the process-wide singleton

3. EVALUATE first feature flag
   Check client.initialize (property, no parens) before evaluating
   Call variation() or variation_detail() with user dict and flag key

4. INTEGRATE into framework
   Flask: init in create_app(), store on app.config
   Django: init in AppConfig.ready()
   FastAPI: init in lifespan handler
   Scripts: init at module top-level

5. CLOSE client on shutdown
   Call client.stop() to flush pending analytics events
```

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Configure the singleton client
- [ ] Step 3: Evaluate the first feature flag
- [ ] Step 4: Shut down cleanly

**Step 1: Install the package**

Run:

```bash
pip install fb-python-sdk
```

**Step 2: Configure the singleton client**

```python
from fbclient import get, set_config
from fbclient.config import Config

set_config(Config(
    env_secret='<your-env-secret>',
    event_url='http://localhost:5100',
    streaming_url='ws://localhost:5100'
))
client = get()
```

**Why `set_config` + `get()`**: This initializes a process-wide singleton. Call `set_config` once at application startup; every subsequent `get()` returns the same instance.

**Step 3: Evaluate the first feature flag**

```python
if client.initialize:
    user = {'key': 'user-key-123', 'name': 'Jane'}
    detail = client.variation_detail('flag-key', user, default=None)
    print(f'flag returns {detail.variation}, reason: {detail.reason}')
```

**Why check `client.initialize`**: This property (no parentheses) confirms the SDK has synced flag data. If false, `env_secret`, `event_url`, or `streaming_url` is likely wrong.

**Step 4: Shut down cleanly**

```python
client.stop()
```

Call `client.stop()` when the process exits to flush pending analytics events.

## Feature Flag Evaluation

```python
user = {'key': 'user-key-123', 'name': 'Jane'}

# Value only
flag_value = client.variation('flag-key', user, False)

# Value + reason
detail = client.variation_detail('flag-key', user, default=None)
print(detail.variation)  # the resolved value
print(detail.reason)     # why this value was returned

# All flags for a user
all_flags = client.get_all_latest_flag_variations(user)
```

Call `variation` when only the resolved value is needed. Call `variation_detail` when the caller needs to know why a specific value was returned (targeting rule, default, etc.).

## User Custom Properties

Represent a user as a plain `dict`. Add any extra keys directly — no builder class is required:

```python
user = {
    'key': 'bob-id',       # required: unique identifier
    'name': 'Bob',         # required: display name
    'age': 25,             # custom property
    'country': 'FR',       # custom property
    'plan': 'premium',     # custom property
}
flag_value = client.variation('flag-key', user, False)
```

Reference any key in this dict from FeatBit targeting rules.

