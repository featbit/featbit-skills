---
name: featbit-java-sdk
description: Expert guidance for integrating FeatBit Java Server-Side SDK in backend applications. Use for Spring Boot, Servlet, or any Java server application. Not for Android - use Android SDK for mobile.
appliesTo:
  - "**/*.java"
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/src/main/**/*.java"
  - "**/application.properties"
  - "**/application.yml"
---

# FeatBit Java Server SDK Expert

Expert knowledge for integrating FeatBit Server-Side SDK in Java applications.

**Source**: https://github.com/featbit/featbit-java-sdk

⚠️ **Important**: This is a **server-side SDK** for multi-user systems (web servers, APIs). Not for Android apps - use Android SDK for mobile.

## Data Synchronization

- **WebSocket** for real-time sync with FeatBit server
- Data stored in memory by default
- Changes pushed to SDK in <100ms average
- Auto-reconnects after internet outage

## Installation

### Maven

```xml
<dependencies>
    <dependency>
      <groupId>co.featbit</groupId>
      <artifactId>featbit-java-sdk</artifactId>
      <version>1.4.4</version>
    </dependency>
</dependencies>
```

### Gradle

```gradle
implementation 'co.featbit:featbit-java-sdk:1.4.4'
```

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

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
            // The flag key to be evaluated
            String flagKey = "use-new-algorithm";

            // The user
            FBUser user = new FBUser.Builder("bot-id")
                    .userName("bot")
                    .build();

            // Evaluate a boolean flag for a given user
            Boolean flagValue = client.boolVariation(flagKey, user, false);
            System.out.printf("flag %s, returns %b for user %s%n", flagKey, flagValue, user.getUserName());

            // Evaluate a boolean flag for a given user with evaluation detail
            EvalDetail<Boolean> ed = client.boolVariationDetail(flagKey, user, false);
            System.out.printf("flag %s, returns %b for user %s, reason: %s%n", flagKey, ed.getVariation(), user.getUserName(), ed.getReason());
        }

        // Close the client to ensure that all insights are sent out before the app exits
        client.close();
        System.out.println("APP FINISHED");
    }
}
```

## FBClient

Applications **SHOULD instantiate a single FBClient instance** for the lifetime of the application.

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
FBClient client = new FBClientImp(sdkKey, config);

// later, when you want to wait for initialization to finish:
boolean inited = client.getDataUpdateStatusProvider().waitForOKState(Duration.ofSeconds(10))
if (inited) {
    // the client is ready
}
```

## Spring Boot Integration

```java
// FeatBitConfig.java
import co.featbit.server.FBClient;
import co.featbit.server.FBClientImp;
import co.featbit.server.FBConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PreDestroy;
import java.time.Duration;

@Configuration
public class FeatBitConfig {
    
    @Value("${featbit.env-secret}")
    private String envSecret;
    
    @Value("${featbit.streaming-url}")
    private String streamingUrl;
    
    @Value("${featbit.event-url}")
    private String eventUrl;
    
    private FBClient fbClient;
    
    @Bean
    public FBClient fbClient() {
        FBConfig config = new FBConfig.Builder()
                .streamingURL(streamingUrl)
                .eventURL(eventUrl)
                .startWaitTime(Duration.ofSeconds(15))
                .build();
        
        fbClient = new FBClientImp(envSecret, config);
        
        if (!fbClient.isInitialized()) {
            throw new RuntimeException("FeatBit SDK failed to initialize");
        }
        
        return fbClient;
    }
    
    @PreDestroy
    public void cleanup() {
        if (fbClient != null) {
            try {
                fbClient.close();
            } catch (Exception e) {
                // Log error
            }
        }
    }
}

// application.properties
featbit.env-secret=your-env-secret
featbit.streaming-url=wss://app-eval.featbit.co
featbit.event-url=https://app-eval.featbit.co
```

```java
// FeatureService.java
import co.featbit.server.FBClient;
import co.featbit.commons.model.FBUser;
import org.springframework.stereotype.Service;

@Service
public class FeatureService {
    
    private final FBClient fbClient;
    
    public FeatureService(FBClient fbClient) {
        this.fbClient = fbClient;
    }
    
    public boolean isFeatureEnabled(String userId, String flagKey) {
        FBUser user = new FBUser.Builder(userId).build();
        return fbClient.boolVariation(flagKey, user, false);
    }
    
    public <T> T getFeatureValue(String userId, String flagKey, T defaultValue) {
        FBUser user = new FBUser.Builder(userId).build();
        return fbClient.variation(flagKey, user, defaultValue);
    }
}
```

```java
// FeatureController.java
import co.featbit.commons.model.FBUser;
import co.featbit.server.FBClient;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class FeatureController {
    
    private final FBClient fbClient;
    
    public FeatureController(FBClient fbClient) {
        this.fbClient = fbClient;
    }
    
    @GetMapping("/feature")
    public ResponseEntity<Map<String, Object>> checkFeature(
            @RequestHeader("X-User-Id") String userId) {
        
        FBUser user = new FBUser.Builder(userId)
                .userName("User " + userId)
                .custom("role", "user")
                .build();
        
        boolean isEnabled = fbClient.boolVariation("new-feature", user, false);
        
        if (!isEnabled) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body(Map.of("error", "Feature not available"));
        }
        
        return ResponseEntity.ok(Map.of("data", "feature data"));
    }
    
    @PostMapping("/purchase")
    public ResponseEntity<Map<String, Object>> purchase(
            @RequestHeader("X-User-Id") String userId,
            @RequestBody PurchaseRequest request) {
        
        FBUser user = new FBUser.Builder(userId).build();
        
        String variant = fbClient.variation("pricing-strategy", user, "standard");
        double price = calculatePrice(request, variant);
        
        PurchaseResult result = processPurchase(price);
        
        if (result.isSuccess()) {
            fbClient.trackMetric(user, "purchase-completed", price);
        }
        
        return ResponseEntity.ok(Map.of(
                "success", result.isSuccess(),
                "variant", variant,
                "price", price
        ));
    }
}
```

## FBUser Builder

```java
import co.featbit.commons.model.FBUser;

// Basic user
FBUser user = new FBUser.Builder("user-key-123")
        .userName("John Doe")
        .build();

// User with custom properties
FBUser user = new FBUser.Builder("user-key-456")
        .userName("Bob Wilson")
        .custom("role", "admin")
        .custom("country", "US")
        .custom("subscription", "premium")
        .custom("age", 30)
        .build();
```

## Configuration Options

```java
import java.time.Duration;

FBConfig config = new FBConfig.Builder()
        // Required
        .streamingURL("wss://app-eval.featbit.co")
        .eventURL("https://app-eval.featbit.co")
        
        // Optional
        .startWaitTime(Duration.ofSeconds(15))  // Wait time for initialization
        .offline(false)                         // Offline mode
        .disableEvents(false)                   // Disable event sending
        .build();
```

### HTTP Configuration

```java
import co.featbit.server.Factory;
import co.featbit.server.HttpConfigFactory;
import java.time.Duration;

HttpConfigFactory factory = Factory.httpConfigFactory()
        .connectTime(Duration.ofMillis(3000))
        .readTime(Duration.ofMillis(5000))
        .writeTime(Duration.ofMillis(5000))
        .httpProxy("my-proxy", 9000);

FBConfig config = new FBConfig.Builder()
        .httpConfigFactory(factory)
        .build();
```

## Flag Evaluation

### Boolean Variation

```java
Boolean isEnabled = client.boolVariation("flag-key", user, false);
```

### String Variation

```java
String theme = client.variation("theme", user, "default");
```

### Numeric Variations

```java
// Double
Double percentage = client.doubleVariation("rollout-percentage", user, 0.0);

// Long
Long timeout = client.longVariation("timeout", user, 30000L);

// Integer
Integer maxRetries = client.intVariation("max-retries", user, 3);
```

### JSON Variation

```java
import com.google.gson.JsonObject;

JsonObject config = client.jsonVariation("config", user, new JsonObject());
String apiKey = config.get("apiKey").getAsString();
int maxConnections = config.get("maxConnections").getAsInt();
```

### With Evaluation Detail

```java
import co.featbit.commons.model.EvalDetail;

EvalDetail<String> detail = client.variationDetail("flag-key", user, "default");
System.out.println("Value: " + detail.getVariation());
System.out.println("Reason: " + detail.getReason());
System.out.println("Kind: " + detail.getKind());
```

### Get All Flags

```java
import co.featbit.commons.model.AllFlagStates;

AllFlagStates states = client.getAllLatestFlagsVariations(user);
Collection<String> flagKeys = states.getFlagKeys();

// Get specific flag values
EvalDetail<String> detail = states.getStringDetail("flag-key", "default");
String value = states.getString("flag-key", "default");
```

## Event Tracking

```java
// Simple event
client.trackMetric(user, "button-clicked");

// Event with numeric value
client.trackMetric(user, "purchase-completed", 99.99);

// For A/B testing
client.trackMetric(user, "conversion-event");

// Flush events manually
client.flush();
```

## Flag Tracking (Listeners)

```java
client.getFlagTracker().addFlagValueChangeListener(flagKey, user, event -> {
    System.out.println("Flag " + event.getKey() + " changed");
    System.out.println("Old value: " + event.getOldValue());
    System.out.println("New value: " + event.getNewValue());
});
```

## Servlet Integration

```java
import co.featbit.server.FBClient;
import co.featbit.commons.model.FBUser;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class FeatureServlet extends HttpServlet {
    
    private FBClient fbClient;
    
    @Override
    public void init() {
        FBConfig config = new FBConfig.Builder()
                .streamingURL(getServletContext().getInitParameter("featbit.streaming.url"))
                .eventURL(getServletContext().getInitParameter("featbit.event.url"))
                .build();
        
        fbClient = new FBClientImp(
                getServletContext().getInitParameter("featbit.env.secret"),
                config
        );
    }
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
            throws IOException {
        
        String userId = req.getParameter("userId");
        FBUser user = new FBUser.Builder(userId).build();
        
        boolean isEnabled = fbClient.boolVariation("new-feature", user, false);
        
        resp.setContentType("application/json");
        resp.getWriter().write("{\"enabled\": " + isEnabled + "}");
    }
    
    @Override
    public void destroy() {
        if (fbClient != null) {
            try {
                fbClient.close();
            } catch (Exception e) {
                log("Error closing FeatBit client", e);
            }
        }
    }
}
```

## Offline Mode

```java
FBConfig config = new FBConfig.Builder()
        .streamingURL(streamUrl)
        .eventURL(eventUrl)
        .offline(true)
        .build();

FBClient client = new FBClientImp(envSecret, config);

// Load flags from JSON file
String json = Resources.toString(
        Resources.getResource("featbit-bootstrap.json"), 
        Charsets.UTF_8
);

if(client.initFromJsonFile(json)){
    // the client is ready
}
```

### Generate Bootstrap JSON

```bash
# Get current flags
curl -H "Authorization: <your-env-secret>" \
     http://localhost:5100/api/public/sdk/server/latest-all > featbit-bootstrap.json
```

## Best Practices

### 1. Single Client Instance

```java
// ✅ Good: Use Dependency Injection
@Bean
public FBClient fbClient() {
    return new FBClientImp(envSecret, config);
}

// ❌ Bad: Creating per request
@GetMapping("/feature")
public void checkFeature() {
    FBClient client = new FBClientImp(...); // Don't do this!
}
```

### 2. Wait for Initialization

```java
// ✅ Good: Wait and verify
if (!fbClient.isInitialized()) {
    throw new RuntimeException("SDK not ready");
}

// ❌ Bad: Use without checking
FBClient client = new FBClientImp(...);
client.boolVariation(...); // May return defaults
```

### 3. Graceful Shutdown

```java
@PreDestroy
public void cleanup() {
    if (fbClient != null) {
        fbClient.close();  // Flush events
    }
}
```

### 4. Error Handling

```java
try {
    Boolean isEnabled = fbClient.boolVariation("flag-key", user, false);
    // Use feature
} catch (Exception e) {
    logger.error("Flag evaluation failed", e);
    // Fall back to default
    Boolean isEnabled = false;
}
```

## Common Use Cases

### Progressive Rollout

```java
@GetMapping("/api/new-feature")
public ResponseEntity<Data> newFeature(@AuthenticationPrincipal User user) {
    FBUser fbUser = new FBUser.Builder(user.getId()).build();
    
    boolean useNewVersion = fbClient.boolVariation("new-api-version", fbUser, false);
    
    if (useNewVersion) {
        return ResponseEntity.ok(newVersionHandler());
    } else {
        return ResponseEntity.ok(oldVersionHandler());
    }
}
```

### A/B Testing

```java
@GetMapping("/api/recommendations")
public List<Recommendation> getRecommendations(@AuthenticationPrincipal User user) {
    FBUser fbUser = new FBUser.Builder(user.getId()).build();
    
    String algorithm = fbClient.variation("recommendation-algorithm", fbUser, "default");
    
    List<Recommendation> recommendations = generateRecommendations(algorithm);
    
    // Track which algorithm was used
    fbClient.trackMetric(fbUser, "algorithm-" + algorithm);
    
    return recommendations;
}
```

### Remote Configuration

```java
JsonObject config = fbClient.jsonVariation("api-config", systemUser, defaultConfig);

int timeout = config.get("timeout").getAsInt();
int maxRetries = config.get("maxRetries").getAsInt();
int rateLimit = config.get("rateLimit").getAsInt();

// Use configuration
configureHttpClient(timeout, maxRetries, rateLimit);
```

## Troubleshooting

### SDK Not Initializing

```java
FBClient client = new FBClientImp(envSecret, config);
if (!client.isInitialized()) {
    // Check:
    // 1. envSecret is correct
    // 2. Network connectivity
    // 3. Firewall rules for WebSocket
    logger.error("FeatBit SDK failed to initialize");
}
```

### Connection Issues

```java
// Increase timeout
FBConfig config = new FBConfig.Builder()
        .startWaitTime(Duration.ofSeconds(30))
        .build();
```

### Events Not Sent

```java
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    if (fbClient != null) {
        fbClient.close();  // Flush events
    }
}));
```

## Testing

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.AfterAll;

public class FeatureTest {
    
    private static FBClient fbClient;
    
    @BeforeAll
    public static void setUp() {
        FBConfig config = new FBConfig.Builder()
                .streamingURL("wss://test-eval.featbit.co")
                .eventURL("https://test-eval.featbit.co")
                .build();
        
        fbClient = new FBClientImp("test-env-secret", config);
    }
    
    @Test
    public void testFeatureFlag() {
        FBUser user = new FBUser.Builder("test-user").build();
        Boolean isEnabled = fbClient.boolVariation("test-feature", user, false);
        
        assertNotNull(isEnabled);
    }
    
    @AfterAll
    public static void tearDown() {
        if (fbClient != null) {
            try {
                fbClient.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

## OpenFeature Provider

FeatBit also provides an OpenFeature provider for Java:

```xml
<dependency>
    <groupId>co.featbit</groupId>
    <artifactId>featbit-openfeature-provider-java-server</artifactId>
    <version>1.0.0</version>
</dependency>
```

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

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-java-sdk
- **Maven Central**: https://mvnrepository.com/artifact/co.featbit/featbit-java-sdk
- **Examples**: https://github.com/featbit/featbit-samples
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/java
