# REST API Quickstart

*Direct HTTP requests to Gemini API using cURL*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/quickstart?lang=rest
Verified: 2025-01-14
Key Info: Use x-goog-api-key header (lowercase), v1beta endpoint
Note: JSON structure same across all language SDKs
-->

## Quick Reference
- **Base URL**: `https://generativelanguage.googleapis.com/v1beta`
- **Auth**: API key in header or URL param
- **Format**: JSON request/response
- **Methods**: POST for all generation

## First Request

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Explain quantum computing in one paragraph"
      }]
    }]
  }'
```

## Authentication Options

```bash
# Option 1: API key in URL
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$GEMINI_API_KEY"

# Option 2: API key in header (recommended)
curl -H "x-goog-api-key: $GEMINI_API_KEY" \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"

# Option 3: Bearer token (OAuth)
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"
```

## Request Structure

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "text": "Your prompt here"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.9,
    "topK": 1,
    "topP": 1,
    "maxOutputTokens": 2048
  }
}
```

## Response Structure

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Generated response text"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0,
      "safetyRatings": [...]
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 10,
    "candidatesTokenCount": 150,
    "totalTokenCount": 160
  }
}
```

## Chat Conversation

```bash
# Multi-turn conversation
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      {
        "role": "user",
        "parts": [{"text": "Hello, my name is Alice"}]
      },
      {
        "role": "model",
        "parts": [{"text": "Hello Alice! Nice to meet you."}]
      },
      {
        "role": "user",
        "parts": [{"text": "What did I just tell you my name was?"}]
      }
    ]
  }'
```

## Generation Parameters

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Write a creative story"}]
    }],
    "generationConfig": {
      "temperature": 1.0,
      "topK": 40,
      "topP": 0.95,
      "maxOutputTokens": 1000,
      "stopSequences": ["THE END"]
    }
  }'
```

## System Instruction

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "systemInstruction": {
      "parts": [{
        "text": "You are a helpful coding assistant. Always provide code examples."
      }]
    },
    "contents": [{
      "parts": [{"text": "How do I read a file in Python?"}]
    }]
  }'
```

## Error Response

```json
{
  "error": {
    "code": 400,
    "message": "Invalid request",
    "status": "INVALID_ARGUMENT",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.BadRequest",
        "fieldViolations": [
          {
            "field": "contents",
            "description": "Required field"
          }
        ]
      }
    ]
  }
}
```

## Complete Script Example

```bash
#!/bin/bash

# Set your API key
export GEMINI_API_KEY="your-api-key-here"

# Function to call Gemini
gemini_generate() {
  local prompt="$1"
  local response=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [{
          \"text\": \"$prompt\"
        }]
      }]
    }")
  
  # Extract text using jq
  echo "$response" | jq -r '.candidates[0].content.parts[0].text'
}

# Usage
result=$(gemini_generate "What is the capital of France?")
echo "$result"
```

## Using with Different Tools

### Python requests
```python
import requests
import os

api_key = os.environ.get('GEMINI_API_KEY')
url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"

headers = {
    "x-goog-api-key": api_key,
    "Content-Type": "application/json"
}

payload = {
    "contents": [{
        "parts": [{
            "text": "Hello, Gemini!"
        }]
    }]
}

response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

### JavaScript fetch
```javascript
const response = await fetch(
  'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent',
  {
    method: 'POST',
    headers: {
      'x-goog-api-key': process.env.GEMINI_API_KEY,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      contents: [{
        parts: [{
          text: "Hello, Gemini!"
        }]
      }]
    })
  }
);

const data = await response.json();
console.log(data.candidates[0].content.parts[0].text);
```

## Common Headers

```bash
# Required
Content-Type: application/json

# Authentication (choose one)
x-goog-api-key: YOUR_API_KEY
Authorization: Bearer ACCESS_TOKEN

# Optional
X-Goog-User-Project: project-id
X-Goog-Api-Version: v1beta
```

## Next Steps
- [Endpoints](endpoints.md) - All available endpoints
- [Authentication](authentication.md) - Auth methods
- [Text Generation](text-generation.md) - Generation examples

## See Also
- [Error Responses](error-responses.md)
- [cURL Examples](../examples/rest/curl-scripts.md)