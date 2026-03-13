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

## When to Use This Skill

Use for server-side .NET applications — Console apps, Web API, Worker Services, and ASP.NET Core — that need real-time feature flag evaluation.

Why server-side SDK: establishes one persistent streaming connection, evaluates all flags locally, and returns results with near-zero latency on each call. Do not use for Blazor WebAssembly or MAUI — those require a client-side SDK.

## Source

https://github.com/featbit/featbit-dotnet-sdk

## Setup Workflow

Copy and track progress:
- [ ] Step 1: Install the package
- [ ] Step 2: Initialize the client
- [ ] Step 3: Evaluate the first feature flag
- [ ] Step 4: If this is ASP.NET Core, switch to dependency injection
- [ ] Step 5: If advanced behavior is needed, read the specific reference or README section

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

## Feature Flag Evaluation

After the client is initialized, evaluate a feature flag with a user and a fallback value:

```csharp
const string flagKey = "game-runner";
var user = FbUser.Builder("anonymous").Build();

var boolVariation = client.BoolVariation(flagKey, user, defaultValue: false);
var boolVariationDetail = client.BoolVariationDetail(flagKey, user, defaultValue: false);
```

Use `BoolVariation` when only the flag value is needed. Use `BoolVariationDetail` when the user also asks why a value was returned.

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

## Read Next Only When Needed

- Read the official README section for [evaluating flags](https://github.com/featbit/featbit-dotnet-sdk#evaluating-flags) when the user asks how to evaluate a feature flag, inspect reasons, or choose the correct variation method.
- Read the official README section for [FbUser](https://github.com/featbit/featbit-dotnet-sdk#fbuser) when the user asks about user attributes, custom properties, targeting fields, or user construction patterns.
- Read `references/openfeature-integration.md` only when the user asks about OpenFeature or the FeatBit OpenFeature provider for .NET.
- Read the official README sections for [offline mode](https://github.com/featbit/featbit-dotnet-sdk#offline-mode), [disable events collection](https://github.com/featbit/featbit-dotnet-sdk#disable-events-collection), [experiments](https://github.com/featbit/featbit-dotnet-sdk#experiments), and [dependency injection](https://github.com/featbit/featbit-dotnet-sdk#dependency-injection) only when those topics are requested.
