---
name: projects-api
description: REST API endpoints for managing FeatBit projects (create, list, update, delete).
---

# Projects API

APIs for managing FeatBit projects. Projects are top-level containers for organizing feature flags across different applications.

## Table of Contents

1. [Common Response Shape](#common-response-shape)
2. [ProjectWithEnvs Schema](#projectwithenvs-schema)
3. [Create Project](#create-project)
4. [Get a Project](#get-a-project)
5. [Get Project List](#get-project-list)
6. [Source Code References](#source-code-references)

---

## Common Response Shape

All endpoints return the standard envelope:

```json
{ "success": true, "data": <payload>, "errors": [] }
```

On error: `success: false`, `data: null`, `errors` contains `{ code, message }` objects.

---

## ProjectWithEnvs Schema

Every project response uses this shape (`data` for single, `data[]` for list):

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
| `environments[].secrets[].name` | string | Secret name ("Server Key" or "Client Key") |
| `environments[].secrets[].type` | string | `Server` or `Client` |
| `environments[].secrets[].value` | string | Secret value |
| `environments[].settings` | array | Environment settings |
| `environments[].createdAt` | datetime | Creation timestamp (UTC) |
| `environments[].updatedAt` | datetime | Last update timestamp (UTC) |

**Example environment object** (referenced in endpoint examples below):

```json
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
```

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
{ "name": "E-Commerce Platform", "key": "ecommerce" }
```

### Success Response (200 OK)

Returns a `ProjectWithEnvs` object. Two default environments ("Prod" and "Dev") are auto-created, each with a Server Key and Client Key.

```json
{
  "success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "E-Commerce Platform",
    "key": "ecommerce",
    "environments": [ /* Prod env, Dev env — see schema above */ ]
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
- **Organization Context**: Organization ID is extracted from the authenticated user's context

### cURL Example

```bash
curl -X POST "https://your-featbit-instance.com/api/v1/projects" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {jwt_token}" \
  -d '{"name": "E-Commerce Platform", "key": "ecommerce"}'
```

---

## Get a Project

Retrieve detailed information about a specific project, including all its environments and secrets.

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

Returns a single `ProjectWithEnvs` object (see [schema](#projectwithenvs-schema)).

```json
{
  "success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "E-Commerce Platform",
    "key": "ecommerce",
    "environments": [ /* see schema above */ ]
  },
  "errors": []
}
```

### Error Response (404 Not Found)

```json
{
  "success": false,
  "data": null,
  "errors": [{ "code": "NotFound", "message": "Project not found" }]
}
```

### cURL Example

```bash
curl -X GET "https://your-featbit-instance.com/api/v1/projects/{projectId}" \
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

None. Organization ID is extracted from the authenticated user's context.

### Success Response (200 OK)

Returns `ProjectWithEnvs[]` — an array of projects (see [schema](#projectwithenvs-schema)).

```json
{
  "success": true,
  "data": [
    { "id": "...", "name": "E-Commerce Platform", "key": "ecommerce", "environments": [ /* ... */ ] },
    { "id": "...", "name": "Mobile App", "key": "mobile-app", "environments": [ /* ... */ ] }
  ],
  "errors": []
}
```

Empty org returns `{ "success": true, "data": [], "errors": [] }`.

### Notes

- **No Pagination**: Returns all projects — no pagination parameters
- **Includes Environments**: Each project includes complete environment info with secrets

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
