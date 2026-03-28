---
name: featbit-sdks-go
description: Expert guidance for integrating FeatBit Go Server SDK. Use when user asks about "Go SDK", "Golang feature flags", "FeatBit Go", or mentions .go files with FeatBit. Do not use for Node.js, Python, Java, .NET, or JavaScript questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit Go Server SDK

Server-side Go SDK for real-time feature flag evaluation in HTTP servers, background workers, and CLI tools. Maintains one persistent WebSocket connection, synchronizes all feature flag data locally in under 100 ms, then evaluates every feature flag locally in under 10 ms per call on average. Do not use for frontend JavaScript or mobile applications — those require a client-side SDK.

## Execution Procedure

```
1. Install module
   go get github.com/featbit/featbit-go-sdk

2. Build FBClient
   client = NewFBClient(envSecret, streamingUrl, eventUrl)
   IF err → validate config, retry

3. Evaluate first feature flag
   user = NewUserBuilder(key).UserName(name).Build()
   value, detail = client.BoolVariation(flagKey, user, fallback)
   verify value ≠ fallback (confirms SDK connected)

4. Integrate into app
   call *Variation methods at decision points
   add Custom() properties when targeting rules need them

5. Close client on shutdown
   defer client.Close()
```

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the module
- [ ] Step 2: Create the client with secret and URLs
- [ ] Step 3: Build the user and evaluate the first feature flag
- [ ] Step 4: If initialization fails, validate configuration and retry

**Step 1: Install the module**

Run:

```bash
go get github.com/featbit/featbit-go-sdk
```

**Step 2: Create the client**

Use this minimal setup:

```go
client, err := featbit.NewFBClient(envSecret, streamingUrl, eventUrl)
if err != nil {
  return err
}
defer client.Close()
```

**Step 3: Evaluate the first feature flag**

Use the official pattern:

```go
user, _ := interfaces.NewUserBuilder("<unique-user-key>").UserName("Jane").Build()
_, detail, _ := client.BoolVariation("flag-key", user, false)
fmt.Printf("feature flag %s returns %s, reason: %s\n", detail.KeyName, detail.Variation, detail.Reason)
```

**Step 4: Validate the integration**

If `client.IsInitialized()` is false or evaluation returns the fallback unexpectedly, verify `envSecret`, `streamingUrl`, and `eventUrl`, then retry.

## Feature Flag Evaluation

After the client is initialized, evaluate a feature flag with a user and a fallback value:

```go
user, _ := interfaces.NewUserBuilder("<unique-user-key>").UserName("Jane").Build()
value, detail, _ := client.BoolVariation("flag-key", user, false)
fmt.Println(value, detail.Reason)
```

Use the first return value when only the feature flag result is needed. Use `detail` when the evaluation reason is also needed — `detail.Reason` explains why the value was returned, `detail.KeyName` and `detail.Variation` identify the matched variation.

Also available for non-boolean feature flags: `Variation` (string), `IntVariation`, `DoubleVariation`, `JsonVariation`.

## User Custom Properties

Add custom properties to `FBUser` when targeting rules depend on user attributes beyond `key` and `userName`:

```go
user, _ := interfaces.NewUserBuilder("a-unique-key-of-user").
    UserName("bob").
    Custom("age", "15").
    Custom("country", "FR").
    Build()
```

Use `Custom(key, value)` for any attribute that must be referenced in feature flag targeting rules.

