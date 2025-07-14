# Authentication

*API keys, OAuth, and access control for Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/authentication
Verified: 2025-01-14
Key Info: API keys for most use cases, OAuth for user data
Note: ADC recommended for Google Cloud deployments
-->

## Quick Reference
- **API Key**: Simplest method for personal projects
- **OAuth**: Required for user-specific data access
- **ADC**: Best for Google Cloud environments

## API Key Authentication

### Getting an API Key
1. Visit [Google AI Studio](https://aistudio.google.com)
2. Click "Get API key"
3. Create new or select existing project
4. Copy generated key

### Usage
```bash
# Environment variable (recommended)
export GEMINI_API_KEY="your-api-key"

# In request header (lowercase)
x-goog-api-key: YOUR_API_KEY

# URL parameter (not recommended)
?key=YOUR_API_KEY
```

### Security Best Practices
- Never commit keys to version control
- Use environment variables
- Rotate keys regularly
- Set API restrictions in console
- Use separate keys per environment

## OAuth 2.0

### When to Use
- Accessing user's Google Drive files
- Google Workspace integration
- Multi-user applications

### Scopes
```
https://www.googleapis.com/auth/generative-language
https://www.googleapis.com/auth/cloud-platform
```

### Flow Example
```python
from google.auth.transport.requests import Request
from google_auth_oauthlib.flow import Flow

flow = Flow.from_client_secrets_file(
    'credentials.json',
    scopes=['https://www.googleapis.com/auth/generative-language']
)
```

## Application Default Credentials (ADC)

### Setup
```bash
# Local development
gcloud auth application-default login

# Service account
export GOOGLE_APPLICATION_CREDENTIALS="path/to/service-account.json"
```

### Best For
- Google Cloud Run
- Cloud Functions
- GKE deployments
- CI/CD pipelines

## API Key Restrictions

### IP Restrictions
- Whitelist specific IPs
- Use CIDR notation for ranges

### HTTP Referrer
- Restrict to specific domains
- Use wildcards: `*.example.com/*`

### Application Restrictions
- Android apps: Package name
- iOS apps: Bundle ID

## Common Headers

```http
# API Key
X-Goog-Api-Key: YOUR_API_KEY

# OAuth Token
Authorization: Bearer ACCESS_TOKEN

# API Version
X-Goog-Api-Version: v1beta
```

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Invalid credentials |
| 403 | Permission denied |
| 429 | Rate limit exceeded |

## Regional Endpoints

- Global: `generativelanguage.googleapis.com`
- US: `us-central1-generativelanguage.googleapis.com`
- EU: `europe-west4-generativelanguage.googleapis.com`

## See Also
- [Rate Limits](rate-limits.md)
- [Error Handling](errors.md)