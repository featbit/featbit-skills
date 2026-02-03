# Offline Mode and Bootstrap Data

## Offline Mode

To stop making remote calls to FeatBit:

```python
from fbclient.config import Config

config = Config(env_secret, event_url, streaming_url, offline=True)
```

In offline mode:
- No events are sent to the server.
- All evaluations return fallback values unless you provide external data.

The SDK allows you to populate flags and segments from a JSON string (see example file `tests/fbclient_test_data.json` in the SDK repository).

## Bootstrap JSON

You can populate flags and segments from JSON exported from FeatBit.

```shell
# replace http://localhost:5100 with your evaluation server url
curl -H "Authorization: <your-env-secret>" http://localhost:5100/api/public/sdk/server/latest-all > featbit-bootstrap.json
```

Then initialize the SDK with this JSON:

```python
# load json from file first
client.initialize_from_external_json(json)
```

The data format is defined by FeatBit and may change. Using FeatBit’s export endpoint is the recommended way to generate the file.
