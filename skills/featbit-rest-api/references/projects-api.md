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

## Get Project List

Retrieve all projects in the current organization, including all environments and their secrets.

### Endpoint

```http
GET /api/v1/projects
```

### Authorization

- **Permission**: `CanAccessProject`
- **Scope**: Organization level

### Request Headers

```http
Authorization: Bearer {jwt_token}
```

### Parameters

None. The organization ID is automatically extracted from the authenticated user's context.

### Success Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
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
            { "id": "secret-guid-1", "name": "Server Key", "type": "Server", "value": "AbCdEf123456-8d7e9f12345678" },
            { "id": "secret-guid-2", "name": "Client Key", "type": "Client", "value": "XyZaBc789012-8d7e9f12345678" }
          ],
          "settings": [],
          "createdAt": "2026-02-09T10:30:00Z",
          "updatedAt": "2026-02-09T10:30:00Z"
        }
      ]
    },
    {
      "id": "b2c3d4e5-f6a7-8901-2345-678901abcdef",
      "name": "Mobile App",
      "key": "mobile-app",
      "environments": [
        {
          "id": "c3d4e5f6-a7b8-9012-3456-789012bcdef0",
          "projectId": "b2c3d4e5-f6a7-8901-2345-678901abcdef",
          "name": "Prod",
          "key": "prod",
          "description": "",
          "secrets": [
            { "id": "secret-guid-5", "name": "Server Key", "type": "Server", "value": "StUvWx678901-c3d4e5f6a7b890" },
            { "id": "secret-guid-6", "name": "Client Key", "type": "Client", "value": "YzAbCd234567-c3d4e5f6a7b890" }
          ],
          "settings": [],
          "createdAt": "2026-01-15T08:00:00Z",
          "updatedAt": "2026-01-15T08:00:00Z"
        }
      ]
    }
  ],
  "errors": []
}
```

### Empty Response (200 OK)

```json
{
  "success": true,
  "data": [],
  "errors": []
}
```

### Response Data Schema

Returns `ProjectWithEnvs[]` — an array of projects with nested environments:

| Field | Type | Description |
|-------|------|-------------|
| `id` | guid | Project unique identifier |
| `name` | string | Project display name |
| `key` | string | Project unique key |
| `environments` | array | All environments in the project |
| `environments[].id` | guid | Environment unique identifier |
| `environments[].projectId` | guid | Parent project ID |
| `environments[].name` | string | Environment display name |
| `environments[].key` | string | Environment unique key |
| `environments[].description` | string | Environment description |
| `environments[].secrets` | array | Environment secrets (Server Key + Client Key) |
| `environments[].secrets[].id` | string | Secret ID |
| `environments[].secrets[].name` | string | Secret name |
| `environments[].secrets[].type` | string | Secret type: `Server` or `Client` |
| `environments[].secrets[].value` | string | Secret value |
| `environments[].settings` | array | Environment settings |
| `environments[].createdAt` | datetime | Creation timestamp (UTC) |
| `environments[].updatedAt` | datetime | Last update timestamp (UTC) |

### Notes

- **No Pagination**: Returns all projects in the organization — no pagination parameters
- **Includes Environments**: Each project includes complete environment info with secrets
- **Organization Context**: Organization ID is extracted from the authenticated user's context automatically

### cURL Example

```bash
curl -X GET "https://your-featbit-instance.com/api/v1/projects" \
  -H "Authorization: Bearer {jwt_token}"
```

---

## Source Code References

- **Controller**: `modules/back-end/src/Api/Controllers/ProjectController.cs`
- **Application**: `modules/back-end/src/Application/Projects/CreateProject.cs`
- **Domain Model**: `modules/back-end/src/Domain/Projects/ProjectWithEnvs.cs`
