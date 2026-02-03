# OpenFeature Integration for FeatBit .NET Server SDK

This is the [OpenFeature](https://github.com/open-feature/dotnet-sdk) provider for the [FeatBit .NET Server SDK](https://github.com/featbit/featbit-dotnet-sdk).

## Installation

```bash
dotnet add package FeatBit.OpenFeature.ServerProvider
```

**NuGet Package**: https://www.nuget.org/packages/FeatBit.OpenFeature.ServerProvider/

## Quick Start

```csharp
using FeatBit.OpenFeature.ServerProvider;
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Options;
using OpenFeature;
using OpenFeature.Model;

// Setup SDK options
var options = new FbOptionsBuilder("<replace-with-your-env-secret>")
    .Event(new Uri("<replace-with-your-event-url>"))
    .Streaming(new Uri("<replace-with-your-streaming-url>"))
    .Build();

// Creates a new client instance that connects to FeatBit with the custom option.
var fbClient = new FbClient(options);

// Use the FeatBit client with OpenFeature
var provider = new FeatBitProvider(fbClient);
await Api.Instance.SetProviderAsync(provider);
var client = Api.Instance.GetClient();

// Flag to be evaluated
const string flagKey = "game-runner";

// Create an evaluation context
var context = EvaluationContext.Builder().SetTargetingKey("anonymous").Build();

// Evaluate a boolean flag for a given context
var boolVariation = await client.GetBooleanValueAsync(flagKey, defaultValue: false, context);
Console.WriteLine($"flag '{flagKey}' returns {boolVariation} for {context.TargetingKey}");

// Evaluate a boolean flag for a given context with evaluation detail
var boolVariationDetail = await client.GetBooleanDetailsAsync(flagKey, defaultValue: false, context);
Console.WriteLine(
    $"flag '{flagKey}' returns {boolVariationDetail.Value} for {context.TargetingKey}. " +
    $"Reason: {boolVariationDetail.Reason}"
);

// Optionally track the event with a metric for experimentation
var trackingDetail = TrackingEventDetails.Builder().SetValue(1.0).Build();
client.Track("games-run", context, trackingDetail);

// Shut down OpenFeature
await Api.Instance.ShutdownAsync();

// Close the client to ensure that all insights are sent out before the app exits
await fbClient.CloseAsync();
```

## Official Resources

- **GitHub**: https://github.com/featbit/openfeature-provider-dotnet-server
- **NuGet**: https://www.nuget.org/packages/FeatBit.OpenFeature.ServerProvider/
- **Examples**: [Console App Example](https://github.com/featbit/openfeature-provider-dotnet-server/tree/main/examples/ConsoleApp)

## Support

- **FeatBit Slack**: [Join Community](https://join.slack.com/t/featbit/shared_invite/zt-1ew5e2vbb-x6Apan1xZOaYMnFzqZkGNQ)
- **Issues**: [Submit on GitHub](https://github.com/featbit/openfeature-provider-dotnet-server/issues/new)
