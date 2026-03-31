---
name: featbit-sdks-java
description: Expert guidance for integrating FeatBit Java Server SDK. Use when user asks about "Java SDK", "Java feature flags", "FeatBit Java", "Maven feature flags", or mentions .java or build.gradle files with FeatBit. Do not use for .NET, Go, Node.js, Python, or JavaScript questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit Java Server SDK

## When to Use This Skill

Use for server-side Java applications — Spring Boot, Jakarta EE, standalone services — that need real-time feature flag evaluation. Requires Java SE 8 or above.

Why server-side SDK: maintains one persistent WebSocket connection, synchronizes all feature flag data locally in under 100 ms, then evaluates every feature flag locally in under 10 ms per call on average. Do not use for Android or client-side JavaScript — those require a client-side SDK.

## Source

https://github.com/featbit/featbit-java-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Add the dependency
- [ ] Step 2: Build `FBConfig` and create the client
- [ ] Step 3: Build `FBUser` and evaluate the first feature flag
- [ ] Step 4: Validate initialization and close the client
- [ ] Step 5: Read advanced README sections only when needed

**Step 1: Add the dependency**

Use one build tool:

```xml
<dependency>
  <groupId>co.featbit</groupId>
  <artifactId>featbit-java-sdk</artifactId>
  <version>1.4.5</version>
</dependency>
```

```groovy
implementation 'co.featbit:featbit-java-sdk:1.4.5'
```

**Step 2: Create the client**

Use this minimal setup:

```java
FBConfig config = new FBConfig.Builder()
    .streamingURL("ws://localhost:5100")   // replace with your FeatBit server URL
    .eventURL("http://localhost:5100")     // replace with your FeatBit server URL
    .build();

FBClient client = new FBClientImp("<your-env-secret>", config);
```

**Step 3: Evaluate the first feature flag**

Use the official pattern:

```java
FBUser user = new FBUser.Builder("<unique-user-key>").userName("Jane").build();

// Feature flag value only
Boolean flagValue = client.boolVariation("flag-key", user, false);
System.out.printf("feature flag returns %b for user %s%n", flagValue, user.getUserName());

// Feature flag value with evaluation detail
EvalDetail<Boolean> ed = client.boolVariationDetail("flag-key", user, false);
System.out.printf("feature flag returns %b, reason: %s%n", ed.getVariation(), ed.getReason());
```

**Step 4: Validate the integration**

If `client.isInitialized()` is false or evaluation returns the fallback unexpectedly, verify the secret and both URLs, then retry. Call `client.close()` during shutdown.

## Feature Flag Evaluation

After the client is initialized, evaluate a feature flag with a user and a fallback value:

```java
FBUser user = new FBUser.Builder("<unique-user-key>").userName("Jane").build();

// Value only
Boolean value = client.boolVariation("flag-key", user, false);

// Value with evaluation detail
EvalDetail<Boolean> detail = client.boolVariationDetail("flag-key", user, false);
System.out.printf("returns %b, reason: %s%n", detail.getVariation(), detail.getReason());
```

Use `boolVariation` when only the feature flag value is needed. Use `boolVariationDetail` when the evaluation reason is also needed — `detail.getReason()` explains why the variation was returned.

Also available for non-boolean feature flags: `variation`/`variationDetail` (string), `intVariation`/`intVariationDetail`, `longVariation`/`longVariationDetail`, `doubleVariation`/`doubleVariationDetail`, `jsonVariation`/`jsonVariationDetail`.

## User Custom Properties

Add custom properties to `FBUser` when targeting rules depend on user attributes beyond `key` and `userName`:

```java
FBUser user = new FBUser.Builder("a-unique-key-of-user")
    .userName("bob")
    .custom("age", "15")
    .custom("country", "FR")
    .build();
```

Use `.custom(key, value)` for any attribute that must be referenced in feature flag targeting rules.

## Read Next Only When Needed

- Read the official README section for [evaluation](https://github.com/featbit/featbit-java-sdk#evaluation) when the user asks how to evaluate a feature flag, use non-boolean typed variants, inspect evaluation detail, or call `getAllLatestFlagsVariations` to get all feature flags for a user at once.
- Read the official README section for [FBUser](https://github.com/featbit/featbit-java-sdk#fbuser) when the user asks about user attributes, custom properties, targeting fields, or user construction patterns.
- Read the official README section for [flag tracking](https://github.com/featbit/featbit-java-sdk#flag-tracking) when the user asks how to listen for feature flag value changes at runtime.
- Read the official README sections for [FBClient](https://github.com/featbit/featbit-java-sdk#fbclient), [FBConfig and components](https://github.com/featbit/featbit-java-sdk#fbconfig-and-components), [offline mode](https://github.com/featbit/featbit-java-sdk#offline-mode), and [experiments](https://github.com/featbit/featbit-java-sdk#experiments-abn-testing) only when those topics are requested.
