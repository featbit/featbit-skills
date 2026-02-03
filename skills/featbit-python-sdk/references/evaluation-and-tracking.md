# Evaluation and Tracking

## User Model

Use a dictionary with required `key` and `name` fields. Add custom properties as needed.

```python
user = {"key": user_key, "name": user_name, "age": age}
```

## Flag Evaluation

```python
if client.initialize:
    user = {"key": user_key, "name": user_name, "age": age}
    flag_value = client.variation(flag_key, user, default_value)
    detail = client.variation_detail(flag_key, user, default=None)
```

If evaluation happens before initialization or with invalid keys, the SDK returns the default value. `variation_detail()` includes the reason.

## All Flags for a User

```python
if client.initialize:
    user = {"key": user_key, "name": user_name}
    all_flag_values = client.get_all_latest_flag_variations(user)
    for flag_key in all_flag_values.keys():
        detail = all_flag_values.get(flag_key, default=None)
        value = all_flag_values.get_variation(flag_key, default=None)
```

`get_all_latest_flag_variations()` returns `AllFlagStates`. Use `get()` for details and `get_variation()` for values.

## Flag Change Tracking

The SDK can notify you when flag configurations change, or when a specific user’s flag value changes.

```python
if client.initialize:
    client.flag_tracker.add_flag_value_maybe_changed_listener(
        flag_key, user, flag_value_maybe_changed_callback_fn
    )
    client.flag_tracker.add_flag_value_changed_listener(
        flag_key, user, flag_value_changed_callback_fn
    )
```

- `maybe_changed`: the flag’s configuration may have changed.
- `changed`: the evaluated value for the user definitely changed.

The callback function receives two arguments: the flag key and the latest flag value.

Change notices only work when the SDK is connected to FeatBit. In offline mode, the SDK cannot detect changes.

## Experiments (A/B/n Testing)

Track custom metrics for experiments:

```python
client.track_metric(user, event_name, numeric_value)
```

`numeric_value` defaults to `1` if omitted. Call `track_metric()` after evaluating the related feature flag to ensure it is included in experiment results.
