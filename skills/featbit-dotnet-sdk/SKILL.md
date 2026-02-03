---
name: featbit-dotnet-sdk
description: Expert guidance for integrating FeatBit .NET Server SDK in .NET applications. Use when user asks about ".NET SDK", "C# feature flags", "ASP.NET Core FeatBit", "dependency injection", "console app integration", "OpenFeature .NET", or mentions .cs, .csproj, Program.cs files.
license: MIT
metadata:
  author: FeatBit
  version: 2.0.0
  category: sdk-integration
---

# FeatBit .NET Server SDK Integration

Expert guidance for integrating the FeatBit .NET Server-Side SDK into .NET applications, including ASP.NET Core, console applications, and worker services.

## When to Use This Skill

Activate when users:
- Ask about .NET SDK integration or setup
- Need dependency injection configuration for ASP.NET Core
- Want to evaluate feature flags in C# code
- Ask about console applications, worker services, or background services
- Need examples of flag variations (bool, string, int, double, JSON)
- Want to implement A/B testing with custom events
- Mention offline mode or bootstrapping from JSON
- Ask about OpenFeature integration or vendor-neutral feature flagging

## Prerequisites

Before integration, obtain:
- **Environment Secret**: [How to get it](https://docs.featbit.co/sdk/faq#how-to-get-the-environment-secret)
- **SDK URLs**: [How to get them](https://docs.featbit.co/sdk/faq#how-to-get-the-sdk-urls)

## Quick Start

### Installation

```bash
dotnet add package FeatBit.ServerSdk
```

### Basic Console App Example

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Model;
using FeatBit.Sdk.Server.Options;

// Setup SDK options
var options = new FbOptionsBuilder("<replace-with-your-env-secret>")
    .Event(new Uri("https://app-eval.featbit.co"))
    .Streaming(new Uri("wss://app-eval.featbit.co"))
    .Build();

// Create client instance
var client = new FbClient(options);
if (!client.Initialized)
{
    Console.WriteLine("FbClient failed to initialize. Using fallback values.");
}

// Create user
var user = FbUser.Builder("user-key-123")
    .Name("User Name")
    .Custom("role", "admin")
    .Build();

// Evaluate feature flag
var isEnabled = client.BoolVariation("game-runner", user, defaultValue: false);
Console.WriteLine($"Feature enabled: {isEnabled}");

// Close client before exit
await client.CloseAsync();
```

## ASP.NET Core Integration

### Setup with Dependency Injection

```csharp
using FeatBit.Sdk.Server.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

// Register FeatBit service (uses singleton pattern)
builder.Services.AddFeatBit(options =>
{
    options.EnvSecret = "<replace-with-your-env-secret>";
    options.StreamingUri = new Uri("wss://app-eval.featbit.co");
    options.EventUri = new Uri("https://app-eval.featbit.co");
    options.StartWaitTime = TimeSpan.FromSeconds(3);
    options.DisableEvents = true; // Optional: disable event tracking
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

### Using in Controllers

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Model;
using Microsoft.AspNetCore.Mvc;

public class HomeController : ControllerBase
{
    private readonly IFbClient _fbClient;

    public HomeController(IFbClient fbClient)
    {
        _fbClient = fbClient;
    }

    [HttpGet("check-feature")]
    public IActionResult CheckFeature()
    {
        // Authenticated user
        var user = FbUser.Builder(User.Identity?.Name ?? "anonymous")
            .Name(User.Identity?.Name)
            .Custom("role", "admin")
            .Custom("country", "US")
            .Build();

        var isEnabled = _fbClient.BoolVariation("new-feature", user, defaultValue: false);
        return Ok(new { featureEnabled = isEnabled });
    }

    [HttpGet("public-feature")]
    public IActionResult PublicFeature()
    {
        // Anonymous user
        var sessionId = HttpContext.Session?.Id ?? Guid.NewGuid().ToString();
        var anonymousUser = FbUser.Builder($"anonymous-{sessionId}").Build();

        var hasAccess = _fbClient.BoolVariation("beta-access", anonymousUser, false);
        return Ok(new { betaAccess = hasAccess });
    }
}
```

## FbClient Overview

### Client Lifecycle

**ASP.NET Core**: Registered as singleton via DI, managed automatically  
**Console Apps**: Create one instance, reuse throughout lifetime, call `CloseAsync()` before exit  
**Worker Services**: Inject via DI, same lifecycle as ASP.NET Core

### Custom Configuration Options

```csharp
using FeatBit.Sdk.Server.Options;
using Microsoft.Extensions.Logging;

var loggerFactory = LoggerFactory.Create(x => x.AddConsole());

var options = new FbOptionsBuilder("<your-env-secret>")
    .Streaming(new Uri("wss://app-eval.featbit.co"))
    .Event(new Uri("https://app-eval.featbit.co"))
    .StartWaitTime(TimeSpan.FromSeconds(3))
    .DisableEvents(true)
    .LoggerFactory(loggerFactory)
    .Build();

var client = new FbClient(options);
```

### Logging Support

SDK supports standard .NET logging via `Microsoft.Extensions.Logging`:

```csharp
// Create logger factory with desired providers
var loggerFactory = LoggerFactory.Create(builder => builder.AddConsole());

// Pass to SDK
var options = new FbOptionsBuilder(secret)
    .LoggerFactory(loggerFactory)
    .Build();
```

In ASP.NET Core, `AddFeatBit()` automatically uses the host's logger factory.

## OpenFeature Integration

FeatBit supports [OpenFeature](https://openfeature.dev/), the vendor-neutral feature flagging standard.

### Quick Start with OpenFeature

```bash
dotnet add package FeatBit.OpenFeature.ServerProvider
```

```csharp
using FeatBit.OpenFeature.ServerProvider;
using OpenFeature;
using OpenFeature.Model;

// Setup FeatBit client (same as before)
builder.Services.AddFeatBit(options => { /* ... */ });

// Add OpenFeature
builder.Services.AddOpenFeature((sp, builder) =>
{
    var fbClient = sp.GetRequiredService<IFbClient>();
    builder.AddProvider(new FeatBitProvider(fbClient));
});
```

**Using in code**:
```csharp
public class HomeController : ControllerBase
{
    private readonly IFeatureClient _featureClient;

    public HomeController(IFeatureClient featureClient)
    {
        _featureClient = featureClient;
    }

    public async Task<IActionResult> Index()
    {
        var context = EvaluationContext.Builder()
            .SetTargetingKey(User.Identity?.Name ?? "anonymous")
            .Set("role", "admin")
            .Build();

        var isEnabled = await _featureClient.GetBooleanValueAsync(
            "new-feature", 
            false, 
            context
        );

        return View(new { featureEnabled = isEnabled });
    }
}
```

**📄 Complete Guide**: [OpenFeature Integration](references/openfeature-integration.md)  
Includes: ASP.NET Core setup, all variation types, event tracking, migration guide, and best practices.

## FbUser: User Context

### Building Users

FbUser defines user attributes for flag evaluation. The `key` is mandatory and must uniquely identify each user.

```csharp
// Minimal user
var user = FbUser.Builder("unique-user-key").Build();

// User with built-in and custom attributes
var user = FbUser.Builder("user-123")
    .Name("Bob Smith")
    .Custom("age", "15")
    .Custom("country", "FR")
    .Custom("subscription", "premium")
    .Build();
```

**Built-in attributes**: `key` (required), `name`  
**Custom attributes**: Any key-value pairs for targeting rules and analytics

## Evaluating Flags

The SDK evaluates flags **locally** using cached data synchronized via WebSocket. No network call per evaluation.

### Available Variation Methods

```csharp
// Boolean
var flag = _fbClient.BoolVariation("feature-key", user, defaultValue: false);

// String
var theme = _fbClient.StringVariation("theme-key", user, defaultValue: "default");

// Integer
var maxItems = _fbClient.IntVariation("max-items", user, defaultValue: 10);

// Double
var discount = _fbClient.DoubleVariation("discount-rate", user, defaultValue: 0.0);

// Float (similar to Double)
var ratio = _fbClient.FloatVariation("ratio-key", user, defaultValue: 1.0f);

// JSON (use StringVariation for JSON strings)
var configJson = _fbClient.StringVariation("config-key", user, defaultValue: "{}");
var config = JsonSerializer.Deserialize<MyConfig>(configJson);
```

### Variation with Evaluation Detail

Get flag value plus evaluation metadata:

```csharp
var detail = _fbClient.BoolVariationDetail("feature-key", user, defaultValue: false);
Console.WriteLine($"Value: {detail.Value}");
Console.WriteLine($"Reason: {detail.Kind} - {detail.Reason}");
```

### Default Values

Always provide default values. They're used when:
- SDK is not initialized
- Flag doesn't exist
- Network issues occur
- Evaluation fails

## Experiments (A/B Testing)

Track custom events for experiments and analytics:

```csharp
// Track event without value
_fbClient.Track(user, "purchase-completed");

// Track event with numeric value (default is 1.0)
_fbClient.Track(user, "revenue", 99.99);
```

**Important**: Call `Track()` AFTER evaluating the related feature flag.

## Advanced Scenarios
Worker Service Integration

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.DependencyInjection;
using Microsoft.Extensions.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
                // Configure FeatBit from appsettings.json or inline
                services.AddFeatBit(options =>
                {
                    options.EnvSecret = hostContext.Configuration["FeatBit:EnvSecret"] 
                        ?? "<your-env-secret>";
                    options.StreamingUri = new Uri(
                        hostContext.Configuration["FeatBit:StreamingUri"] 
                        ?? "wss://app-eval.featbit.co");
                    options.EventUri = new Uri(
                        hostContext.Configuration["FeatBit:EventUri"] 
                        ?? "https://app-eval.featbit.co");
                    options.StartWaitTime = TimeSpan.FromSeconds(3);
                });
                
                services.AddHostedService<Worker>();
            });
}

public class Worker : BackgroundService
{
    private readonly IFbClient _fbClient;
    private readonly ILogger<Worker> _logger;

    public Worker(IFbClient fbClient, ILogger<Worker> logger)
    {
        _fbClient = fbClient;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var user = FbUser.Builder("worker-instance").Build();
            var shouldProcess = _fbClient.BoolVariation("enable-processing", user, true);
            
            if (shouldProcess)
            {
                _logger.LogInformation("Processing task at: {time}", DateTimeOffset.Now);
                // Do work here
            }
            else
            {
                _logger.LogInformation("Processing disabled by feature flag");
            }
            
            await Task.Delay(60000, stoppingToken); // Wait 1 minute
        }
    }
}
```

**appsettings.json configuration**:
```json
{
  "FeatBit": {
    "EnvSecret": "your-env-secret",
    "StreamingUri": "wss://app-eval.featbit.co",
    "EventUri": "https://app-eval.featbit.co"
  }
}
```

### 
### Offline Mode

Stop remote calls and optionally bootstrap from JSON:

```csharp
var options = new FbOptionsBuilder()
    .Offline(true)
    .Build();

var client = new FbClient(options);
```

**Bootstrapping from JSON** (only in offline mode):

```bash
# Get current flags from server
curl -H "Authorization: <env-secret>" \
  http://localhost:5100/api/public/sdk/server/latest-all > featbit-bootstrap.json
```

```csharp
var json = File.ReadAllText("featbit-bootstrap.json");

var options = new FbOptionsBuilder()
    .Offline(true)
    .UseJsonBootstrapProvider(json)
    .Build();

var client = new FbClient(options);
```

### Disable Event Collection

Disable automatic events while staying online:

```csharp
var options = new FbOptionsBuilder()
    .DisableEvents(true)
    .Build();
```

## Common Use Cases

### Gradual Rollout
```csharp
var user = FbUser.Builder(userId).Build();
var useNewPayment = _fbClient.BoolVariation("new-payment-processor", user, false);

return useNewPayment 
    ? await _newPaymentService.ProcessAsync(payment)
    : await _legacyPaymentService.ProcessAsync(payment);
```

### Maintenance Mode
```csharp
var systemUser = FbUser.Builder("system").Build();
var isMaintenance = _fbClient.BoolVariation("maintenance-mode", systemUser, false);

if (isMaintenance)
    return StatusCode(503, "Service under maintenance");
```

### Remote Configuration
```csharp
// Simple numeric configuration
var user = FbUser.Builder(userId).Build();
var maxRetries = _fbClient.IntVariation("max-retry-attempts", user, 3);
var timeout = _fbClient.IntVariation("api-timeout-seconds", user, 30);

// Complex JSON configuration
var configJson = _fbClient.StringVariation("app-config", user, "{}");
var appConfig = JsonSerializer.Deserialize<AppConfig>(configJson);

// Use the configuration
var httpClient = new HttpClient
{
    Timeout = TimeSpan.FromSeconds(appConfig.TimeoutSeconds)
};
```

**Example JSON configuration**:
```json
{
  "timeoutSeconds": 30,
  "maxRetries": 3,
  "enableCaching": true,
  "cacheExpiryMinutes": 60,
  "apiEndpoint": "https://api.example.com",
  "features": {
    "enableLogging": true,
    "logLevel": "Information"
  }
}
```

**AppConfig model**:
```csharp
public class AppConfig
{
    public int TimeoutSeconds { get; set; } = 30;
    OpenFeature Provider**: https://www.nuget.org/packages/FeatBit.OpenFeature.ServerProvider
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/dotnet
- **Getting Started**: https://docs.featbit.co/getting-started/connect-an-sdk#net
- **Examples**: https://github.com/featbit/featbit-samples

## Reference Documentation

For detailed information on specific topics, see:
- **[OpenFeature Integration](references/openfeature-integration.md)** - Complete OpenFeature setup and usage guide
    public string ApiEndpoint { get; set; } = "https://api.example.com";
    public FeatureSettings Features { get; set; } = new();
}

public class FeatureSettings
{
    public bool EnableLogging { get; set; } = true;
    public string LogLevel { get; set; } = "Information";
}
```

### A/B Testing
```csharp
var user = FbUser.Builder(userId).Build();
var checkoutFlow = _fbClient.StringVariation("checkout-flow", user, "original");

if (purchaseSuccessful)
{
    _fbClient.Track(user, "purchase-completed");
    _fbClient.Track(user, "revenue", purchaseAmount);
}
```

## Troubleshooting

**Client Not Initializing**:
- Verify `EnvSecret` is correct
- Check network connectivity to FeatBit server
- Ensure WebSocket connections are allowed (firewall)
- Increase `StartWaitTime` if initialization is slow

**Flags Not Updating**:
- Confirm WebSocket connection is active (check logs)
- Ensure SDK is not in offline mode
- Verify server logs for connection errors

**Events Not Tracked**:
- Check `DisableEvents` is false
- Verify `EventUri` is accessible
- Confirm network connectivity

## Platform Support

This SDK targets:
- **.NET 8.0+**: Runs on .NET 6.0 and higher
- **.NET Core 3.1+**: Runs on .NET Core 3.1 and later
- **.NET Framework 4.6.2+**: Runs on .NET Framework 4.6.2 and above
- **.NET Standard 2.0/2.1**: Runs in any .NET Standard 2.x project

**Note**: `System.Text.Json` is required and included as a dependency for platforms that don't include it.

## Official Resources

- **GitHub**: https://github.com/featbit/featbit-dotnet-sdk
- **NuGet**: https://www.nuget.org/packages/FeatBit.ServerSdk
- **Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/dotnet
- **Getting Started**: https://docs.featbit.co/getting-started/connect-an-sdk#net
- **Examples**: https://github.com/featbit/featbit-samples

## Support

- **Slack Community**: [Join FeatBit Slack](https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ)
- **Issues**: [Submit on GitHub](https://github.com/featbit/featbit/issues/new)
