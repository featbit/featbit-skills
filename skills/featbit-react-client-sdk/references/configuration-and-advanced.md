# Configuration and Advanced Topics

## Populating the SDK with fallback flag values

As mentioned above, you can use the **options.bootstrap** option to populate the SDK with default values for your flags. This option is useful when you want to provide default values for your flags before the SDK initializes.
If a flag is not available from the SDK, the SDK uses the default value you provide in the bootstrap object.

```javascript
options: {
  bootstrap: [
    { 
        "id": "my-flag",
        "variation": "on"
    },
    {
      "id": "second-flag",
      "variation": "AlphaGo"
    }
  ]
}
```

## Flag keys
FeatBit primarily identifies feature flags by a key which must contain only alphanumeric characters, dots (.), underscores (_), and dashes (-). These keys are used across all of our APIs as well as in the SDKs to identify flags.

However, JavaScript and React cannot access keys with a dot notation, so the React SDK can change all flag keys to camel case (you need to activate this with the **reactOptions.useCamelCaseFlagKeys** parameter). A flag with key `dev-flag-test` is accessible as `flags.devFlagTest`. This notation **flags['dev-flag-test']** should be used if useCamelCaseFlagKeys is disabled, which is by default.

Be aware, by activating useCamelCaseFlagKeys, you would see following problems:

- It is possible to induce a key collision if there are multiple flag keys which resolve to the same camel-case key. For example, `dev-flag-test` and `dev.flag.test` are unique keys, but the React SDK changes them to the same camel-case key.
- If a flag key contains three or more capital letters in a row, the SDK automatically converts all letters between the first and last capital letter to lower case. For example, the SDK converts a flag with the key `devQAFlagTest` to `devQaFlagTest`. If you use devQAFlagTest with the useFlags() hook, the SDK does not find the flag.
- Because the camel-case functionality is implemented in the React SDK instead of in the underlying JavaScript SDK, the underlying client object and functionality provided by the JavaScript SDK reflect flag keys in their original format. Only React-specific contexts such as your injected props use camel case.

**Note:** If you've enabled the useCamelCaseFlagKeys option to true, attempting to access a flag using its original key will trigger a warning message in the console:
**You're attempting to access a flag with its original keyId: xxx, even though useCamelCaseFlagKeys is set to true.**

## Importing types

In addition to its own bundled types, the React SDK uses types from `@featbit/js-client-sdk`, these types are re-exported by the React SDK. You can import these types directly from the React SDK.
