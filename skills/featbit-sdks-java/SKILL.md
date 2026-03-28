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

Use for server-side Java applications — Spring Boot, Jakarta EE, standalone services — that need real-time feature flag evaluation. Requires Java SE 8 or above. Maintains one persistent WebSocket connection, synchronizes all feature flag data locally in under 100 ms, then evaluates every feature flag locally in under 10 ms per call on average. Do not use for Android or client-side JavaScript — those require a client-side SDK.

## Execution Procedure

```
1. Add Maven or Gradle dependency (co.featbit:featbit-java-sdk)
2. Build FBConfig with streaming + event URLs → create FBClient
3. Verify client.isInitialized() == true
4. Build FBUser → evaluate first feature flag with boolVariation
5. Add .custom() properties to FBUser when targeting rules need them
6. Call client.close() on application shutdown
```

## Setup

**Add the dependency** — pick one build tool:

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

**Create the client:**

```java
FBConfig config = new FBConfig.Builder()
    .streamingURL("ws://localhost:5100")
    .eventURL("http://localhost:5100")
    .build();

FBClient client = new FBClientImp("<your-env-secret>", config);
```

If `client.isInitialized()` is false, verify the environment secret and both URLs, then retry. Call `client.close()` during application shutdown.

## Feature Flag Evaluation

Build an `FBUser` and evaluate a feature flag with a fallback value:

```java
FBUser user = new FBUser.Builder("<unique-user-key>")
    .userName("Jane")
    .build();

// Feature flag value only
Boolean value = client.boolVariation("flag-key", user, false);

// Feature flag value with evaluation detail
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
