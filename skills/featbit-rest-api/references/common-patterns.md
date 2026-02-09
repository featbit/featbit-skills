# Common Patterns

Workflow examples, error code reference, and best practices for FeatBit REST API usage.

---

## End-to-End Workflow

A complete setup flow: create project → add environment → create feature flag.

```bash
# Set variables
BASE_URL="https://your-featbit-instance.com"
TOKEN="your-jwt-token"

# Step 1: Create a project (auto-creates Prod and Dev environments)
PROJECT_RESPONSE=$(curl -s -X POST "$BASE_URL/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"My App","key":"my-app"}')

PROJECT_ID=$(echo $PROJECT_RESPONSE | jq -r '.data.id')
DEV_ENV_ID=$(echo $PROJECT_RESPONSE | jq -r '.data.environments[1].id')

echo "Project ID: $PROJECT_ID"
echo "Dev Env ID: $DEV_ENV_ID"

# Step 2: Create additional environment (optional)
STAGING_RESPONSE=$(curl -s -X POST "$BASE_URL/api/v1/projects/$PROJECT_ID/envs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"Staging","key":"staging","description":"QA environment"}')

STAGING_ENV_ID=$(echo $STAGING_RESPONSE | jq -r '.data.id')
echo "Staging Env ID: $STAGING_ENV_ID"

# Step 3: Create a feature flag in the dev environment
FLAG_RESPONSE=$(curl -s -X POST "$BASE_URL/api/v1/envs/$DEV_ENV_ID/feature-flags" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "New Feature",
    "key": "new-feature",
    "isEnabled": false,
    "variationType": "boolean",
    "variations": [
      {"id": "var-1", "name": "On", "value": "true"},
      {"id": "var-2", "name": "Off", "value": "false"}
    ],
    "enabledVariationId": "var-1",
    "disabledVariationId": "var-2",
    "tags": ["backend"]
  }')

echo "Flag created: $(echo $FLAG_RESPONSE | jq -r '.data.key')"

# Step 4: Verify by getting the project
curl -s -X GET "$BASE_URL/api/v1/projects/$PROJECT_ID" \
  -H "Authorization: Bearer $TOKEN" | jq '.data'
```

---

## Error Handling Pattern

Always check the `success` field in responses:

```bash
RESPONSE=$(curl -s -X POST "$BASE_URL/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"","key":""}')

SUCCESS=$(echo $RESPONSE | jq -r '.success')
if [ "$SUCCESS" = "false" ]; then
  echo "Error occurred:"
  echo $RESPONSE | jq '.errors[]'
else
  echo "Project created: $(echo $RESPONSE | jq -r '.data.id')"
fi
```

---

## Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `Required:{field}` | Required field is missing | Provide the required field in the request body |
| `Invalid:{field}` | Field value is invalid | Check field format and constraints |
| `KeyHasBeenUsed` | Key already exists in the scope | Use a different unique key |
| `NotFound` | Resource not found | Verify the resource ID exists |
| `Unauthorized` | Authentication failed | Check token/API key validity |
| `Forbidden` | Permission denied | Verify user has the required permission |

---

## Key Naming Conventions

| Convention | Example | Recommended |
|-----------|---------|-------------|
| kebab-case | `new-checkout-flow` | ✅ Yes |
| dot.notation | `checkout.new.flow` | ✅ Yes |
| snake_case | `new_checkout_flow` | ✅ Yes |
| camelCase | `newCheckoutFlow` | ✅ Yes |
| Spaces/special chars | `new checkout flow!` | ❌ No |

**Key pattern**: `^[a-zA-Z0-9._-]+$`

---

## Variation Types Quick Reference

| Type | Description | Example Values | Use Case |
|------|-------------|----------------|----------|
| `boolean` | True/false flags | `"true"`, `"false"` | Feature on/off toggles |
| `string` | Text values | `"basic"`, `"pro"` | Plan tiers, themes, versions |
| `number` | Numeric values | `"42"`, `"3.14"` | Rate limits, thresholds |
| `json` | Complex objects | `"{\"color\":\"blue\"}"` | UI config, complex settings |

**Note**: All values are stored and transmitted as strings. SDKs parse them based on `variationType`.

---

## Permission Reference

| Permission | Scope | Used By |
|-----------|-------|---------|
| `CreateProject` | Organization | Create Project API |
| `CanAccessProject` | Project | Get Project API |
| `CreateEnv` | Project | Create Environment API |
| `CreateFlag` | Environment | Create Feature Flag API |

---

## Tips for Automation

1. **Capture IDs from responses** — chain API calls by extracting IDs with `jq`
2. **Use environment variables** — store `BASE_URL`, `TOKEN`, and resource IDs
3. **Idempotency** — keys must be unique; check for `KeyHasBeenUsed` errors before retrying
4. **Batch creation** — create flags in a loop for multi-environment setups:
   ```bash
   for ENV_ID in $DEV_ENV_ID $STAGING_ENV_ID $PROD_ENV_ID; do
     curl -s -X POST "$BASE_URL/api/v1/envs/$ENV_ID/feature-flags" \
       -H "Content-Type: application/json" \
       -H "Authorization: Bearer $TOKEN" \
       -d "$FLAG_JSON"
   done
   ```
5. **CI/CD integration** — use OpenAPI Key authentication for non-interactive workflows
