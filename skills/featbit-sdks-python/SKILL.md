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

## When to Use This Skill

Use for server-side Python applications — Flask, Django, FastAPI, background workers, or scripts — that evaluate feature flags on the backend.

**Why server SDK**: Flags sync via WebSocket on startup (<100ms) and are evaluated locally with no remote call per request. The process holds one persistent singleton client.

Do not use for browser JavaScript, React, React Native, or any frontend context.

## Source

https://github.com/featbit/featbit-python-sdk

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

Use `variation` when only the resolved value is needed. Use `variation_detail` when the caller needs to know why a specific value was returned (targeting rule, default, etc.).

## User Custom Properties

The Python SDK represents a user as a plain `dict`. Add any extra keys directly — no builder class is required:

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

FeatBit targeting rules can reference any key in this dict.

## Read Next Only When Needed

- [Flag tracking / listeners](https://github.com/featbit/featbit-python-sdk#flag-tracking) — only when the user asks about reacting to flag changes at runtime.
- [Experiments / A-B testing](https://github.com/featbit/featbit-python-sdk#experiments-abn-testing) — only when the user asks about `track_metric` or experiment events.
- [Offline mode](https://github.com/featbit/featbit-python-sdk#offline-mode) — only when the user asks about `Config(..., offline=True)` or bootstrapping without a server.
- [Data synchronization](https://github.com/featbit/featbit-python-sdk#data-synchonization) — only when the user asks about WebSocket reconnect or sync behavior.
- [FBClient reference](https://github.com/featbit/featbit-python-sdk#fbclient) — only when the user asks about advanced client configuration or lifecycle options.
