# REST API Endpoints

*Complete endpoint reference for Gemini API*

<!-- METADATA
Source: https://ai.google.dev/api/rest
Verified: 2025-01-14
Key Info: v1beta current version, regional endpoints available
Note: Model names updated to latest versions
-->

## Quick Reference
- **Base URL**: `https://generativelanguage.googleapis.com`
- **Version**: `/v1beta` (current)
- **Format**: `/{version}/{resource}:{method}`
- **Auth**: Required for all endpoints

## Base URLs by Region

```bash
# Global (auto-routing)
https://generativelanguage.googleapis.com

# United States
https://us-central1-generativelanguage.googleapis.com

# Europe
https://europe-west4-generativelanguage.googleapis.com

# Asia Pacific
https://asia-northeast1-generativelanguage.googleapis.com
```

## Core Endpoints

### Generate Content
```bash
POST /v1beta/models/{model}:generateContent

# Example
POST /v1beta/models/gemini-2.5-flash:generateContent
```

### Stream Generate Content
```bash
POST /v1beta/models/{model}:streamGenerateContent

# Server-sent events stream
POST /v1beta/models/gemini-2.5-flash:streamGenerateContent
```

### Count Tokens
```bash
POST /v1beta/models/{model}:countTokens

# Count tokens without generating
POST /v1beta/models/gemini-2.5-flash:countTokens
```

## Model Management

### List Models
```bash
GET /v1beta/models

# Response includes all available models
curl "https://generativelanguage.googleapis.com/v1beta/models?key=$API_KEY"
```

### Get Model
```bash
GET /v1beta/models/{model}

# Get specific model details
GET /v1beta/models/gemini-2.5-flash
```

## Batch Processing

### Create Batch Job
```bash
POST /v1beta/batchPredictionJobs

# Submit batch of requests
{
  "requests": [
    {"model": "models/gemini-2.5-flash", "contents": [...]},
    {"model": "models/gemini-2.5-flash", "contents": [...]}
  ]
}
```

### Get Batch Job
```bash
GET /v1beta/batchPredictionJobs/{jobId}

# Check batch status
GET /v1beta/batchPredictionJobs/job-123456
```

## File Management

### Upload File
```bash
POST /v1beta/files

# Multipart upload
curl -X POST "https://generativelanguage.googleapis.com/v1beta/files?key=$API_KEY" \
  -F "file=@image.jpg"
```

### List Files
```bash
GET /v1beta/files

# List uploaded files
GET /v1beta/files?pageSize=10
```

### Get File
```bash
GET /v1beta/files/{fileId}

# Get file metadata
GET /v1beta/files/file-abc123
```

### Delete File
```bash
DELETE /v1beta/files/{fileId}

# Remove uploaded file
DELETE /v1beta/files/file-abc123
```

## Embeddings

### Embed Content
```bash
POST /v1beta/models/{model}:embedContent

# Generate embeddings
POST /v1beta/models/text-embedding-004:embedContent
{
  "content": {
    "parts": [{"text": "Your text here"}]
  }
}
```

### Batch Embed
```bash
POST /v1beta/models/{model}:batchEmbedContents

# Multiple embeddings
POST /v1beta/models/text-embedding-004:batchEmbedContents
{
  "requests": [
    {"content": {"parts": [{"text": "Text 1"}]}},
    {"content": {"parts": [{"text": "Text 2"}]}}
  ]
}
```

## Caching

### Create Cached Content
```bash
POST /v1beta/cachedContents

# Cache large context
{
  "model": "models/gemini-1.5-flash",
  "contents": [...],
  "ttl": "3600s"
}
```

### List Cached Content
```bash
GET /v1beta/cachedContents

# View cached items
GET /v1beta/cachedContents?pageSize=10
```

### Delete Cached Content
```bash
DELETE /v1beta/cachedContents/{cacheId}

# Remove cache
DELETE /v1beta/cachedContents/cache-xyz789
```

## Tuned Models

### Create Tuning Job
```bash
POST /v1beta/tuningJobs

# Start fine-tuning
{
  "baseModel": "models/gemini-1.5-flash",
  "trainingData": {...}
}
```

### List Tuning Jobs
```bash
GET /v1beta/tuningJobs

# Check tuning status
GET /v1beta/tuningJobs?filter=state=ACTIVE
```

## Operations

### Get Operation
```bash
GET /v1beta/operations/{operationId}

# Check long-running operation
GET /v1beta/operations/operation-123
```

### List Operations
```bash
GET /v1beta/operations

# List all operations
GET /v1beta/operations?filter=done=false
```

### Cancel Operation
```bash
POST /v1beta/operations/{operationId}:cancel

# Cancel running operation
POST /v1beta/operations/operation-123:cancel
```

## Response Formats

### Successful Response
```json
{
  "candidates": [...],
  "usageMetadata": {...},
  "modelVersion": "gemini-2.5-flash"
}
```

### Error Response
```json
{
  "error": {
    "code": 400,
    "message": "Invalid request",
    "status": "INVALID_ARGUMENT"
  }
}
```

### Streaming Response
```
data: {"candidates":[{"content":{"parts":[{"text":"Hello"}]}}]}
data: {"candidates":[{"content":{"parts":[{"text":" world"}]}}]}
data: [DONE]
```

## Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `key` | API key | `key=AIza...` |
| `alt` | Response format | `alt=sse` |
| `pageSize` | Items per page | `pageSize=50` |
| `pageToken` | Pagination token | `pageToken=abc123` |
| `filter` | Resource filter | `filter=state=ACTIVE` |

## See Also
- [Authentication](authentication.md)
- [Request Format](request-format.md)
- [Error Responses](error-responses.md)