---
name: featbit-python-sdk
description: Expert guidance for integrating FeatBit Python Server-Side SDK in backend applications. Use for Flask, Django, FastAPI, or any Python server application. Not for client-side use.
appliesTo:
  - "**/*.py"
  - "**/app.py"
  - "**/main.py"
  - "**/server.py"
  - "**/views.py"
  - "**/api/**/*.py"
  - "**/backend/**/*.py"
---

# FeatBit Python Server SDK Expert

Expert knowledge for integrating FeatBit Server-Side SDK in Python applications.

**Source**: https://github.com/featbit/featbit-python-sdk

⚠️ **Important**: This is a **server-side SDK** for multi-user systems (web servers, APIs). Not for client-side use.

## Data Synchronization

- **WebSocket Streaming** (recommended) or **Polling** for real-time sync
- Data stored in memory by default
- Changes pushed to SDK in <100ms average
- Auto-reconnects after internet outage

## Installation

```bash
pip install featbit-python-sdk
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```python
from fbclient import get, set_config
from fbclient.config import Config
from fbclient.common_types import FBUser

# Configure SDK
env_secret = 'your_env_secret'
streaming_url = 'wss://app-eval.featbit.co'
events_url = 'https://app-eval.featbit.co'

config = Config(env_secret, streaming_url, events_url)
set_config(config)

# Get client instance
client = get()

# Wait for initialization
if client.initialize():
    # Create user
    user = FBUser.new_user('user-key-123')
    user.with_name('John Doe')
    user.with_custom_property('role', 'admin')
    user.with_custom_property('age', 25)
    
    # Evaluate flag
    flag_value = client.variation('flag-key', user, 'default')
    print(f'Flag value: {flag_value}')
    
    # Close client
    client.close()
else:
    print('SDK failed to initialize')
```

## FBUser Builder

```python
from fbclient.common_types import FBUser

# Basic user
user = FBUser.new_user('user-123')
user.with_name('Jane Doe')

# User with custom properties
user = (FBUser.new_user('user-456')
    .with_name('Bob Wilson')
    .with_custom_property('role', 'admin')
    .with_custom_property('country', 'US')
    .with_custom_property('subscription', 'premium')
    .with_custom_property('age', 30))
```

## Flask Integration

```python
from flask import Flask, request, jsonify, g
from fbclient import get, set_config
from fbclient.config import Config
from fbclient.common_types import FBUser

app = Flask(__name__)

# Initialize FeatBit
env_secret = 'your_env_secret'
config = Config(env_secret, 'wss://app-eval.featbit.co', 'https://app-eval.featbit.co')
set_config(config)
fb_client = get()

# Wait for initialization
@app.before_first_request
def initialize_sdk():
    if not fb_client.initialize():
        app.logger.error('FeatBit SDK failed to initialize')

# Middleware to create user context
@app.before_request
def create_user_context():
    user_id = request.headers.get('X-User-Id') or 'anonymous'
    g.fb_user = (FBUser.new_user(user_id)
        .with_name(request.headers.get('X-User-Name'))
        .with_custom_property('role', request.headers.get('X-User-Role')))

# Route with feature flag
@app.route('/api/feature')
def feature_endpoint():
    is_enabled = fb_client.variation('new-feature', g.fb_user, False)
    
    if not is_enabled:
        return jsonify({'error': 'Feature not available'}), 403
    
    return jsonify({'data': 'feature data'})

# Gradual rollout example
@app.route('/api/checkout', methods=['POST'])
def checkout():
    use_new_checkout = fb_client.variation('new-checkout-flow', g.fb_user, False)
    
    if use_new_checkout:
        result = process_new_checkout(request.json)
    else:
        result = process_old_checkout(request.json)
    
    return jsonify(result)

# A/B testing example
@app.route('/api/purchase', methods=['POST'])
def purchase():
    variant = fb_client.variation('pricing-strategy', g.fb_user, 'standard')
    
    price = calculate_price(request.json, variant)
    result = process_purchase(price)
    
    if result['success']:
        # Track conversion
        fb_client.track_metric(g.fb_user, 'purchase-completed', price)
    
    return jsonify(result)

# Graceful shutdown
@app.teardown_appcontext
def shutdown_sdk(exception=None):
    fb_client.close()

if __name__ == '__main__':
    app.run(debug=True)
```

## Django Integration

```python
# settings.py
FEATBIT_CONFIG = {
    'env_secret': 'your_env_secret',
    'streaming_url': 'wss://app-eval.featbit.co',
    'events_url': 'https://app-eval.featbit.co'
}

# apps.py
from django.apps import AppConfig
from fbclient import get, set_config
from fbclient.config import Config

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        from django.conf import settings
        
        config = Config(
            settings.FEATBIT_CONFIG['env_secret'],
            settings.FEATBIT_CONFIG['streaming_url'],
            settings.FEATBIT_CONFIG['events_url']
        )
        set_config(config)
        
        fb_client = get()
        if not fb_client.initialize():
            print('FeatBit SDK failed to initialize')

# middleware.py
from fbclient import get
from fbclient.common_types import FBUser

class FeatBitMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.fb_client = get()
    
    def __call__(self, request):
        # Create user context
        user_id = request.user.id if request.user.is_authenticated else 'anonymous'
        request.fb_user = (FBUser.new_user(str(user_id))
            .with_name(request.user.username if request.user.is_authenticated else 'Anonymous')
            .with_custom_property('is_staff', request.user.is_staff if request.user.is_authenticated else False))
        
        response = self.get_response(request)
        return response

# views.py
from django.http import JsonResponse
from fbclient import get

def feature_view(request):
    fb_client = get()
    is_enabled = fb_client.variation('new-feature', request.fb_user, False)
    
    if not is_enabled:
        return JsonResponse({'error': 'Feature not available'}, status=403)
    
    return JsonResponse({'data': 'feature data'})

# Add middleware to settings.py
MIDDLEWARE = [
    # ... other middleware
    'myapp.middleware.FeatBitMiddleware',
]
```

## FastAPI Integration

```python
from fastapi import FastAPI, Request, HTTPException, Depends
from fbclient import get, set_config
from fbclient.config import Config
from fbclient.common_types import FBUser

app = FastAPI()

# Initialize FeatBit
@app.on_event("startup")
async def startup_event():
    env_secret = 'your_env_secret'
    config = Config(env_secret, 'wss://app-eval.featbit.co', 'https://app-eval.featbit.co')
    set_config(config)
    
    fb_client = get()
    if not fb_client.initialize():
        raise RuntimeError('FeatBit SDK failed to initialize')

@app.on_event("shutdown")
async def shutdown_event():
    fb_client = get()
    fb_client.close()

# Dependency to get user context
def get_fb_user(request: Request) -> FBUser:
    user_id = request.headers.get('x-user-id', 'anonymous')
    user = (FBUser.new_user(user_id)
        .with_name(request.headers.get('x-user-name', 'Anonymous'))
        .with_custom_property('role', request.headers.get('x-user-role', 'user')))
    return user

# Route with feature flag
@app.get("/api/feature")
async def feature_endpoint(fb_user: FBUser = Depends(get_fb_user)):
    fb_client = get()
    is_enabled = fb_client.variation('new-feature', fb_user, False)
    
    if not is_enabled:
        raise HTTPException(status_code=403, detail='Feature not available')
    
    return {'data': 'feature data'}

# A/B testing example
@app.post("/api/recommendations")
async def get_recommendations(fb_user: FBUser = Depends(get_fb_user)):
    fb_client = get()
    algorithm = fb_client.variation('recommendation-algorithm', fb_user, 'default')
    
    recommendations = generate_recommendations(algorithm)
    
    # Track which algorithm was used
    fb_client.track_metric(fb_user, f'algorithm-{algorithm}')
    
    return {'recommendations': recommendations, 'algorithm': algorithm}
```

## Flag Evaluation

### Boolean Variation

```python
is_enabled = client.variation('flag-key', user, False)
```

### String Variation

```python
theme = client.variation('theme', user, 'default')
```

### Number Variation

```python
max_retries = client.variation('max-retries', user, 3)
```

### JSON Variation

```python
import json

config_json = client.variation('config', user, '{}')
config = json.loads(config_json)
```

### With Evaluation Detail

```python
detail = client.variation_detail('flag-key', user, 'default')
print(f'Value: {detail.value}')
print(f'Reason: {detail.reason}')
print(f'Kind: {detail.kind}')
```

### Get All Flags

```python
all_flags = client.get_all_latest_flag_variations(user)
print(all_flags)
```

## Event Tracking

```python
# Simple event
client.track_metric(user, 'button-clicked')

# Event with numeric value
client.track_metric(user, 'purchase-completed', 99.99)

# For A/B testing
client.track_metric(user, 'conversion-event')

# Flush events manually
client.flush()
```

## Configuration Options

```python
from fbclient.config import Config

config = Config(
    env_secret='your_env_secret',
    event_url='https://app-eval.featbit.co',
    streaming_url='wss://app-eval.featbit.co',
    
    # Optional settings
    start_wait=5,              # Wait time for initialization in seconds
    max_retries=3,             # Max retry attempts
    use_streaming=True,        # Use WebSocket streaming (recommended)
    offline=False              # Set to True for offline mode
)
```

## Offline Mode

```python
from fbclient.config import Config
import json

# Enable offline mode
config = Config(
    env_secret='your_env_secret',
    offline=True
)

# Load flags from file
with open('feature_flags.json', 'r') as f:
    flags_data = json.load(f)

# Use flags locally
# Note: In offline mode, flags won't update from server
```

## Best Practices

### 1. Single Client Instance

```python
# ✅ Good: Create once at app startup
from fbclient import get, set_config

config = Config(env_secret, streaming_url, events_url)
set_config(config)
fb_client = get()  # Reuse this instance

# ❌ Bad: Creating per request
def handle_request():
    client = get()  # Don't do this repeatedly without proper setup
```

### 2. Wait for Initialization

```python
# ✅ Good: Wait before serving traffic
fb_client = get()
if not fb_client.initialize():
    raise RuntimeError('SDK initialization failed')

# Start server after initialization
app.run()

# ❌ Bad: Start immediately
fb_client = get()
app.run()  # Flags may return defaults
```

### 3. Graceful Shutdown

```python
import atexit

def cleanup():
    fb_client = get()
    fb_client.close()  # Flush events

atexit.register(cleanup)
```

### 4. Error Handling

```python
try:
    is_enabled = fb_client.variation('flag-key', user, False)
    # Use feature
except Exception as e:
    print(f'Flag evaluation failed: {e}')
    # Fall back to default
    is_enabled = False
```

### 5. User Context

```python
# ✅ Good: Use stable identifiers
user_id = current_user.id  # From auth system
user = FBUser.new_user(str(user_id))

# ✅ Good: Add relevant properties
user = (FBUser.new_user(str(user_id))
    .with_name(current_user.name)
    .with_custom_property('role', current_user.role)
    .with_custom_property('plan', current_user.subscription))

# ❌ Bad: Random IDs
import random
user = FBUser.new_user(str(random.random()))  # Don't do this
```

## Common Use Cases

### Progressive Rollout

```python
@app.route('/api/new-feature')
def new_feature():
    use_new_version = fb_client.variation('new-api-version', g.fb_user, False)
    
    if use_new_version:
        return jsonify(new_version_handler())
    else:
        return jsonify(old_version_handler())
```

### A/B Testing

```python
@app.route('/api/recommendations')
def recommendations():
    algorithm = fb_client.variation('recommendation-algorithm', g.fb_user, 'default')
    
    recommendations = get_recommendations(g.fb_user, algorithm)
    
    # Track which algorithm was used
    fb_client.track_metric(g.fb_user, f'algorithm-{algorithm}')
    
    return jsonify(recommendations)
```

### Remote Configuration

```python
import json

config_json = fb_client.variation('api-config', user, json.dumps({
    'timeout': 30,
    'retries': 3,
    'rate_limit': 1000
}))

config = json.loads(config_json)

# Use configuration
timeout = config['timeout']
retries = config['retries']
```

### Feature Toggle

```python
@app.before_request
def check_maintenance():
    system_user = FBUser.new_user('system')
    maintenance_mode = fb_client.variation('maintenance-mode', system_user, False)
    
    if maintenance_mode and request.path != '/health':
        return jsonify({'error': 'Service temporarily unavailable'}), 503
```

## Threading and Async

### Threading Support

```python
import threading
from fbclient import get

def worker_thread():
    fb_client = get()  # Thread-safe
    user = FBUser.new_user('worker-1')
    
    flag_value = fb_client.variation('worker-feature', user, False)
    print(f'Worker flag: {flag_value}')

thread = threading.Thread(target=worker_thread)
thread.start()
```

### Async/Await (Python 3.7+)

```python
import asyncio
from fbclient import get

async def async_handler():
    fb_client = get()
    user = FBUser.new_user('async-user')
    
    # Note: SDK operations are synchronous
    # Run in executor if needed
    loop = asyncio.get_event_loop()
    flag_value = await loop.run_in_executor(
        None,
        fb_client.variation,
        'async-feature',
        user,
        False
    )
    
    return flag_value
```

## Troubleshooting

### SDK Not Initializing

```python
fb_client = get()
if not fb_client.initialize():
    # Check:
    # 1. env_secret is correct
    # 2. Network connectivity
    # 3. Firewall rules for WebSocket
    print('SDK initialization failed')
```

### Connection Issues

```python
# Increase timeout
config = Config(
    env_secret=env_secret,
    streaming_url=streaming_url,
    events_url=events_url,
    start_wait=10  # 10 seconds
)
```

### Events Not Sent

```python
import atexit

def flush_events():
    fb_client = get()
    fb_client.flush()

# Register cleanup
atexit.register(flush_events)
```

## Performance Optimization

### Caching User Context

```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_cached_user(user_id):
    return FBUser.new_user(user_id)
```

### Batch Operations

```python
def get_feature_config(user):
    fb_client = get()
    
    return {
        'feature1': fb_client.variation('feature-1', user, False),
        'feature2': fb_client.variation('feature-2', user, False),
        'feature3': fb_client.variation('feature-3', user, 'default')
    }
```

## Testing

```python
# test_features.py
import unittest
from fbclient import get, set_config
from fbclient.config import Config
from fbclient.common_types import FBUser

class TestFeatureFlags(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        # Use test environment
        config = Config(
            'test-env-secret',
            'wss://test-eval.featbit.co',
            'https://test-eval.featbit.co'
        )
        set_config(config)
        cls.fb_client = get()
        cls.fb_client.initialize()
    
    def test_feature_enabled(self):
        user = FBUser.new_user('test-user')
        is_enabled = self.fb_client.variation('test-feature', user, False)
        self.assertIsInstance(is_enabled, bool)
    
    @classmethod
    def tearDownClass(cls):
        cls.fb_client.close()
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-python-sdk
- **PyPI**: https://pypi.org/project/featbit-python-sdk/
- **Examples**: https://github.com/featbit/featbit-samples
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/python
