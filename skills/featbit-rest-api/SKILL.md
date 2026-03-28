---
name: featbit-rest-api
description: Expert guidance for using the FeatBit REST API to manage projects, environments, and feature flags programmatically. Use when user asks about "FeatBit API", "REST API", "create project API", "create environment API", "create feature flag API", "API authentication", "OpenAPI key", or needs to automate FeatBit operations via HTTP endpoints. Do not use for SDK integration, deployment configuration, or flag evaluation questions.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: api-reference
---

# FeatBit REST API

Expert guidance for interacting with FeatBit services through the REST API. Use these endpoints to manage projects, environments, feature flags, and more programmatically.

## Execution Procedure

```
1. AUTHENTICATE
   - Obtain credentials: JWT Bearer token (user auth) OR OpenAPI key (machine-to-machine)
   - Read references/authentication.md for token acquisition details

2. IDENTIFY OPERATION
   - Determine the resource: project | environment | feature flag
   - Determine the action: list | create | get

3. READ REFERENCE
   - Project operations → references/projects-api.md
   - Environment operations → references/environments-api.md
   - Feature flag operations → references/feature-flags-api.md
   - Workflow patterns → references/common-patterns.md

4. CONSTRUCT API CALL
   - Base URL: https://{host}/api/v1/{resource}
   - Set headers: Content-Type: application/json, Authorization
   - Build request body per reference schema

5. HANDLE RESPONSE
   - Check "success" field in response wrapper
   - On success: extract "data" payload
   - On failure: inspect "errors" array, map HTTP status code, retry or report
```

## Base URL and Versioning

```
https://your-featbit-instance.com/api/v{version}
```

- **Current Version**: `v1` (URL pattern: `/api/v1/{resource}`)
- Replace `your-featbit-instance.com` with your actual FeatBit host

## Authentication

Authenticate using one of two methods:

### 1. JWT Bearer Token (User Authentication)

```http
Authorization: Bearer {jwt_token}
```

Use for interactive sessions and user-scoped operations.

### 2. OpenAPI Key (Machine-to-Machine)

```http
Authorization: {api_key}
```

Use for CI/CD pipelines, automation scripts, and service-to-service calls.

**Detailed Guide**: [references/authentication.md](references/authentication.md)

## Response Format

Expect all API responses in this standardized wrapper:

```json
{
  "success": true,
  "data": { ... },
  "errors": []
}
```

On failure:

```json
{
  "success": false,
  "data": null,
  "errors": [
    { "code": "Required:name", "message": "The name field is required" }
  ]
}
```

## Common HTTP Status Codes

| Code | Description |
|------|-------------|
| `200` | Success |
| `400` | Bad Request — validation errors |
| `401` | Unauthorized — missing or invalid authentication |
| `403` | Forbidden — insufficient permissions |
| `404` | Not Found — resource doesn't exist |
| `500` | Internal Server Error |

## API Endpoints

### Project Management

| Method | Endpoint | Description | Permission |
|--------|----------|-------------|------------|
| `GET` | `/api/v1/projects` | List all projects in the organization | `CanAccessProject` |
| `POST` | `/api/v1/projects` | Create a new project | `CreateProject` |
| `GET` | `/api/v1/projects/{projectId}` | Get project details with environments | `CanAccessProject` |

**Quick example — Create a project:**

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{"name": "E-Commerce Platform", "key": "ecommerce"}'
```

- Auto-generates two environments: **Prod** and **Dev**
- Each environment gets a Server Key and Client Key

**Full Reference**: [references/projects-api.md](references/projects-api.md)

### Environment Management

| Method | Endpoint | Description | Permission |
|--------|----------|-------------|------------|
| `POST` | `/api/v1/projects/{projectId}/envs` | Create a new environment | `CreateEnv` |

**Quick example — Create an environment:**

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/projects/{projectId}/envs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{"name": "Staging", "key": "staging", "description": "QA environment"}'
```

- Auto-generates Server Key and Client Key
- Key must be unique within the project

**Full Reference**: [references/environments-api.md](references/environments-api.md)

### Feature Flag Management

| Method | Endpoint | Description | Permission |
|--------|----------|-------------|------------|
| `POST` | `/api/v1/envs/{envId}/feature-flags` | Create a new feature flag | `CreateFlag` |

**Quick example — Create a boolean feature flag:**

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/envs/{envId}/feature-flags" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{
    "name": "New Checkout Flow",
    "key": "new-checkout-flow",
    "isEnabled": false,
    "variationType": "boolean",
    "variations": [
      {"id": "var-on", "name": "On", "value": "true"},
      {"id": "var-off", "name": "Off", "value": "false"}
    ],
    "enabledVariationId": "var-on",
    "disabledVariationId": "var-off",
    "tags": ["checkout"]
  }'
```

- Specify variation type as one of: `boolean`, `string`, `number`, `json`
- Key pattern: `^[a-zA-Z0-9._-]+$`

**Full Reference**: [references/feature-flags-api.md](references/feature-flags-api.md)

## Workflow Example

Follow this setup flow to create a project, add an environment, and create a flag:

```bash
# 1. Create a project (auto-creates Prod and Dev environments)
PROJECT=$(curl -s -X POST "$BASE_URL/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"My App","key":"my-app"}')

PROJECT_ID=$(echo $PROJECT | jq -r '.data.id')
DEV_ENV_ID=$(echo $PROJECT | jq -r '.data.environments[1].id')

# 2. (Optional) Create a staging environment
curl -s -X POST "$BASE_URL/api/v1/projects/$PROJECT_ID/envs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"Staging","key":"staging","description":"QA environment"}'

# 3. Create a feature flag in the dev environment
curl -s -X POST "$BASE_URL/api/v1/envs/$DEV_ENV_ID/feature-flags" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name":"New Feature","key":"new-feature","isEnabled":false,
    "variationType":"boolean",
    "variations":[{"id":"v1","name":"On","value":"true"},{"id":"v2","name":"Off","value":"false"}],
    "enabledVariationId":"v1","disabledVariationId":"v2","tags":["backend"]
  }'
```

**More Patterns**: [references/common-patterns.md](references/common-patterns.md)

## Best Practices

1. **Key Naming**: Use descriptive, kebab-case keys (e.g., `new-checkout-flow`)
2. **Variation IDs**: Use GUIDs or unique strings to avoid collisions
3. **Tags**: Organize flags by feature, team, or category
4. **Secrets**: Store API keys securely; never commit them to version control
5. **Test First**: Always test API calls in a development environment before production
6. **Environments**: Maintain separate environments for dev, staging, and production

## Reference Guides

- [references/authentication.md](references/authentication.md) — JWT vs OpenAPI Key, obtaining tokens
- [references/projects-api.md](references/projects-api.md) — Create project, get project (full request/response)
- [references/environments-api.md](references/environments-api.md) — Create environment (full request/response)
- [references/feature-flags-api.md](references/feature-flags-api.md) — Create feature flag, variation types, targeting
- [references/common-patterns.md](references/common-patterns.md) — Workflow examples, error codes, variation types reference

## Official Resources

- **Interactive API Docs**: https://app-api.featbit.co/docs/index.html
- **OpenAPI Schema**: https://app-api.featbit.co/swagger/OpenApi/swagger.json
