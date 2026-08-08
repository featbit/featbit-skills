# FeatBit SDK sources

Use this reference when application code needs to evaluate feature flags, receive updates, or send evaluation insights and custom events. Select a server-side SDK for trusted multi-user backends and a client-side SDK for single-user browser, desktop, or mobile applications. Read the [official SDK overview](https://docs.featbit.co/sdk/overview) and the selected repository's current README.

## Official SDKs

| SDK | GitHub repository | Use |
|---|---|---|
| JavaScript client | https://github.com/featbit/featbit-js-client-sdk | Use in browser-based JavaScript or TypeScript applications. |
| React client | https://github.com/featbit/featbit-react-client-sdk | Use in React and Next.js browser applications. |
| React Native client | https://github.com/featbit/featbit-react-native-sdk | Use in React Native mobile applications. |
| Node.js server | https://github.com/featbit/featbit-node-server-sdk | Use in trusted Node.js backends and server frameworks. |
| .NET server | https://github.com/featbit/featbit-dotnet-sdk | Use in multi-user .NET and ASP.NET Core backends. |
| .NET client | https://github.com/featbit/featbit-dotnet-client-sdk | Use in single-user .NET desktop, mobile, or embedded applications. |
| Java server | https://github.com/featbit/featbit-java-sdk | Use in trusted Java and Spring Boot backends. |
| Python server | https://github.com/featbit/featbit-python-sdk | Use in trusted Python services and web backends. |
| Go server | https://github.com/featbit/featbit-go-sdk | Use in trusted Go services and APIs. |

## Official OpenFeature providers

| Provider | GitHub repository | Use |
|---|---|---|
| JavaScript client | https://github.com/featbit/openfeature-provider-js-client | Use OpenFeature with FeatBit in browser applications. |
| Node.js server | https://github.com/featbit/openfeature-provider-node-server | Use OpenFeature with FeatBit in Node.js backends. |
| .NET server | https://github.com/featbit/openfeature-provider-dotnet-server | Use OpenFeature with the FeatBit .NET server SDK. |
| Java server | https://github.com/featbit/featbit-openfeature-provider-java-server | Use OpenFeature with the FeatBit Java server SDK. |

Treat the SDKs and providers listed by the official overview as the supported catalog. If the target platform is absent, use the [Flag Evaluation API](https://docs.featbit.co/api-docs/flag-evaluation-api) and [Track Insights API](https://docs.featbit.co/api-docs/track-insights-api) instead of inventing an SDK package.
