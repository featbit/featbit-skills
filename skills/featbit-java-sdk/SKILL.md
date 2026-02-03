---
name: featbit-java-sdk
description: Integrates FeatBit feature flags in Java server applications (Spring Boot, Servlet, JAX-RS). Use when working with .java files, pom.xml, build.gradle, or when user asks about "Java SDK", "Spring Boot integration", "feature flags in Java", "OpenFeature Java", or "FeatBit Java".
appliesTo:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/src/main/**/*.java"
  - "**/application.properties"
  - "**/application.yml"
---

# FeatBit Java Server SDK

Expert guidance for integrating FeatBit Server-Side SDK in Java applications.

📚 **Official Repository**: https://github.com/featbit/featbit-java-sdk

> **For complete documentation and latest updates**, visit the [official GitHub repository](https://github.com/featbit/featbit-java-sdk).

## Overview

⚠️ **Server-Side SDK**: Designed for multi-user systems (web servers, APIs). Not for Android - use Android SDK for mobile.

**Data Synchronization**:
- WebSocket for real-time sync
- In-memory storage (default)
- Changes pushed in <100ms average
- Auto-reconnects after internet outage

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Installation

**Version**: 1.4.5

### Maven

```xml
<dependency>
  <groupId>co.featbit</groupId>
  <artifactId>featbit-java-sdk</artifactId>
  <version>1.4.5</version>
</dependency>
```

### Gradle

```gradle
implementation 'co.featbit:featbit-java-sdk:1.4.5'
```

## Quick Start

```java
import co.featbit.commons.model.FBUser;
import co.featbit.commons.model.EvalDetail;
import co.featbit.server.FBClientImp;
import co.featbit.server.FBConfig;
import co.featbit.server.exterior.FBClient;
import java.io.IOException;

class Main {
    public static void main(String[] args) throws IOException {
        String envSecret = "<replace-with-your-env-secret>";
        String streamUrl = "ws://localhost:5100";
        String eventUrl = "http://localhost:5100";

        FBConfig config = new FBConfig.Builder()
                .streamingURL(streamUrl)
                .eventURL(eventUrl)
                .build();

        FBClient client = new FBClientImp(envSecret, config);
        if (client.isInitialized()) {
            String flagKey = "use-new-algorithm";

            FBUser user = new FBUser.Builder("bot-id")
                    .userName("bot")
                    .build();

            // Evaluate a boolean flag
            Boolean flagValue = client.boolVariation(flagKey, user, false);
            System.out.printf("flag %s, returns %b for user %s%n", flagKey, flagValue, user.getUserName());

            // Evaluate with detail
            EvalDetail<Boolean> ed = client.boolVariationDetail(flagKey, user, false);
            System.out.printf("flag %s, returns %b for user %s, reason: %s%n", flagKey, ed.getVariation(), user.getUserName(), ed.getReason());
        }

        // Close to ensure insights are sent
        client.close();
        System.out.println("APP FINISHED");
    }
}
```

📖 **Example**: https://github.com/featbit/featbit-samples/blob/main/samples/dino-game/demo-java/src/main/java/co/featbit/demo/JavaDemo.java

## FBClient

**CRITICAL**: Applications **SHOULD instantiate a single FBClient instance** for the lifetime of the application.

### Bootstrapping

```java
FBConfig config = new FBConfig.Builder()
    .streamingURL(streamUrl)
    .eventURL(eventUrl)
    .startWaitTime(Duration.ofSeconds(10))
    .build();

FBClient client = new FBClientImp(envSecret, config);
if(client.isInitialized()){
    // the client is ready
}
```

### Asynchronous Initialization

```java
FBConfig config = new FBConfig.Builder()
    .streamingURL(streamUrl)
    .eventURL(eventUrl)
    .startWaitTime(Duration.ZERO)
    .build();
FBClient client = new FBClientImp(envSecret, config);

// later, wait for initialization:
boolean inited = client.getDataUpdateStatusProvider().waitForOKState(Duration.ofSeconds(10));
if (inited) {
    // the client is ready
}
```

## FBConfig and Components

Required: `streamingURL`, `eventURL`

Optional:
- `startWaitTime`: Duration to block awaiting successful data sync
- `offline`: Set SDK offline (no connection to server)
- `disableEvents`: Disable sending events

```java
FBConfig config = new FBConfig.Builder()
        .streamingURL(streamUrl)
        .eventURL(eventUrl)
        .build();
```

### HttpConfigFactory

```java
import co.featbit.server.Factory;
import co.featbit.server.HttpConfigFactory;

HttpConfigFactory factory = Factory.httpConfigFactory()
        .connectTime(Duration.ofMillis(3000))
        .httpProxy("my-proxy", 9000);

FBConfig config = new FBConfig.Builder()
        .httpConfigFactory(factory)
        .build();
```

### DataStorageFactory

SDK uses `Factory#inMemoryDataStorageFactory()` by default. Developers can customize to persist data in Redis, MongoDB, etc.

### DataSynchronizerFactory

SDK uses `Factory#dataSynchronizerFactory()` for WebSocket streaming by default.

### InsightProcessorFactory

SDK uses `Factory#insightProcessorFactory()` for analytics events by default.

## FBUser

```java
FBUser user = new FBUser.Builder("key")
        .userName("name")
        .custom("property", "value")
        .build();
```

## Evaluation

SDK calculates flag values locally and synchronously. Average evaluation time is **< 10ms**.

### Variation Methods

- `variation` / `variationDetail` - String
- `boolVariation` / `boolVariationDetail` - Boolean
- `doubleVariation` / `doubleVariationDetail` - Double
- `longVariation` / `longVariationDetail` - Long
- `intVariation` / `intVariationDetail` - Integer
- `jsonVariation` / `jsonVariationDetail` - JSON

### Get All Flags

```java
AllFlagStates states = client.getAllLatestFlagsVariations(user);

// Get all flag keys
Collection<String> flagKeys = states.getFlagKeys();

// Get specific values
EvalDetail<String> detail = states.getStringDetail("flag key", "default");
String value = states.getString("flag key", "default");
```

### Get All Data as JSON

```java
String json = client.getAllDataAsJson();
```

## Flag Tracking

Register listeners for flag value changes:

```java
client.getFlagTracker().addFlagValueChangeListener(flagKey, user, event -> {
    // Called only if flag value changes
});
```

## Offline Mode

```java
FBConfig config = new FBConfig.Builder()
        .streamingURL(streamUrl)
        .eventURL(eventUrl)
        .offline(true)
        .build();

FBClient client = new FBClientImp(envSecret, config);
```

### Bootstrap from JSON

```java
FBClient client = new FBClientImp(envSecret, config);

String json = Resources.toString(Resources.getResource("featbit-bootstrap.json"), Charsets.UTF_8);
if(client.initFromJsonFile(json)){
    // the client is ready
}
```

### Generate Bootstrap JSON

```bash
curl -H "Authorization: <your-env-secret>" \
     http://localhost:5100/api/public/sdk/server/latest-all > featbit-bootstrap.json
```

## Experiments (A/B/n Testing)

We support automatic experiments for pageviews and clicks. For custom events:

```java
client.trackMetric(user, eventName, numericValue);
```

**numericValue** is optional (default: 1).

⚠️ **Important**: Call `trackMetric` **after** the related feature flag is evaluated.

## Best Practices

### 1. Single Client Instance

```java
// ✅ Good
@Bean
public FBClient fbClient() {
    return new FBClientImp(envSecret, config);
}

// ❌ Bad
@GetMapping("/feature")
public void checkFeature() {
    FBClient client = new FBClientImp(...); // Don't do this!
}
```

### 2. Wait for Initialization

```java
if (!fbClient.isInitialized()) {
    throw new RuntimeException("SDK not ready");
}
```

### 3. Graceful Shutdown

```java
@PreDestroy
public void cleanup() {
    if (fbClient != null) {
        fbClient.close();
    }
}
```

## OpenFeature Integration

FeatBit provides an [OpenFeature](https://openfeature.dev/) provider for Java server applications.

📚 **OpenFeature Provider Repository**: https://github.com/featbit/featbit-openfeature-provider-java-server

### Installation

```xml
<dependency>
    <groupId>co.featbit</groupId>
    <artifactId>featbit-openfeature-provider-java-server</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Usage

```java
import dev.openfeature.sdk.OpenFeatureAPI;
import co.featbit.openfeature.FeatBitProvider;

FBConfig config = new FBConfig.Builder()
        .streamingURL(STREAM_URL)
        .eventURL(EVENT_URL)
        .build();

// Synchronous
OpenFeatureAPI.getInstance().setProviderAndWait(new FeatBitProvider(ENV_SECRET, config));

// Asynchronous
OpenFeatureAPI.getInstance().setProvider(new FeatBitProvider(ENV_SECRET, config));
```

### Evaluation Context

**Required**: FeatBit requires a context with a targeting key for evaluation.

**Targeting Key**: Specify using `targetingKey` (OpenFeature standard), or `key`/`keyid` (FeatBit identifier). If both specified, `targetingKey` takes precedence.

**Name Attribute**: Used to search users quickly. If not set explicitly, FeatBit uses the targeting key as the name.

**Custom Attributes**: Only string type values are supported.

```java
EvaluationContext ctx = new ImmutableContext("user-key", new HashMap() {{
    put("name", new Value("user-name"));
    put("country", new Value("USA"));
}});
```

### Evaluation

```java
Client client = OpenFeatureAPI.getInstance().getClient();

// Evaluation Context
EvaluationContext evalCtx = new ImmutableContext("user-key", new HashMap() {{
    put("name", new Value("user-name"));
    put("country", new Value("USA"));
}});

// Evaluate a feature flag
String result = client.getStringValue(flagKey, defaultValue, evalCtx);

// Evaluate with details
FlagEvaluationDetails<String> details = client.getStringDetails(flagKey, defaultValue, evalCtx);
```

**Object Conversion**: When using `Client#getObjectValue` or `Client#getObjectDetails`:
1. SDK converts result to `Value` type according to default `Value`
2. If default value is `List` or `Structure` Value, SDK parses result as JSON object
3. Wrong type of default value may throw an exception

**More Info**: [OpenFeature Java Documentation](https://openfeature.dev/docs/reference/technologies/server/java)

## Additional Resources

- **GitHub Repository**: https://github.com/featbit/featbit-java-sdk
- **OpenFeature Provider**: https://github.com/featbit/featbit-openfeature-provider-java-server
- **Code Samples**: https://github.com/featbit/featbit-samples
- **FeatBit Documentation**: https://docs.featbit.co/
- **OpenFeature Documentation**: https://openfeature.dev/docs/reference/intro

---

**Need more details?** Visit the [official GitHub repository](https://github.com/featbit/featbit-java-sdk) for complete documentation and advanced use cases.

