# Projects API

APIs for managing FeatBit projects. Projects are top-level containers for organizing feature flags across different applications.

---

## Create Project

Create a new project within your organization.

### Endpoint

```http
POST /api/v1/projects
```

### Authorization

- **Permission**: `CreateProject`
- **Scope**: Organization level

### Request Headers

```http
Content-Type: application/json
Authorization: Bearer {jwt_token}
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Display name of the project |
| `key` | string | Yes | Unique identifier (alphanumeric, dots, underscores, hyphens) |

### Example Request

```json
{
  "name": "E-Commerce Platform",
  "key": "ecommerce"
}
```

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "E-Commerce Platform",
    "key": "ecommerce",
    "environments": [
      {
        "id": "8d7e9f12-3456-7890-abcd-ef1234567890",
        "projectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "name": "Prod",
        "key": "prod",
        "description": "",
        "secrets": [
          {
            "id": "secret-guid-1",
            "name": "Server Key",
            "type": "Server",
            "value": "AbCdEf123456-8d7e9f12345678"
          },
          {
            "id": "secret-guid-2",
            "name": "Client Key",
            "type": "Client",
            "value": "XyZaBc789012-8d7e9f12345678"
          }
        ],
        "settings": [],
        "createdAt": "2026-02-09T10:30:00Z",
        "updatedAt": "2026-02-09T10:30:00Z"
      },
      {
        "id": "9e8f0a23-4567-8901-bcde-f12345678901",
        "projectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "name": "Dev",
        "key": "dev",
        "description": "",
        "secrets": [
          {
            "id": "secret-guid-3",
            "name": "Server Key",
            "type": "Server",
            "value": "GhIjKl456789-9e8f0a23456789"
          },
          {
            "id": "secret-guid-4",
            "name": "Client Key",
            "type": "Client",
            "value": "MnOpQr012345-9e8f0a23456789"
          }
        ],
        "settings": [],
        "createdAt": "2026-02-09T10:30:00Z",
        "updatedAt": "2026-02-09T10:30:00Z"
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
    { "code": "Required:name", "message": "The name field is required" },
    { "code": "KeyHasBeenUsed", "message": "The key has already been used" }
  ]
}
```

### Validation Rules

- `name`: Cannot be empty
- `key`: Cannot be empty, must be unique within the organization

### Notes

- **Auto-generated Environments**: Creates two default environments: "Prod" and "Dev"
- **Auto-generated Secrets**: Each environment automatically gets a Server Key and Client Key
- **Organization Context**: The organization ID is automatically extracted from the authenticated user's context

### cURL Example

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{
    "name": "E-Commerce Platform",
    "key": "ecommerce"
  }'
```

---

## Get a Project

Retrieve detailed information about a specific project, including all its environments and associated secrets.

### Endpoint

```http
GET /api/v1/projects/{projectId}
```

### Authorization

- **Permission**: `CanAccessProject`
- **Scope**: Project level

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `projectId` | guid | Yes | The unique identifier of the project |

### Request Headers

```http
Authorization: Bearer {jwt_token}
```

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "E-Commerce Platform",
    "key": "ecommerce",
    "environments": [
      {
        "id": "8d7e9f12-3456-7890-abcd-ef1234567890",
        "projectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "name": "Prod",
        "key": "prod",
        "description": "Production environment",
        "secrets": [
          {
            "id": "secret-guid-1",
            "name": "Server Key",
            "type": "Server",
            "value": "AbCdEf123456-8d7e9f12345678"
          },
          {
            "id": "secret-guid-2",
            "name": "Client Key",
            "type": "Client",
            "value": "XyZaBc789012-8d7e9f12345678"
          }
        ],
        "settings": [],
        "createdAt": "2026-02-09T10:30:00Z",
        "updatedAt": "2026-02-09T10:30:00Z"
      }
    ]
  },
  "errors": []
}
```

### Error Response (404 Not Found)

```json
{
  "success": false,
  "data": null,
  "errors": [
    { "code": "NotFound", "message": "Project not found" }
  ]
}
```

### cURL Example

```bash
curl -X GET "https://your-featbit-instance.com/api/v1/projects/3fa85f64-5717-4562-b3fc-2c963f66afa6" \
  -H "Authorization: Bearer {jwt_token}"
```

---

## Source Code References

- **Controller**: `modules/back-end/src/Api/Controllers/ProjectController.cs`
- **Application**: `modules/back-end/src/Application/Projects/CreateProject.cs`
- **Domain Model**: `modules/back-end/src/Domain/Projects/ProjectWithEnvs.cs`
