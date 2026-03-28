---
name: featbit-sdks-dotnet
description: Expert guidance for integrating FeatBit .NET Server SDK into C# and ASP.NET Core applications. Use when user asks about ".NET SDK", "C# feature flags", "ASP.NET Core FeatBit", "dotnet feature flags", or mentions .cs or .csproj files. Do not use for Node.js, Python, Java, Go, or client-side JavaScript questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit .NET Server SDK

Use for server-side .NET applications — Console apps, Web API, Worker Services, and ASP.NET Core — that need real-time feature flag evaluation. Establishes one persistent streaming connection, evaluates all feature flags locally, and returns results with near-zero latency on each call. Do not use for Blazor WebAssembly or MAUI — those require a client-side SDK.

## Execution Procedure

```
1. Install package → dotnet add package FeatBit.ServerSdk
2. Configure FbClient with EnvSecret, Streaming URI, Event URI
3. Evaluate first feature flag → BoolVariation / StringVariation
   - IF client.Initialized == false → verify EnvSecret and URIs, retry
4. IF ASP.NET Core project → switch to DI registration (AddFeatBit)
5. IF user mentions OpenFeature → read references/openfeature-integration.md
```

## Setup Workflow

**Step 1: Install the package**

Run:

```bash
dotnet add package FeatBit.ServerSdk
```

**Step 2: Initialize the client**

Use this minimal setup:

```csharp
using FeatBit.Sdk.Server;
using FeatBit.Sdk.Server.Model;
using FeatBit.Sdk.Server.Options;

var options = new FbOptionsBuilder("<your-env-secret>")
  .Streaming(new Uri("ws://localhost:5100"))
  .Event(new Uri("http://localhost:5100"))
  .Build();

var client = new FbClient(options);
var user = FbUser.Builder("anonymous").Build();
var flagValue = client.BoolVariation("flag-key", user, defaultValue: false);
await client.CloseAsync();
```

**Step 3: Validate the integration**

If `client.Initialized` is false or the first variation returns the fallback unexpectedly, verify `EnvSecret`, `Streaming` URI, and `Event` URI, then retry.

**Step 4: If this is ASP.NET Core, use dependency injection**

Why: ensures a single `FbClient` is shared across the application lifetime and is disposed automatically when the host shuts down.

Use this pattern:

```csharp
using FeatBit.Sdk.Server.DependencyInjection;

builder.Services.AddFeatBit(options =>
{
  options.EnvSecret = "<your-env-secret>";
  options.StreamingUri = new Uri("ws://localhost:5100");
  options.EventUri = new Uri("http://localhost:5100");
});
```

Inject `IFbClient` where feature flag evaluation is needed.

**Step 5: If the user mentions OpenFeature, read `references/openfeature-integration.md`.**

## Feature Flag Evaluation

After the client is initialized, evaluate a feature flag with a user and a fallback value:

```csharp
const string flagKey = "game-runner";
var user = FbUser.Builder("anonymous").Build();

var boolVariation = client.BoolVariation(flagKey, user, defaultValue: false);
var boolVariationDetail = client.BoolVariationDetail(flagKey, user, defaultValue: false);
```

Use `BoolVariation` when only the feature flag value is needed. Use `BoolVariationDetail` when the user also asks why a value was returned.

## User Custom Properties

Add custom properties to `FbUser` when targeting rules depend on user attributes beyond `key` and `name`:

```csharp
var user = FbUser.Builder("a-unique-key-of-user")
  .Name("bob")
  .Custom("age", "15")
  .Custom("country", "FR")
  .Build();
```

Use built-in properties for stable identity fields. Use `Custom(key, value)` for targeting attributes that must be referenced in feature flag rules.
