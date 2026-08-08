# FeatBit REST API sources

Use this reference for programmatic management of FeatBit resources, direct feature flag evaluation on platforms without an official SDK, or Track Insights calls. Use an official SDK for normal application evaluation whenever one supports the target platform.

| API area | Authoritative source | Use |
|---|---|---|
| Management API introduction | https://docs.featbit.co/api-docs/using-featbit-rest-api | Read for resource hierarchy, request basics, and supported authentication. |
| Cloud Management OpenAPI | https://app-api.featbit.co/swagger/OpenApi/swagger.json | Inspect the current cloud paths, parameters, request bodies, responses, and schemas. |
| Cloud Management ReDoc | https://app-api.featbit.co/docs/index.html | Browse the same Management contract in rendered form. |
| API access tokens | https://docs.featbit.co/integrations/api-access-tokens | Create and scope personal or service tokens for Management API calls. |
| Flag Evaluation API | https://docs.featbit.co/api-docs/flag-evaluation-api | Evaluate feature flags directly when no suitable official SDK exists. |
| Track Insights API | https://docs.featbit.co/api-docs/track-insights-api | Send documented evaluation and metric insight payloads. |
| Implementation source | https://github.com/featbit/featbit | Inspect the matching release tag when the contract does not explain behavior. |

For a self-hosted instance, prefer its own `/swagger/OpenApi/swagger.json` or `/docs` endpoint and its matching release source over the cloud schema. Keep all credentials secret and inspect only the relevant OpenAPI operation and referenced schemas at task time.
