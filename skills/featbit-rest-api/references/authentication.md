# Authentication

FeatBit supports two methods for authenticating API requests.

## 1. JWT Bearer Token (User Authentication)

Used for interactive sessions and user-scoped operations.

### Header Format

```http
Authorization: Bearer {jwt_token}
```

### How to Obtain

1. Log in to the FeatBit portal
2. The JWT token is issued during login via the authentication endpoint
3. Use it in the `Authorization` header for subsequent API calls

### Characteristics

- Tied to a specific user identity
- Carries user permissions and organization context
- Expires after a configured session duration
- Best for: admin dashboards, interactive tools, user-specific operations

## 2. OpenAPI Key (Machine-to-Machine)

Used for automation, CI/CD pipelines, and service-to-service communication.

### Header Format

```http
Authorization: {api_key}
```

**Note**: No `Bearer` prefix — the API key is sent directly as the `Authorization` header value.

### How to Obtain

1. Navigate to your FeatBit portal
2. Go to **Organization Settings** → **API Keys** (or equivalent section)
3. Generate a new OpenAPI key
4. Copy and store the key securely

### Characteristics

- Not tied to a specific user
- Scoped to the organization
- Does not expire unless revoked
- Best for: CI/CD pipelines, automation scripts, scheduled tasks, service integrations

## Choosing the Right Method

| Criteria | JWT Bearer Token | OpenAPI Key |
|----------|-----------------|-------------|
| **Use case** | Interactive / user sessions | Automation / machine-to-machine |
| **Identity** | User-specific | Organization-scoped |
| **Expiration** | Session-based | Until revoked |
| **Header format** | `Bearer {token}` | `{api_key}` (no prefix) |
| **Best for** | Admin tools, dashboards | CI/CD, scripts, integrations |

## Security Best Practices

1. **Never commit tokens or keys** to version control
2. **Use environment variables** to store credentials:
   ```bash
   export FEATBIT_TOKEN="your-jwt-token"
   export FEATBIT_API_KEY="your-api-key"
   ```
3. **Rotate keys** periodically for OpenAPI keys
4. **Use least privilege** — create keys with only the permissions needed
5. **Audit usage** — monitor API access through FeatBit logs

## Example Usage

### With JWT Bearer Token

```bash
curl -X GET "https://your-featbit-instance.com/api/v1/projects" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

### With OpenAPI Key

```bash
curl -X GET "https://your-featbit-instance.com/api/v1/projects" \
  -H "Authorization: api-abc123def456..."
```
