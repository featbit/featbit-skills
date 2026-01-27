---
name: featbit-go-sdk
description: Expert guidance for integrating FeatBit Go Server-Side SDK in backend applications. Use for Gin, Echo, or any Go server application. Not for client-side use.
appliesTo:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
  - "**/main.go"
  - "**/server.go"
  - "**/api/**/*.go"
  - "**/handlers/**/*.go"
---

# FeatBit Go Server SDK Expert

Expert knowledge for integrating FeatBit Server-Side SDK in Go applications.

**Source**: https://github.com/featbit/featbit-go-sdk

⚠️ **Important**: This is a **server-side SDK** for multi-user systems (web servers, APIs). Not for client-side use.

## Data Synchronization

- **WebSocket** for real-time sync with FeatBit server
- Data stored in memory by default
- Changes pushed to SDK in <100ms average
- Auto-reconnects after internet outage

## Installation

```bash
go get github.com/featbit/featbit-go-sdk
```

**Requirements**: Go 1.13 or above

## Prerequisites

- **Environment Secret**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

```go
package main

import (
	"fmt"
	"github.com/featbit/featbit-go-sdk"
	"github.com/featbit/featbit-go-sdk/interfaces"
)

func main() {
	envSecret := "<replace-with-your-env-secret>"
	streamingUrl := "ws://localhost:5100"
	eventUrl := "http://localhost:5100"

	client, err := featbit.NewFBClient(envSecret, streamingUrl, eventUrl)

	defer func() {
		if client != nil {
			// ensure that the SDK shuts down cleanly and has a chance to deliver events to FeatBit before the program exits
			_ = client.Close()
		}
	}()

	if err == nil && client.IsInitialized() {
		user, _ := interfaces.NewUserBuilder("<replace-with-your-user-key>").UserName("<replace-with-your-user-name>").Build()
		_, ed, _ := client.BoolVariation("<replace-with-your-feature-flag-key>", user, false)
		fmt.Printf("flag %s, returns %s for user %s, reason: %s \n", ed.KeyName, ed.Variation, user.GetKey(), ed.Reason)
	} else {
		fmt.Println("SDK initialization failed")
    }
}
```

## FBClient

Applications **SHOULD instantiate a single FBClient instance** for the lifetime of the application.

### Bootstrapping

```go
import (
	"github.com/featbit/featbit-go-sdk"
	"time"
)

config := featbit.FBConfig{StartWait: 10 * time.Second}
// DO NOT forget to close the client when you don't need it anymore
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
if err == nil && client.IsInitialized() {
    // the client is ready
}
```

### Asynchronous Initialization

```go
config := featbit.FBConfig{StartWait: 0}
// DO NOT forget to close the client when you don't need it anymore
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
if err != nil {
    return
}
ok := client.GetDataSourceStatusProvider().WaitForOKState(10 * time.Second)
if ok {
    // the client is ready
}
```

## Gin Framework Integration

```go
package main

import (
	"github.com/featbit/featbit-go-sdk"
	"github.com/featbit/featbit-go-sdk/interfaces"
	"github.com/gin-gonic/gin"
	"net/http"
)

var fbClient *featbit.FBClient

func main() {
	// Initialize FeatBit client
	var err error
	fbClient, err = featbit.NewFBClient(
		"your-env-secret",
		"wss://app-eval.featbit.co",
		"https://app-eval.featbit.co",
	)
	
	if err != nil || !fbClient.IsInitialized() {
		panic("FeatBit SDK failed to initialize")
	}
	
	defer fbClient.Close()
	
	// Setup Gin router
	router := gin.Default()
	
	// Middleware to create user context
	router.Use(func(c *gin.Context) {
		userId := c.GetHeader("X-User-Id")
		if userId == "" {
			userId = "anonymous"
		}
		
		user, _ := interfaces.NewUserBuilder(userId).
			UserName(c.GetHeader("X-User-Name")).
			Custom("role", c.GetHeader("X-User-Role")).
			Build()
		
		c.Set("fb_user", user)
		c.Next()
	})
	
	// Route with feature flag
	router.GET("/api/feature", func(c *gin.Context) {
		user := c.MustGet("fb_user").(interfaces.IUser)
		
		isEnabled, _, _ := fbClient.BoolVariation("new-feature", user, false)
		
		if !isEnabled {
			c.JSON(http.StatusForbidden, gin.H{"error": "Feature not available"})
			return
		}
		
		c.JSON(http.StatusOK, gin.H{"data": "feature data"})
	})
	
	// Gradual rollout example
	router.POST("/api/checkout", func(c *gin.Context) {
		user := c.MustGet("fb_user").(interfaces.IUser)
		
		useNewCheckout, _, _ := fbClient.BoolVariation("new-checkout-flow", user, false)
		
		var result map[string]interface{}
		if useNewCheckout {
			result = processNewCheckout(c)
		} else {
			result = processOldCheckout(c)
		}
		
		c.JSON(http.StatusOK, result)
	})
	
	// A/B testing example
	router.POST("/api/purchase", func(c *gin.Context) {
		user := c.MustGet("fb_user").(interfaces.IUser)
		
		variant, _, _ := fbClient.Variation("pricing-strategy", user, "standard")
		
		price := calculatePrice(c, variant)
		result := processPurchase(price)
		
		if result["success"].(bool) {
			// Track conversion
			fbClient.TrackNumericMetric(user, "purchase-completed", price)
		}
		
		c.JSON(http.StatusOK, result)
	})
	
	router.Run(":8080")
}
```

## Echo Framework Integration

```go
package main

import (
	"github.com/featbit/featbit-go-sdk"
	"github.com/featbit/featbit-go-sdk/interfaces"
	"github.com/labstack/echo/v4"
	"github.com/labstack/echo/v4/middleware"
	"net/http"
)

var fbClient *featbit.FBClient

func main() {
	// Initialize FeatBit
	var err error
	fbClient, err = featbit.NewFBClient(
		"your-env-secret",
		"wss://app-eval.featbit.co",
		"https://app-eval.featbit.co",
	)
	
	if err != nil || !fbClient.IsInitialized() {
		panic("FeatBit SDK failed to initialize")
	}
	
	defer fbClient.Close()
	
	// Setup Echo
	e := echo.New()
	e.Use(middleware.Logger())
	e.Use(middleware.Recover())
	
	// Middleware to create user context
	e.Use(func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c echo.Context) error {
			userId := c.Request().Header.Get("X-User-Id")
			if userId == "" {
				userId = "anonymous"
			}
			
			user, _ := interfaces.NewUserBuilder(userId).
				UserName(c.Request().Header.Get("X-User-Name")).
				Build()
			
			c.Set("fb_user", user)
			return next(c)
		}
	})
	
	// Routes
	e.GET("/api/feature", func(c echo.Context) error {
		user := c.Get("fb_user").(interfaces.IUser)
		
		isEnabled, _, _ := fbClient.BoolVariation("new-feature", user, false)
		
		if !isEnabled {
			return c.JSON(http.StatusForbidden, map[string]string{
				"error": "Feature not available",
			})
		}
		
		return c.JSON(http.StatusOK, map[string]string{
			"data": "feature data",
		})
	})
	
	e.Logger.Fatal(e.Start(":8080"))
}
```

## FBUser Builder

```go
import "github.com/featbit/featbit-go-sdk/interfaces"

// Basic user
user, err := interfaces.NewUserBuilder("user-key-123").
    UserName("John Doe").
    Build()

// User with custom properties
user, err := interfaces.NewUserBuilder("user-key-456").
    UserName("Bob Wilson").
    Custom("role", "admin").
    Custom("country", "US").
    Custom("subscription", "premium").
    Custom("age", "30").
    Build()
```

## Configuration Options

```go
import (
	"github.com/featbit/featbit-go-sdk"
	"time"
)

// Default configuration
client, err := featbit.NewFBClient(envSecret, streamingUrl, eventUrl)

// Custom configuration
config := featbit.FBConfig{
    StartWait: 15 * time.Second,  // Wait time for initialization
    Offline:   false,              // Offline mode
}

client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
```

### Network Configuration

```go
import "github.com/featbit/featbit-go-sdk/factories"

factory := factories.NewNetworkBuilder()
factory.ProxyUrl("http://username:password@146.137.9.45:65233")

config := featbit.DefaultFBConfig
config.NetworkFactory = factory
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, *config)
```

## Flag Evaluation

### Boolean Variation

```go
isEnabled, detail, err := client.BoolVariation("flag-key", user, false)
if err != nil {
    // Handle error
}
fmt.Printf("Value: %v, Reason: %s\n", isEnabled, detail.Reason)
```

### String Variation

```go
theme, detail, err := client.Variation("theme", user, "default")
```

### Numeric Variations

```go
// Integer
maxRetries, detail, err := client.IntVariation("max-retries", user, 3)

// Double
percentage, detail, err := client.DoubleVariation("rollout-percentage", user, 0.0)
```

### JSON Variation

```go
import "encoding/json"

config, detail, err := client.JsonVariation("config", user, "{}")

var configObj map[string]interface{}
json.Unmarshal([]byte(config), &configObj)

timeout := configObj["timeout"].(float64)
maxRetries := configObj["maxRetries"].(float64)
```

### Get All Flags

```go
allState, err := client.AllLatestFlagsVariations(user)
if err != nil {
    // Handle error
}

variation, detail, err := allState.GetStringVariation("flag-key", "default")
```

## Event Tracking

```go
// Percentage event (for A/B testing)
client.TrackPercentageMetric(user, "button-clicked")

// Numeric event (for measuring values)
client.TrackNumericMetric(user, "purchase-completed", 99.99)

// Flush events manually
client.Flush()
```

## Standard HTTP Server Integration

```go
package main

import (
	"encoding/json"
	"github.com/featbit/featbit-go-sdk"
	"github.com/featbit/featbit-go-sdk/interfaces"
	"net/http"
)

var fbClient *featbit.FBClient

func main() {
	// Initialize FeatBit
	var err error
	fbClient, err = featbit.NewFBClient(
		"your-env-secret",
		"wss://app-eval.featbit.co",
		"https://app-eval.featbit.co",
	)
	
	if err != nil || !fbClient.IsInitialized() {
		panic("FeatBit SDK failed to initialize")
	}
	
	defer fbClient.Close()
	
	// Routes
	http.HandleFunc("/api/feature", featureHandler)
	http.HandleFunc("/api/recommendations", recommendationsHandler)
	
	http.ListenAndServe(":8080", nil)
}

func featureHandler(w http.ResponseWriter, r *http.Request) {
	userId := r.Header.Get("X-User-Id")
	if userId == "" {
		userId = "anonymous"
	}
	
	user, _ := interfaces.NewUserBuilder(userId).Build()
	
	isEnabled, _, _ := fbClient.BoolVariation("new-feature", user, false)
	
	w.Header().Set("Content-Type", "application/json")
	if !isEnabled {
		w.WriteHeader(http.StatusForbidden)
		json.NewEncoder(w).Encode(map[string]string{
			"error": "Feature not available",
		})
		return
	}
	
	json.NewEncoder(w).Encode(map[string]string{
		"data": "feature data",
	})
}

func recommendationsHandler(w http.ResponseWriter, r *http.Request) {
	userId := r.Header.Get("X-User-Id")
	user, _ := interfaces.NewUserBuilder(userId).Build()
	
	algorithm, _, _ := fbClient.Variation("recommendation-algorithm", user, "default")
	
	recommendations := generateRecommendations(algorithm)
	
	// Track which algorithm was used
	fbClient.TrackPercentageMetric(user, "algorithm-"+algorithm)
	
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(map[string]interface{}{
		"recommendations": recommendations,
		"algorithm":       algorithm,
	})
}
```

## Offline Mode

```go
config := featbit.DefaultFBConfig
featbit.Offline = true
featbit.StartWait = 1 * time.Millisecond
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, *config)
// or
config := FBConfig{Offline: true, StartWait: 1 * time.Millisecond}
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)

// Load flags from JSON
jsonBytes, _ := os.ReadFile("featbit-bootstrap.json")
ok, _ := client.InitializeFromExternalJson(string(jsonBytes))
if ok {
    // Client is ready with local data
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

```go
// ✅ Good: Create once at app startup
var fbClient *featbit.FBClient

func init() {
    fbClient, _ = featbit.NewFBClient(...)
}

// ❌ Bad: Creating per request
func handler(w http.ResponseWriter, r *http.Request) {
    client, _ := featbit.NewFBClient(...) // Don't do this!
}
```

### 2. Wait for Initialization

```go
// ✅ Good: Wait before starting server
client, err := featbit.NewFBClient(...)
if err != nil || !client.IsInitialized() {
    log.Fatal("SDK initialization failed")
}

// Start server after initialization
http.ListenAndServe(":8080", nil)

// ❌ Bad: Start immediately
client, _ := featbit.NewFBClient(...)
http.ListenAndServe(":8080", nil) // Flags may return defaults
```

### 3. Graceful Shutdown

```go
func main() {
    client, _ := featbit.NewFBClient(...)
    
    defer func() {
        if client != nil {
            client.Close()  // Flush events
        }
    }()
    
    // Application code
}
```

### 4. Error Handling

```go
isEnabled, detail, err := fbClient.BoolVariation("flag-key", user, false)
if err != nil {
    log.Printf("Flag evaluation failed: %v", err)
    // Fall back to default
    isEnabled = false
}
```

### 5. User Context

```go
// ✅ Good: Use stable identifiers
user, _ := interfaces.NewUserBuilder(currentUser.ID).
    UserName(currentUser.Name).
    Custom("role", currentUser.Role).
    Custom("plan", currentUser.Subscription).
    Build()

// ❌ Bad: Random IDs
import "math/rand"
user, _ := interfaces.NewUserBuilder(fmt.Sprintf("%d", rand.Int())).Build()
```

## Common Use Cases

### Progressive Rollout

```go
func newFeatureHandler(w http.ResponseWriter, r *http.Request) {
    user := getUserFromRequest(r)
    
    useNewVersion, _, _ := fbClient.BoolVariation("new-api-version", user, false)
    
    if useNewVersion {
        json.NewEncoder(w).Encode(newVersionHandler(r))
    } else {
        json.NewEncoder(w).Encode(oldVersionHandler(r))
    }
}
```

### A/B Testing

```go
func recommendationsHandler(w http.ResponseWriter, r *http.Request) {
    user := getUserFromRequest(r)
    
    algorithm, _, _ := fbClient.Variation("recommendation-algorithm", user, "default")
    
    recommendations := getRecommendations(user, algorithm)
    
    // Track which algorithm was used
    fbClient.TrackPercentageMetric(user, fmt.Sprintf("algorithm-%s", algorithm))
    
    json.NewEncoder(w).Encode(recommendations)
}
```

### Remote Configuration

```go
config, _, _ := fbClient.JsonVariation("api-config", systemUser, `{
    "timeout": 30,
    "retries": 3,
    "rateLimit": 1000
}`)

var configObj map[string]interface{}
json.Unmarshal([]byte(config), &configObj)

timeout := int(configObj["timeout"].(float64))
retries := int(configObj["retries"].(float64))
rateLimit := int(configObj["rateLimit"].(float64))

// Use configuration
configureHttpClient(timeout, retries, rateLimit)
```

### Feature Toggle

```go
func maintenanceMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        systemUser, _ := interfaces.NewUserBuilder("system").Build()
        maintenanceMode, _, _ := fbClient.BoolVariation("maintenance-mode", systemUser, false)
        
        if maintenanceMode && r.URL.Path != "/health" {
            w.WriteHeader(http.StatusServiceUnavailable)
            json.NewEncoder(w).Encode(map[string]string{
                "error": "Service temporarily unavailable",
            })
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

## Troubleshooting

### SDK Not Initializing

```go
client, err := featbit.NewFBClient(envSecret, streamingUrl, eventUrl)
if err != nil {
    log.Printf("SDK initialization error: %v", err)
    // Check:
    // 1. envSecret is correct
    // 2. Network connectivity
    // 3. Firewall rules for WebSocket
}

if !client.IsInitialized() {
    log.Println("SDK is not initialized")
}
```

### Connection Issues

```go
// Increase timeout
config := featbit.FBConfig{
    StartWait: 30 * time.Second,
}
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
```

### Events Not Sent

```go
import "os/signal"

func main() {
    client, _ := featbit.NewFBClient(...)
    
    // Handle shutdown signals
    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt, syscall.SIGTERM)
    
    go func() {
        <-c
        client.Close()  // Flush events
        os.Exit(0)
    }()
    
    // Application code
}
```

## Testing

```go
package main

import (
	"github.com/featbit/featbit-go-sdk"
	"github.com/featbit/featbit-go-sdk/interfaces"
	"testing"
)

var testClient *featbit.FBClient

func TestMain(m *testing.M) {
	// Setup
	var err error
	testClient, err = featbit.NewFBClient(
		"test-env-secret",
		"wss://test-eval.featbit.co",
		"https://test-eval.featbit.co",
	)
	
	if err != nil || !testClient.IsInitialized() {
		panic("Test SDK initialization failed")
	}
	
	// Run tests
	code := m.Run()
	
	// Cleanup
	testClient.Close()
	
	os.Exit(code)
}

func TestFeatureFlag(t *testing.T) {
	user, _ := interfaces.NewUserBuilder("test-user").Build()
	
	isEnabled, detail, err := testClient.BoolVariation("test-feature", user, false)
	
	if err != nil {
		t.Fatalf("Flag evaluation failed: %v", err)
	}
	
	t.Logf("Flag value: %v, Reason: %s", isEnabled, detail.Reason)
}
```

## Additional Resources

- **GitHub**: https://github.com/featbit/featbit-go-sdk
- **Examples**: https://github.com/featbit/featbit-samples
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/go
