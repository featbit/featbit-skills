---
name: featbit-dotnet-sdk
description: Expert guidance for integrating FeatBit .NET Server SDK in ASP.NET Core applications and console applications. Use when users ask about .NET SDK integration, dependency injection, feature flag evaluation in C#, or .NET-specific implementation.
appliesTo:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/Program.cs"
  - "**/Startup.cs"
---

# FeatBit .NET Server SDK Integration Expert

You are an expert on integrating the FeatBit .NET Server SDK into .NET applications, with deep knowledge of both ASP.NET Core and console application patterns.

## When to Use This Skill

Activate when users:
- Ask about integrating FeatBit in .NET applications
- Need help with ASP.NET Core DI (dependency injection) setup
- Want to evaluate feature flags in C# code
- Ask about console apps, worker services, or background services
- Need examples of different flag variation types (bool, string, int, double, JSON)
- Want to track custom events for A/B testing
- Need configuration guidance for appsettings.json

## Official Resources

- **GitHub Repository**: https://github.com/featbit/featbit-dotnet-sdk
- **NuGet Package**: https://www.nuget.org/packages/FeatBit.ServerSdk
- **SDK Documentation**: https://docs.featbit.co/sdk-docs/server-side-sdks/dotnet

## Installation

```bash
dotnet add package FeatBit.ServerSdk
```

## ASP.NET Core Integration

### Basic Setup with Dependency Injection

```csharp
using FeatBit.Sdk.Server.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

// Add FeatBit service
builder.Services.AddFeatBit(options =>
{
    options.EnvSecret = "<replace-with-your-env-secret>";
    options.StreamingUri = new Uri("wss://app-eval.featbit.co");
    options.EventUri = new Uri("https://app-eval.featbit.co");
    options.StartWaitTime = TimeSpan.FromSeconds(3);
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

### Configuration from appsettings.json

```json
{
  "FeatBit": {
    "EnvSecret": "<your-env-secret>",
    "StreamingUri": "wss://app-eval.featbit.co",
    "EventUri": "https://app-eval.featbit.co",
    "StartWaitTime": "00:00:03"
  }
}
```

```csharp
builder.Services.AddFeatBit(builder.Configuration.GetSection("FeatBit"));
```

### Using in Controllers (Authenticated Users)

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
        var user = FbUser.Builder(User.Identity.Name ?? "anonymous")
            .Name(User.Identity.Name)
            .Custom("role", "admin")
            .Custom("subscription", "premium")
            .Custom("country", "US")
            .Build();

        var isEnabled = _fbClient.BoolVariation("new-feature", user, defaultValue: false);
        
        return Ok(new { featureEnabled = isEnabled });
    }
}
```

### Using with Anonymous Users

```csharp
[HttpGet("public-feature")]
public IActionResult CheckPublicFeature()
{
    var sessionId = HttpContext.Session?.Id ?? Guid.NewGuid().ToString();
    var anonymousUser = FbUser.Builder($"anonymous-{sessionId}").Build();

    var flagValue = _fbClient.BoolVariation("public-beta-feature", anonymousUser, defaultValue: false);
    
    return Ok(new { betaAccess = flagValue });
}
```

## All Flag Variation Types

### Boolean Flags
```csharp
var isEnabled = _fbClient.BoolVariation("feature-key", user, defaultValue: false);
```

### String Flags
```csharp
var theme = _fbClient.StringVariation("theme-key", user, defaultValue: "default");
```

### Integer Flags
```csharp
var maxItems = _fbClient.IntVariation("max-items-key", user, defaultValue: 10);
```

### Double Flags
```csharp
var discountRate = _fbClient.DoubleVariation("discount-rate", user, defaultValue: 0.0);
```

### JSON Flags
```csharp
var config = _fbClient.JsonVariation("config-key", user, defaultValue: "{}");
var configObject = JsonSerializer.Deserialize<MyConfig>(config);
```

## Tracking Custom Events (A/B Testing)

```csharp
// Track a conversion event
_fbClient.TrackMetric(user, "purchase-completed");

// Track with numeric value
_fbClient.TrackMetric(user, "revenue", 99.99);
```

## Console Application Integration

### Basic Console App Setup

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Options;
using FeatBit.Sdk.Server.Model;

var options = new FbOptionsBuilder("<your-env-secret>")
    .StreamingUri(new Uri("wss://app-eval.featbit.co"))
    .EventUri(new Uri("https://app-eval.featbit.co"))
    .StartWaitTime(TimeSpan.FromSeconds(3))
    .Build();

using var client = new FbClient(options);

if (!await client.InitializedAsync)
{
    Console.WriteLine("Failed to initialize FeatBit client");
    return;
}

var user = FbUser.Builder("user-id")
    .Name("User Name")
    .Custom("role", "admin")
    .Build();

var isEnabled = client.BoolVariation("feature-key", user, defaultValue: false);
Console.WriteLine($"Feature enabled: {isEnabled}");
```

### Worker Service Setup

```csharp
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
                services.AddFeatBit(hostContext.Configuration.GetSection("FeatBit"));
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
            var user = FbUser.Builder("worker-user").Build();
            var shouldProcess = _fbClient.BoolVariation("process-enabled", user, true);
            
            if (shouldProcess)
            {
                _logger.LogInformation("Processing enabled at: {time}", DateTimeOffset.Now);
            }
            
            await Task.Delay(60000, stoppingToken);
        }
    }
}
```

## Best Practices

### 1. Client Lifecycle
- **ASP.NET Core**: Use DI with `AddFeatBit()` - client is managed as singleton
- **Console Apps**: Create one client instance and reuse it throughout the app
- **Always wait for initialization**: Check `await client.InitializedAsync` before use

### 2. User Context
- Use meaningful user keys (username, email, user ID)
- For anonymous users, use session IDs or generate consistent identifiers
- Add custom attributes for targeting: role, subscription, country, etc.

### 3. Default Values
- Always provide sensible default values
- Defaults are used when:
  - SDK is not initialized
  - Flag doesn't exist
  - Network issues occur
  - Evaluation fails

### 4. Error Handling
```csharp
try
{
    var isEnabled = _fbClient.BoolVariation("feature-key", user, defaultValue: false);
    // Use feature
}
catch (Exception ex)
{
    _logger.LogError(ex, "Failed to evaluate feature flag");
    // Fall back to default behavior
}
```

### 5. Performance
- Client maintains in-memory cache of all flags
- Flag evaluation is local and extremely fast (microseconds)
- Real-time updates via WebSocket
- No network call on every evaluation

## Configuration Options

```csharp
builder.Services.AddFeatBit(options =>
{
    // Required
    options.EnvSecret = "your-secret";
    
    // Optional - URLs
    options.StreamingUri = new Uri("wss://app-eval.featbit.co");
    options.EventUri = new Uri("https://app-eval.featbit.co");
    
    // Optional - Timing
    options.StartWaitTime = TimeSpan.FromSeconds(3);
    options.ReconnectInterval = TimeSpan.FromSeconds(15);
    
    // Optional - Features
    options.DisableEvents = false; // Set true to disable event tracking
    options.Offline = false; // Set true for offline mode
    
    // Optional - Event buffering
    options.EventFlushInterval = TimeSpan.FromSeconds(5);
    options.EventMaxInQueue = 10000;
});
```

## Common Scenarios

### Scenario 1: Gradual Rollout in Web API
```csharp
// Enable new payment processor for 10% of users
var user = FbUser.Builder(userId).Build();
var useNewProcessor = _fbClient.BoolVariation("new-payment-processor", user, false);

if (useNewProcessor)
{
    return await _newPaymentService.ProcessAsync(payment);
}
else
{
    return await _legacyPaymentService.ProcessAsync(payment);
}
```

### Scenario 2: Feature Toggle for Maintenance
```csharp
// Disable certain features during maintenance
var maintenanceUser = FbUser.Builder("system").Build();
var isMaintenanceMode = _fbClient.BoolVariation("maintenance-mode", maintenanceUser, false);

if (isMaintenanceMode)
{
    return StatusCode(503, "Service under maintenance");
}
```

### Scenario 3: Remote Configuration
```csharp
// Dynamic configuration without deployment
var user = FbUser.Builder(userId).Build();
var maxRetries = _fbClient.IntVariation("max-retry-attempts", user, 3);
var timeoutSeconds = _fbClient.IntVariation("api-timeout-seconds", user, 30);

var policy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(maxRetries, _ => TimeSpan.FromSeconds(1));
```

### Scenario 4: A/B Testing with Metrics
```csharp
var user = FbUser.Builder(userId).Build();
var checkoutFlow = _fbClient.StringVariation("checkout-flow", user, "original");

// User completes purchase
if (purchaseSuccessful)
{
    _fbClient.TrackMetric(user, "purchase-completed");
    _fbClient.TrackMetric(user, "purchase-revenue", purchaseAmount);
}
```

## Troubleshooting

### Client Not Initializing
- Check `EnvSecret` is correct
- Verify network connectivity to FeatBit server
- Check firewall rules for WebSocket connections
- Increase `StartWaitTime` if needed

### Flags Not Updating
- Ensure WebSocket connection is established
- Check server logs for connection errors
- Verify SDK is not in offline mode

### Events Not Being Tracked
- Ensure `DisableEvents` is false
- Check `EventUri` is accessible
- Verify network connectivity

## Migration from Other SDKs

If migrating from LaunchDarkly or other SDKs:
- User context is similar but FeatBit uses `FbUser.Builder()`
- Method names are similar: `BoolVariation()`, `StringVariation()`, etc.
- Configuration is similar but uses `FbOptionsBuilder` or DI

## Additional Resources

- SDK GitHub: https://github.com/featbit/featbit-dotnet-sdk
- Sample Apps: https://github.com/featbit/featbit-samples
- Documentation: https://docs.featbit.co/sdk-docs/server-side-sdks/dotnet
