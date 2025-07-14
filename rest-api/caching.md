# Caching in REST API

*Context caching with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/caching
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Context caching, reduced costs, faster responses
-->

## Quick Reference
- **Create**: `POST /v1beta/cachedContents`
- **Use**: `POST /v1beta/models/{model}:generateContent`
- **TTL**: 5 minutes to 1 hour
- **Min tokens**: 32,768 tokens cached content

## Create Cache

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/cachedContents" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "models/gemini-2.5-flash",
    "contents": [{
      "parts": [{
        "text": "Long document content here that exceeds 32K tokens..."
      }]
    }],
    "ttl": "1800s"
  }'
```

## Response Format

```json
{
  "name": "cachedContents/abc123",
  "model": "models/gemini-2.5-flash",
  "createTime": "2025-01-14T10:30:00Z",
  "updateTime": "2025-01-14T10:30:00Z",
  "expireTime": "2025-01-14T11:00:00Z",
  "usageMetadata": {
    "totalTokenCount": 45000
  }
}
```

## Use Cached Content

```bash
# Extract cache name from create response
CACHE_NAME="cachedContents/abc123"

curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Based on the document, what is the main topic?"
      }]
    }],
    "cachedContent": "'"$CACHE_NAME"'"
  }'
```

## Complete Caching Script

```bash
#!/bin/bash
# document_cache.sh

DOCUMENT_FILE="$1"
QUERY="$2"

# Read document content
CONTENT=$(cat "$DOCUMENT_FILE")

# Create cache
echo "Creating cache..."
CACHE_RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/cachedContents" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "models/gemini-2.5-flash",
    "contents": [{
      "parts": [{
        "text": "'"$CONTENT"'"
      }]
    }],
    "ttl": "3600s"
  }')

# Extract cache name
CACHE_NAME=$(echo "$CACHE_RESPONSE" | jq -r '.name')

if [ "$CACHE_NAME" = "null" ]; then
  echo "Cache creation failed:"
  echo "$CACHE_RESPONSE" | jq '.error'
  exit 1
fi

echo "Cache created: $CACHE_NAME"

# Use cache for query
echo "Querying cached content..."
RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$QUERY"'"
      }]
    }],
    "cachedContent": "'"$CACHE_NAME"'"
  }')

# Extract and display response
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'

# Clean up cache
echo "Deleting cache..."
curl -s -X DELETE \
  "https://generativelanguage.googleapis.com/v1beta/$CACHE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"

echo "Cache deleted"
```

## List Caches

```bash
curl -X GET \
  "https://generativelanguage.googleapis.com/v1beta/cachedContents" \
  -H "x-goog-api-key: $GEMINI_API_KEY" | jq '.cachedContents[]'
```

## Get Cache Details

```bash
CACHE_NAME="cachedContents/abc123"

curl -X GET \
  "https://generativelanguage.googleapis.com/v1beta/$CACHE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Update Cache TTL

```bash
curl -X PATCH \
  "https://generativelanguage.googleapis.com/v1beta/$CACHE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ttl": "7200s"
  }'
```

## Delete Cache

```bash
curl -X DELETE \
  "https://generativelanguage.googleapis.com/v1beta/$CACHE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Python Implementation

```python
import requests
import json
import time

class CacheManager:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def create_cache(self, content, model="gemini-2.5-flash", ttl_seconds=3600):
        """Create a new cache"""
        url = f"{self.base_url}/cachedContents"
        
        data = {
            "model": f"models/{model}",
            "contents": [{
                "parts": [{
                    "text": content
                }]
            }],
            "ttl": f"{ttl_seconds}s"
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        return response.json()
    
    def use_cache(self, cache_name, query, model="gemini-2.5-flash"):
        """Use cached content for generation"""
        url = f"{self.base_url}/models/{model}:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": query
                }]
            }],
            "cachedContent": cache_name
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def list_caches(self):
        """List all caches"""
        url = f"{self.base_url}/cachedContents"
        response = requests.get(url, headers=self.headers)
        return response.json().get("cachedContents", [])
    
    def delete_cache(self, cache_name):
        """Delete a cache"""
        url = f"{self.base_url}/{cache_name}"
        response = requests.delete(url, headers=self.headers)
        return response.status_code == 200

# Usage
cache_manager = CacheManager(os.environ["GEMINI_API_KEY"])

# Create cache
cache = cache_manager.create_cache(
    content="Long document content...",
    ttl_seconds=1800
)

cache_name = cache["name"]
print(f"Cache created: {cache_name}")

# Use cache
result = cache_manager.use_cache(cache_name, "What is the main topic?")
print(result)

# Clean up
cache_manager.delete_cache(cache_name)
```

## Multi-Query Example

```bash
#!/bin/bash
# multi_query_cache.sh

DOCUMENT_FILE="$1"
shift
QUERIES=("$@")

# Create cache
CONTENT=$(cat "$DOCUMENT_FILE")
CACHE_RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/cachedContents" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "models/gemini-2.5-flash",
    "contents": [{
      "parts": [{
        "text": "'"$CONTENT"'"
      }]
    }],
    "ttl": "3600s"
  }')

CACHE_NAME=$(echo "$CACHE_RESPONSE" | jq -r '.name')

# Process multiple queries
for query in "${QUERIES[@]}"; do
  echo "Q: $query"
  
  RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$query"'"
        }]
      }],
      "cachedContent": "'"$CACHE_NAME"'"
    }')
  
  echo "A: $(echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text')"
  echo ""
done

# Clean up
curl -s -X DELETE \
  "https://generativelanguage.googleapis.com/v1beta/$CACHE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Streaming with Cache

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Summarize the document"
      }]
    }],
    "cachedContent": "'"$CACHE_NAME"'"
  }'
```

## PowerShell Implementation

```powershell
function New-GeminiCache {
    param(
        [string]$Content,
        [string]$ApiKey,
        [string]$Model = "gemini-2.5-flash",
        [int]$TTLSeconds = 3600
    )
    
    $Headers = @{
        "x-goog-api-key" = $ApiKey
        "Content-Type" = "application/json"
    }
    
    $Body = @{
        model = "models/$Model"
        contents = @(
            @{
                parts = @(
                    @{
                        text = $Content
                    }
                )
            }
        )
        ttl = "${TTLSeconds}s"
    } | ConvertTo-Json -Depth 4
    
    $Response = Invoke-RestMethod -Uri "https://generativelanguage.googleapis.com/v1beta/cachedContents" -Method POST -Headers $Headers -Body $Body
    return $Response
}

function Use-GeminiCache {
    param(
        [string]$CacheName,
        [string]$Query,
        [string]$ApiKey,
        [string]$Model = "gemini-2.5-flash"
    )
    
    $Headers = @{
        "x-goog-api-key" = $ApiKey
        "Content-Type" = "application/json"
    }
    
    $Body = @{
        contents = @(
            @{
                parts = @(
                    @{
                        text = $Query
                    }
                )
            }
        )
        cachedContent = $CacheName
    } | ConvertTo-Json -Depth 4
    
    $Response = Invoke-RestMethod -Uri "https://generativelanguage.googleapis.com/v1beta/models/$Model`:generateContent" -Method POST -Headers $Headers -Body $Body
    return $Response.candidates[0].content.parts[0].text
}
```

## Error Handling

```bash
# Check cache creation
if echo "$CACHE_RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$CACHE_RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"minimum token count"* ]]; then
        echo "Content too short for caching (need 32K+ tokens)"
    elif [[ "$ERROR_MSG" == *"maximum token count"* ]]; then
        echo "Content too long for caching"
    elif [[ "$ERROR_MSG" == *"TTL"* ]]; then
        echo "Invalid TTL (must be 5min-1hour)"
    else
        echo "Error: $ERROR_MSG"
    fi
    exit 1
fi
```

## Best Practices

1. **Token Minimum**: Only cache content >32K tokens
2. **TTL Management**: Set appropriate cache duration
3. **Cost Monitoring**: Track cache usage and savings
4. **Cleanup**: Delete caches when done
5. **Multi-Query**: Use cache for multiple questions
6. **Model Consistency**: Use same model for cache and queries

## See Also
- [Authentication](authentication.md) - API key setup
- [Error Handling](error-handling.md) - Error management
- [Rate Limits](rate-limits.md) - Usage limits