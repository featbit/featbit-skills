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

## When to Use This Skill

Use for server-side Go applications — HTTP servers, background workers, CLI tools — that need real-time feature flag evaluation.

Why server-side SDK: maintains one persistent WebSocket connection, synchronizes all flag data locally in under 100 ms, then evaluates every flag locally in under 10 ms per call on average. Do not use for frontend JavaScript or mobile applications — those require a client-side SDK.

## Source

https://github.com/featbit/featbit-go-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the module
- [ ] Step 2: Create the client with secret and URLs
- [ ] Step 3: Build the user and evaluate the first feature flag
- [ ] Step 4: If initialization fails, validate configuration and retry
- [ ] Step 5: Read advanced README sections only when needed

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
fmt.Printf("flag %s returns %s, reason: %s\n", detail.KeyName, detail.Variation, detail.Reason)
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

Use the first return value when only the flag result is needed. Use `detail` when the evaluation reason is also needed — `detail.Reason` explains why the value was returned, `detail.KeyName` and `detail.Variation` identify the matched variation.

Also available for non-boolean flags: `Variation` (string), `IntVariation`, `DoubleVariation`, `JsonVariation`.

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

## Read Next Only When Needed

- Read the official README section for [evaluation](https://github.com/featbit/featbit-go-sdk#evaluation) when the user asks how to evaluate a feature flag, use non-boolean variation types (String, Int, Double, Json), inspect evaluation details, or call `AllLatestFlagsVariations` to get all flags for a user at once.
- Read the official README section for [FBUser](https://github.com/featbit/featbit-go-sdk#fbuser) when the user asks about user attributes, custom properties, targeting fields, or user construction patterns.
- Read the official README sections for [FBClient](https://github.com/featbit/featbit-go-sdk#fbclient), [FBConfig and components](https://github.com/featbit/featbit-go-sdk#fbconfig-and-components), [offline mode](https://github.com/featbit/featbit-go-sdk#offline-mode), and [experiments](https://github.com/featbit/featbit-go-sdk#experiments-abn-testing) only when those topics are requested.
