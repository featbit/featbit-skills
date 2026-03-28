---
name: featbit-sdks
description: SDK language router for FeatBit. Routes SDK questions to the correct language-specific skill. Use when user asks about "which SDK to use", "SDK overview", "all FeatBit SDKs", or mentions FeatBit SDK without specifying a language. Do not use for deployment, REST API, observability, or documentation questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: sdk-integration
---

# FeatBit SDK Router

Use this skill only to choose the correct language-specific SDK skill. Do not keep implementation details for every SDK in this file.

## Execution Procedure

```pseudo
1. Infer the runtime from the user request.
2. Look up the Routing Table below → activate exactly one language-specific SDK skill.
3. If the runtime is ambiguous, ask which language or framework the user is using.
4. After the language is known, stop here and continue in the language-specific skill.
```

## Routing Table

- `.NET` or `.cs`, `.csproj`, ASP.NET Core: `featbit-sdks-dotnet`
- `Java` or `.java`, Maven, Gradle: `featbit-sdks-java`
- `Go` or `.go`, Golang: `featbit-sdks-go`
- `Node.js` server or server-side TypeScript/JavaScript: `featbit-sdks-node`
- Browser JavaScript or vanilla JS: `featbit-sdks-javascript`
- React web: `featbit-sdks-react`
- React Native mobile: `featbit-sdks-react-native`
- Python, Flask, Django, FastAPI: `featbit-sdks-python`
