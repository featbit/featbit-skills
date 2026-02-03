# Quick Start (Full)

## Installation

```shell
pip install fb-python-sdk
```

Compatible with Python 3.6 through 3.11.

## Prerequisites

Before using the SDK, obtain the environment secret and SDK URLs:

- Environment secret: https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret
- SDK URLs: https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls

## Basic Usage

> The `env_secret`, `streaming_url`, and `event_url` are required to initialize the SDK.

```python
from fbclient import get, set_config
from fbclient.config import Config

env_secret = "<replace-with-your-env-secret>"
event_url = "http://localhost:5100"
streaming_url = "ws://localhost:5100"

set_config(Config(env_secret, event_url, streaming_url))
client = get()

if client.initialize:
    flag_key = "<replace-with-your-flag-key>"
    user = {"key": "bot-id", "name": "bot"}
    detail = client.variation_detail(flag_key, user, default=None)
    print(
        f"flag {flag_key} returns {detail.variation} for user {user['key']}, reason: {detail.reason}"
    )

# ensure events are flushed before exiting
client.stop()
```

## Example Project

- Python demo: https://github.com/featbit/featbit-samples/blob/main/samples/dino-game/demo-python/demo_python.py
