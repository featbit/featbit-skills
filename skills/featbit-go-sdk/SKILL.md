---
name: featbit-go-sdk
description: Expert knowledge for FeatBit Go Server-Side SDK integration in web servers and backend applications. Use when working with .go files, building feature flags in Go servers, implementing A/B testing in Go, or when user mentions "Go SDK", "featbit go", "feature flags golang", or "go websocket sync". Server-side only, not for client applications.
appliesTo:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
  - "**/main.go"
  - "**/server.go"
  - "**/api/**/*.go"
  - "**/handlers/**/*.go"
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  sdk-type: server-side
  repository: https://github.com/featbit/featbit-go-sdk
---

# FeatBit Go Server-Side SDK

Server-side SDK for feature flags management in Go applications (web servers, APIs, multi-user systems).

**Repository**: https://github.com/featbit/featbit-go-sdk  
**Go Version**: 1.13+

⚠️ **Server-Side Only**: Designed for multi-user systems. Not for client-side use.

> **Note**: This skill contains core SDK knowledge. For issues not covered here or troubleshooting problems, refer users to the [GitHub repository](https://github.com/featbit/featbit-go-sdk) for complete source code, examples, and issue tracking.

## Data Synchronization

- **WebSocket** for real-time sync with FeatBit server
- Data stored in memory by default
- Changes pushed to SDK in <100ms average
- Auto-reconnects after internet outage

## Installation

```bash
go get github.com/featbit/featbit-go-sdk
```

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
			// Ensure SDK shuts down cleanly and delivers events before exit
			_ = client.Close()
		}
	}()

	if err == nil && client.IsInitialized() {
		user, _ := interfaces.NewUserBuilder("<replace-with-your-user-key>").
			UserName("<replace-with-your-user-name>").
			Build()
		
		_, ed, _ := client.BoolVariation("<replace-with-your-feature-flag-key>", user, false)
		fmt.Printf("flag %s, returns %s for user %s, reason: %s \n", 
			ed.KeyName, ed.Variation, user.GetKey(), ed.Reason)
	} else {
		fmt.Println("SDK initialization failed")
	}
}
```

**Examples**: [Go Demo](https://github.com/featbit/featbit-samples/blob/main/samples/dino-game/demo-golang/go_demo.go)

## FBClient

Applications **SHOULD instantiate a single FBClient instance** for the lifetime of the application. For multiple environments, create multiple clients but retain them for the application lifetime.

### Bootstrapping

Constructor returns when it successfully connects, or when `FBConfig.StartWait` timeout (default: 15 seconds) expires. If timeout elapses, you receive client in uninitialized state where feature flags return default values. It will still continue connecting in background unless there's a `net.DNSError` or you close the client.

```go
import (
	"github.com/featbit/featbit-go-sdk"
	"time"
)

config := featbit.FBConfig{StartWait: 10 * time.Second}
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
if err == nil && client.IsInitialized() {
	// the client is ready
}
```

**Check Initialization**: `client.IsInitialized()` returns True if client has succeeded at some point in connecting to feature flag center.

### Asynchronous Initialization

```go
config := featbit.FBConfig{StartWait: 0}
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, config)
if err != nil {
	return
}
ok := client.GetDataUpdateStatusProvider().WaitForOKState(10 * time.Second)
if ok {
	// the client is ready
}
```

> Checking if client is ready is optional. Even if not ready, you can still evaluate feature flags, but default value will be returned.

## FBConfig and Components

In most cases, just initialize SDK like:

```go
client, err := featbit.NewFBClient(envSecret, streamingUrl, eventUrl)
```

**Parameters**:
- `envSecret`: Environment secret from FeatBit
- `streamingURL`: URL to synchronize feature flags, user segments, etc.
- `eventURL`: URL to send analytics events
- `StartWait`: How long constructor will block awaiting successful data sync. Zero or negative = return immediately
- `Offline`: Set SDK offline mode (no connection to platform)

### Network Configuration

```go
import "github.com/featbit/featbit-go-sdk/factories"

factory := factories.NewNetworkBuilder()
factory.ProxyUrl("http://username:password@146.137.9.45:65233")

config := featbit.DefaultFBConfig
config.NetworkFactory = factory
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, *config)
```

**Advanced Components** (not recommended to change):
- `NetworkFactory`: SDK networking configuration (default: `factories.NetworkBuilder`)
- `DataStorageFactory`: Data storage implementation (default: in-memory)
- `DataSynchronizerFactory`: Data synchronization implementation (default: `factories.StreamingBuilder`)
- `InsightProcessorFactory`: Analytics events processing (default: `factories.InsightProcessorBuilder`)

## FBUser

User object with built-in properties (`key`, `userName`) and custom properties.

**Required**:
- `key`: Unique identifier (username, email for authenticated users, or ID for anonymous users)
- `userName`: For quick user search

```go
import "github.com/featbit/featbit-go-sdk/interfaces"

// Basic user
user, err := interfaces.NewUserBuilder("key").
	UserName("name").
	Custom("property", "value").
	Build()
```

## Evaluation

SDK calculates feature flag value for a given user and returns flag value and `interfaces.EvalDetail` describing how the value was determined.

After initialization, SDK has all feature flags in memory and **all evaluation is done locally and synchronously** (average < 10 ms).

### Variation Types

```go
// String
variation, detail, _ := client.Variation("flag key", user, "Not Found")

// Boolean
boolValue, detail, _ := client.BoolVariation("flag key", user, false)

// Integer
intValue, detail, _ := client.IntVariation("flag key", user, 0)

// Double
doubleValue, detail, _ := client.DoubleVariation("flag key", user, 0.0)

// JSON
jsonValue, detail, _ := client.JsonVariation("flag key", user, "{}")
```

### Get All Flags

```go
if client.IsInitialized() {
	allState, _ := client.AllLatestFlagsVariations(user)
	variation, detail, _ := allState.GetStringVariation("flag key", "Not Found")
}
```

> If evaluation called before SDK initialized, wrong flag key/user provided, or flag not found, SDK returns the default value. `interfaces.EvalDetail` explains the details including error reason.

## Offline Mode

Stop making remote calls to FeatBit:

```go
config := featbit.DefaultFBConfig
config.Offline = true
config.StartWait = 1 * time.Millisecond
client, err := featbit.MakeCustomFBClient(envSecret, streamingUrl, eventUrl, *config)
```

In offline mode, no insight messages sent to server and all feature flag evaluations return fallback values (no flags/segments available).

### Bootstrap from JSON

SDK allows populating flags and segments data from JSON string. Example: [fbclient_test_data.json](https://github.com/featbit/featbit-go-sdk/blob/main/fixtures/fbclient_test_data.json)

**Get existing flags from FeatBit server**:

```bash
# Replace with your evaluation server URL
curl -H "Authorization: <your-env-secret>" \
     http://localhost:5100/api/public/sdk/server/latest-all > featbit-bootstrap.json
```

**Load in offline mode**:

```go
jsonBytes, _ := os.ReadFile("featbit-bootstrap.json")
ok, _ := client.InitializeFromExternalJson(string(jsonBytes))
```

## Experiments (A/B/n Testing)

Automatic experiments for page-views and clicks (set on FeatBit platform). For custom events:

```go
// Percentage experiment
client.TrackPercentageMetric(user, eventName)

// Numeric experiment
client.TrackNumericMetric(user, eventName, numericValue)
```

⚠️ Call `TrackPercentageMetric()` or `TrackNumericMetric()` **AFTER** the related feature flag is called, otherwise custom event may not be included in experiment result.

## Support & Resources

- **Slack**: [Join FeatBit Slack](https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ)
- **Issues**: [Submit Issue](https://github.com/featbit/featbit/issues/new)
- **Documentation**: [Connect To Go SDK](https://docs.featbit.co/sdk/overview#go)

> **For complex issues or implementation details not covered here**: Direct users to the [official GitHub repository](https://github.com/featbit/featbit-go-sdk) where they can review the complete source code, examples, and tests.
