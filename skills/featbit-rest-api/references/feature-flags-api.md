# Feature Flags API

APIs for managing FeatBit feature flags. Feature flags enable you to control feature rollouts, perform A/B testing, and manage configuration dynamically.

---

## Create a Feature Flag

Create a new feature flag within an environment.

### Endpoint

```http
POST /api/v1/envs/{envId}/feature-flags
```

### Authorization

- **Permission**: `CreateFlag`
- **Scope**: Environment level

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `envId` | guid | Yes | The environment ID where the feature flag will be created |

### Request Headers

```http
Content-Type: application/json
Authorization: Bearer {jwt_token}
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Display name of the feature flag |
| `key` | string | Yes | Unique identifier (pattern: `^[a-zA-Z0-9._-]+$`) |
| `isEnabled` | boolean | Yes | Initial enabled/disabled state |
| `description` | string | No | Optional description |
| `variationType` | string | Yes | Type: `boolean`, `string`, `number`, or `json` |
| `variations` | array | Yes | Array of variation objects (at least one required) |
| `variations[].id` | string | Yes | Unique identifier for the variation (typically a GUID) |
| `variations[].name` | string | Yes | Display name of the variation |
| `variations[].value` | string | Yes | The value to return for this variation |
| `enabledVariationId` | string | Yes | Variation to serve when flag is enabled |
| `disabledVariationId` | string | Yes | Variation to serve when flag is disabled |
| `tags` | array | No | Array of tag strings for categorization |

### Example: Boolean Flag

```json
{
  "name": "Enable New Checkout Flow",
  "key": "new-checkout-flow",
  "isEnabled": false,
  "description": "Enables the redesigned checkout experience",
  "variationType": "boolean",
  "variations": [
    {
      "id": "1a2b3c4d-5e6f-7890-abcd-ef1234567890",
      "name": "Enabled",
      "value": "true"
    },
    {
      "id": "9f8e7d6c-5b4a-3210-fedc-ba9876543210",
      "name": "Disabled",
      "value": "false"
    }
  ],
  "enabledVariationId": "1a2b3c4d-5e6f-7890-abcd-ef1234567890",
  "disabledVariationId": "9f8e7d6c-5b4a-3210-fedc-ba9876543210",
  "tags": ["checkout", "ui-redesign"]
}
```

### Example: Multi-variate String Flag

```json
{
  "name": "Pricing Tier Display",
  "key": "pricing-tier-display",
  "isEnabled": true,
  "description": "Controls which pricing tier to display",
  "variationType": "string",
  "variations": [
    { "id": "tier-basic-id", "name": "Basic Tier", "value": "basic" },
    { "id": "tier-pro-id", "name": "Pro Tier", "value": "pro" },
    { "id": "tier-enterprise-id", "name": "Enterprise Tier", "value": "enterprise" }
  ],
  "enabledVariationId": "tier-pro-id",
  "disabledVariationId": "tier-basic-id",
  "tags": ["pricing", "display"]
}
```

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "envId": "7c6d5e4f-3210-9876-fedc-ba9876543210",
    "revision": "rev12345-6789-0abc-def0-123456789abc",
    "name": "Enable New Checkout Flow",
    "description": "Enables the redesigned checkout experience",
    "key": "new-checkout-flow",
    "variationType": "boolean",
    "variations": [
      { "id": "1a2b3c4d-5e6f-7890-abcd-ef1234567890", "name": "Enabled", "value": "true" },
      { "id": "9f8e7d6c-5b4a-3210-fedc-ba9876543210", "name": "Disabled", "value": "false" }
    ],
    "targetUsers": [],
    "rules": [],
    "isEnabled": false,
    "disabledVariationId": "9f8e7d6c-5b4a-3210-fedc-ba9876543210",
    "fallthrough": {
      "dispatchKey": null,
      "includedInExpt": true,
      "variations": [
        {
          "id": "1a2b3c4d-5e6f-7890-abcd-ef1234567890",
          "rollout": [0, 1],
          "exptRollout": 1
        }
      ]
    },
    "exptIncludeAllTargets": true,
    "tags": ["checkout", "ui-redesign"],
    "isArchived": false,
    "createdAt": "2026-02-09T11:00:00Z",
    "updatedAt": "2026-02-09T11:00:00Z",
    "createdBy": "user-guid-1234-5678-90ab-cdef",
    "updatedBy": "user-guid-1234-5678-90ab-cdef"
  },
  "errors": []
}
```

### Error Response (400 Bad Request)

```json
{
  "success": false,
  "data": null,
  "errors": [
    { "code": "Invalid:key", "message": "The key format is invalid. Must match pattern: ^[a-zA-Z0-9._-]+$" },
    { "code": "KeyHasBeenUsed", "message": "The key has already been used in this environment" },
    { "code": "Invalid:enabledVariationId", "message": "The enabledVariationId must match a variation ID in the variations array" }
  ]
}
```

### Validation Rules

#### Required Fields

- `name`: Cannot be empty
- `key`: Cannot be empty, must match `^[a-zA-Z0-9._-]+$`, unique within environment
- `variationType`: Must be one of: `boolean`, `string`, `number`, `json`
- `variations`: Must contain at least one variation
- `enabledVariationId`: Must reference a valid variation ID
- `disabledVariationId`: Must reference a valid variation ID

#### Variation Validation

Each variation must have:
- `id`: Non-empty string (typically GUID)
- `name`: Non-empty string
- `value`: Can be any string (interpreted based on `variationType`)

### Response Fields Explained

| Field | Description |
|-------|-------------|
| `revision` | Unique GUID for version tracking |
| `targetUsers` | User-specific targeting rules (empty on creation) |
| `rules` | Conditional targeting rules (empty on creation) |
| `fallthrough` | Default serving rule when no specific targeting matches |
| `fallthrough.rollout` | Percentage range `[0, 1]` representing 0–100% |
| `exptIncludeAllTargets` | Whether to include all targets in experiments |
| `isArchived` | Whether the flag is archived (false on creation) |

### Notes

- **Revision Tracking**: Each flag has a unique revision GUID for change tracking
- **Default Rollout**: By default, 100% of users receive the enabled variation (`[0, 1]` = 0%–100%)
- **Empty Targeting**: New flags start with no user targeting or rules
- **Audit Trail**: `createdBy` and `updatedBy` are populated from the authenticated user
- **Key Pattern**: Flag keys are case-sensitive and support alphanumeric characters, dots, underscores, and hyphens

### cURL Example

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/envs/{envId}/feature-flags" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{
    "name": "Enable New Checkout Flow",
    "key": "new-checkout-flow",
    "isEnabled": false,
    "description": "Enables the redesigned checkout experience",
    "variationType": "boolean",
    "variations": [
      {"id": "1a2b3c4d-5e6f-7890-abcd-ef1234567890", "name": "Enabled", "value": "true"},
      {"id": "9f8e7d6c-5b4a-3210-fedc-ba9876543210", "name": "Disabled", "value": "false"}
    ],
    "enabledVariationId": "1a2b3c4d-5e6f-7890-abcd-ef1234567890",
    "disabledVariationId": "9f8e7d6c-5b4a-3210-fedc-ba9876543210",
    "tags": ["checkout", "ui-redesign"]
  }'
```

---

## Variation Types Reference

| Type | Description | Example Values |
|------|-------------|----------------|
| `boolean` | True/false flags | `"true"`, `"false"` |
| `string` | Text values | `"value1"`, `"premium"`, `"beta"` |
| `number` | Numeric values | `"42"`, `"3.14"`, `"0"` |
| `json` | Complex JSON objects | `"{\"key\":\"value\"}"` |

**Note**: All variation values are stored as strings regardless of `variationType`. The SDK interprets them based on the type.

---

## Source Code References

- **Controller**: `modules/back-end/src/Api/Controllers/FeatureFlagController.cs`
- **Application**: `modules/back-end/src/Application/FeatureFlags/CreateFeatureFlag.cs`
- **Domain Model**: `modules/back-end/src/Domain/FeatureFlags/FeatureFlag.cs`
