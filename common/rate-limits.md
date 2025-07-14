# Rate Limits

*Quotas, limits, and best practices for Gemini API usage*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/quota
Verified: 2025-01-14
Key Info: Free tier available, tier system based on usage
Note: Limits increase automatically with consistent usage
-->

## Quick Reference
- **Free tier**: 2-30 RPM depending on model
- **Tier 1-3**: Progressive increases with usage
- **No daily limits**: Only per-minute limits
- **Token limits**: Separate input/output quotas

## Rate Limits by Tier

### Free Tier
| Model | RPM | TPM |
|-------|-----|-----|
| Gemini 2.5 Pro | 5 | 250K |
| Gemini 2.5 Flash | 10 | 250K |
| Gemini 2.0 Flash | 15 | 1M |
| Gemini 2.0 Flash-Lite | 30 | 1M |
| Gemini 1.5 Pro | 2 | 32K |
| Gemini 1.5 Flash | 15 | 1M |

### Tier 1 (Pay-as-you-go)
| Model | RPM | TPM |
|-------|-----|-----|
| Gemini 2.5 Pro | 150 | 2M |
| Gemini 2.5 Flash | 1,000 | 1M |
| Gemini 2.0 Flash | 2,000 | 4M |
| Gemini 2.0 Flash-Lite | 4,000 | 4M |

### Tier 2
| Model | RPM | TPM |
|-------|-----|-----|
| Gemini 2.5 Pro | 1,000 | 5M |
| Gemini 2.5 Flash | 2,000 | 3M |
| Gemini 2.0 Flash | 10,000 | 10M |
| Gemini 2.0 Flash-Lite | 20,000 | 10M |

### Tier 3
| Model | RPM | TPM |
|-------|-----|-----|
| Gemini 2.5 Pro | 2,000 | 8M |
| Gemini 2.5 Flash | 10,000 | 8M |
| Gemini 2.0 Flash | 30,000 | 30M |
| Gemini 2.0 Flash-Lite | 30,000 | 30M |

## Tokens Per Minute (TPM)

### Input + Output Combined
- Count both prompt and response tokens
- Cached tokens count at 25% rate
- Images: ~258 tokens per image
- Audio: ~32 tokens per second
- Video: ~258 tokens per second

## Token Limits

| Model | Max Input | Max Output |
|-------|-----------|------------|
| 2.5 Pro | 1,048,576 | 65,536 |
| 2.5 Flash | 1,048,576 | 65,536 |
| 2.5 Flash-Lite | 1,000,000 | 64,000 |
| 1.5 Pro | 2,097,152 | 8,192 |
| 1.5 Flash | 1,048,576 | 8,192 |

## Rate Limit Headers

```http
# Response headers
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1673612400
```

## Handling Rate Limits

### Exponential Backoff
```python
import time
import random
from google import genai

def retry_with_backoff(func, max_retries=5):
    for i in range(max_retries):
        try:
            return func()
        except Exception as e:
            if "429" in str(e) or "RESOURCE_EXHAUSTED" in str(e):
                wait = (2 ** i) + random.random()
                time.sleep(wait)
            else:
                raise
```

### Best Practices
1. Implement retry logic
2. Use caching for repeated content
3. Batch requests when possible
4. Monitor usage via headers
5. Set up alerts for limits

## Quota Increases

### Automatic Increases
- Gradual increase with consistent usage
- Based on payment history
- No action required

### Manual Request
1. Go to [Cloud Console](https://console.cloud.google.com)
2. Navigate to Quotas
3. Select Gemini API quotas
4. Click "Edit Quotas"
5. Provide justification

## Special Limits

### File API
- Max file size: 2GB
- Files expire: 48 hours
- Max files per request: 20

### Function Calling
- Max functions per request: 128
- Max function description: 1,000 chars

### Batch API
- Max requests per batch: 100
- Processing time: Up to 24 hours

## Error Responses

```json
{
  "error": {
    "code": 429,
    "message": "Resource exhausted",
    "status": "RESOURCE_EXHAUSTED",
    "details": [{
      "reason": "RATE_LIMIT_EXCEEDED",
      "metadata": {
        "quota_limit": "1000",
        "quota_metric": "requests_per_minute"
      }
    }]
  }
}
```

## Monitoring Usage

### Programmatic
```python
# Check remaining quota
response.headers['X-RateLimit-Remaining']
```

### Console
- View at console.cloud.google.com
- Set up budget alerts
- Export usage reports

## See Also
- [Pricing](pricing.md)
- [Error Handling](errors.md)