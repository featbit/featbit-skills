# Environments API

APIs for managing FeatBit environments. Environments allow you to maintain separate configurations for different deployment stages (e.g., development, staging, production).

---

## Create an Environment

Create a new environment within a project.

### Endpoint

```http
POST /api/v1/projects/{projectId}/envs
```

### Authorization

- **Permission**: `CreateEnv`
- **Scope**: Project level

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `projectId` | guid | Yes | The project ID where the environment will be created |

### Request Headers

```http
Content-Type: application/json
Authorization: Bearer {jwt_token}
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Display name of the environment |
| `key` | string | Yes | Unique identifier within the project |
| `description` | string | No | Optional description of the environment |

### Example Request

```json
{
  "name": "Staging",
  "key": "staging",
  "description": "Staging environment for QA testing"
}
```

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "7c6d5e4f-3210-9876-fedc-ba9876543210",
    "name": "Staging",
    "description": "Staging environment for QA testing",
    "secrets": [
      {
        "id": "secret-guid-5",
        "name": "Server Key",
        "type": "Server",
        "value": "StUvWx678901-7c6d5e4f321098"
      },
      {
        "id": "secret-guid-6",
        "name": "Client Key",
        "type": "Client",
        "value": "YzAbCd234567-7c6d5e4f321098"
      }
    ]
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
    { "code": "KeyHasBeenUsed", "message": "The key has already been used in this project" }
  ]
}
```

### Validation Rules

- `name`: Cannot be empty
- `key`: Cannot be empty, must be unique within the project
- `description`: Optional, can be empty string

### Notes

- **Auto-generated Secrets**: Creates both Server Key and Client Key automatically
- **Secret Format**: Secrets are generated in the format `{header}{body}` where both parts are encoded GUIDs
- **Built-in Properties**: Environment is automatically initialized with built-in user properties for targeting

### cURL Example

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/projects/3fa85f64-5717-4562-b3fc-2c963f66afa6/envs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{
    "name": "Staging",
    "key": "staging",
    "description": "Staging environment for QA testing"
  }'
```

---

## Source Code References

- **Controller**: `modules/back-end/src/Api/Controllers/EnvironmentController.cs`
- **Application**: `modules/back-end/src/Application/Environments/CreateEnvironment.cs`
- **Domain Model**: `modules/back-end/src/Domain/Environments/Environment.cs`
