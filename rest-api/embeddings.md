# Embeddings in REST API

*Generate text embeddings with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/embeddings
API Version: v1beta
Verified: 2025-01-14
Models: text-embedding-004
Key Features: 768-dimensional embeddings, semantic search
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/text-embedding-004:embedContent`
- **Method**: POST
- **Dimensions**: 768
- **Max input**: 2048 tokens

## Basic Embedding Request

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "The quick brown fox jumps over the lazy dog"
        }
      ]
    }
  }'
```

## Response Format

```json
{
  "embedding": {
    "values": [
      0.123456,
      -0.234567,
      0.345678,
      ...
    ]
  }
}
```

## Batch Embeddings Script

```bash
#!/bin/bash
# generate_embeddings.sh

TEXTS=(
  "Machine learning is a subset of AI"
  "Deep learning uses neural networks"
  "Natural language processing handles text"
)

for text in "${TEXTS[@]}"; do
  echo "Processing: $text"
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "content": {
        "parts": [
          {
            "text": "'"$text"'"
          }
        ]
      }
    }' | jq '.embedding.values' > "embedding_$(echo "$text" | tr ' ' '_').json"
done
```

## Python Implementation

```python
import requests
import json
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

class EmbeddingClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
    
    def embed_text(self, text):
        """Generate embedding for single text"""
        url = f"{self.base_url}/models/text-embedding-004:embedContent"
        
        headers = {
            "x-goog-api-key": self.api_key,
            "Content-Type": "application/json"
        }
        
        data = {
            "content": {
                "parts": [
                    {"text": text}
                ]
            }
        }
        
        response = requests.post(url, headers=headers, json=data)
        result = response.json()
        
        return result["embedding"]["values"]
    
    def embed_batch(self, texts):
        """Generate embeddings for multiple texts"""
        embeddings = []
        
        for text in texts:
            embedding = self.embed_text(text)
            embeddings.append(embedding)
        
        return embeddings
    
    def semantic_search(self, query, documents):
        """Perform semantic search"""
        # Generate embeddings
        query_embedding = self.embed_text(query)
        doc_embeddings = self.embed_batch(documents)
        
        # Calculate similarities
        query_array = np.array(query_embedding).reshape(1, -1)
        doc_array = np.array(doc_embeddings)
        
        similarities = cosine_similarity(query_array, doc_array)[0]
        
        # Sort by similarity
        results = []
        for i, doc in enumerate(documents):
            results.append({
                "document": doc,
                "score": similarities[i]
            })
        
        results.sort(key=lambda x: x["score"], reverse=True)
        return results

# Usage
client = EmbeddingClient(os.environ["GEMINI_API_KEY"])

documents = [
    "Python is great for data science",
    "JavaScript excels at web development",
    "Machine learning requires good data"
]

results = client.semantic_search("AI programming languages", documents)
for result in results:
    print(f"{result['score']:.3f}: {result['document']}")
```

## JavaScript Implementation

```javascript
class EmbeddingClient {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async embedText(text) {
        const url = `${this.baseUrl}/models/text-embedding-004:embedContent`;
        
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'x-goog-api-key': this.apiKey,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                content: {
                    parts: [
                        { text: text }
                    ]
                }
            })
        });
        
        const result = await response.json();
        return result.embedding.values;
    }
    
    async embedBatch(texts) {
        const embeddings = [];
        
        for (const text of texts) {
            const embedding = await this.embedText(text);
            embeddings.push(embedding);
        }
        
        return embeddings;
    }
    
    cosineSimilarity(a, b) {
        const dotProduct = a.reduce((sum, val, i) => sum + val * b[i], 0);
        const normA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
        const normB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
        return dotProduct / (normA * normB);
    }
    
    async semanticSearch(query, documents) {
        const queryEmbedding = await this.embedText(query);
        const docEmbeddings = await this.embedBatch(documents);
        
        const results = documents.map((doc, i) => ({
            document: doc,
            score: this.cosineSimilarity(queryEmbedding, docEmbeddings[i])
        }));
        
        return results.sort((a, b) => b.score - a.score);
    }
}
```

## cURL Examples

### Single Text
```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "Your text here"
        }
      ]
    }
  }' | jq '.embedding.values'
```

### With Task Type
```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "Search query example"
        }
      ]
    },
    "taskType": "RETRIEVAL_QUERY"
  }'
```

## Task Types

```json
{
  "taskType": "RETRIEVAL_QUERY",
  "content": {
    "parts": [
      {
        "text": "What is machine learning?"
      }
    ]
  }
}
```

Available task types:
- `RETRIEVAL_QUERY`: Search queries
- `RETRIEVAL_DOCUMENT`: Documents to search
- `SEMANTIC_SIMILARITY`: General similarity
- `CLASSIFICATION`: Text classification
- `CLUSTERING`: Document clustering

## Document Similarity

```bash
#!/bin/bash
# similarity.sh

TEXT1="$1"
TEXT2="$2"

# Get embeddings
EMB1=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "'"$TEXT1"'"
        }
      ]
    }
  }' | jq '.embedding.values')

EMB2=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "'"$TEXT2"'"
        }
      ]
    }
  }' | jq '.embedding.values')

# Calculate cosine similarity (requires additional processing)
echo "Embeddings generated. Use Python/JavaScript for similarity calculation."
```

## Error Handling

```bash
# Check for errors in response
RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": {
      "parts": [
        {
          "text": "'"$TEXT"'"
        }
      ]
    }
  }')

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    echo "Error: $ERROR_MSG"
    exit 1
fi

# Extract embedding
EMBEDDING=$(echo "$RESPONSE" | jq '.embedding.values')
```

## Rate Limiting

```bash
# Add delays for rate limiting
for text in "${TEXTS[@]}"; do
    # Process text
    curl -s ... 
    
    # Wait between requests
    sleep 1
done
```

## PowerShell Example

```powershell
# PowerShell implementation
$ApiKey = $env:GEMINI_API_KEY
$Url = "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent"

$Body = @{
    content = @{
        parts = @(
            @{
                text = "Your text here"
            }
        )
    }
} | ConvertTo-Json -Depth 3

$Headers = @{
    "x-goog-api-key" = $ApiKey
    "Content-Type" = "application/json"
}

$Response = Invoke-RestMethod -Uri $Url -Method POST -Body $Body -Headers $Headers
$Response.embedding.values
```

## Best Practices

1. **Rate Limiting**: Respect API limits
2. **Caching**: Store embeddings to avoid regeneration
3. **Batch Processing**: Process multiple texts efficiently
4. **Error Handling**: Check for API errors
5. **Token Limits**: Stay under 2048 tokens per request

## See Also
- [Authentication](authentication.md) - API key setup
- [Rate Limits](rate-limits.md) - Usage limits
- [Error Codes](error-codes.md) - Error handling