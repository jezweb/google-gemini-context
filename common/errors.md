# Error Handling

*Common errors, status codes, and debugging for Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/troubleshooting
Verified: 2025-01-14
Key Info: 4xx client errors, 5xx server errors, safety blocks
Note: Always implement retry logic with exponential backoff
-->

## Quick Reference
- **4xx errors**: Client issues (fix your request)
- **5xx errors**: Server issues (retry with backoff)
- **SAFETY errors**: Content filtered
- **QUOTA errors**: Rate limit hit

## HTTP Status Codes

| Code | Status | Meaning |
|------|--------|---------|
| 200 | OK | Success |
| 400 | BAD_REQUEST | Invalid request format |
| 401 | UNAUTHENTICATED | Invalid API key |
| 403 | PERMISSION_DENIED | Access forbidden |
| 404 | NOT_FOUND | Resource not found |
| 429 | RESOURCE_EXHAUSTED | Rate limit exceeded |
| 500 | INTERNAL | Server error |
| 503 | UNAVAILABLE | Service down |

## Common Error Types

### Invalid API Key
```json
{
  "error": {
    "code": 401,
    "message": "API key not valid",
    "status": "UNAUTHENTICATED"
  }
}
```
**Fix**: Check API key, ensure no extra spaces

### Rate Limit Exceeded
```json
{
  "error": {
    "code": 429,
    "message": "Quota exceeded",
    "status": "RESOURCE_EXHAUSTED",
    "details": [{
      "reason": "RATE_LIMIT_EXCEEDED",
      "domain": "googleapis.com"
    }]
  }
}
```
**Fix**: Implement exponential backoff

### Invalid Request
```json
{
  "error": {
    "code": 400,
    "message": "Invalid value for 'contents[0].parts[0].text'",
    "status": "INVALID_ARGUMENT"
  }
}
```
**Fix**: Validate request structure

### Model Not Found
```json
{
  "error": {
    "code": 404,
    "message": "Model not found",
    "status": "NOT_FOUND"
  }
}
```
**Fix**: Check model name spelling

## Safety Errors

### Blocked Prompt
```json
{
  "promptFeedback": {
    "blockReason": "SAFETY",
    "safetyRatings": [{
      "category": "HARM_CATEGORY_DANGEROUS",
      "probability": "HIGH"
    }]
  }
}
```

### Blocked Response
```json
{
  "candidates": [{
    "finishReason": "SAFETY",
    "safetyRatings": [...]
  }]
}
```

## Error Handling Patterns

### Python
```python
import time
from google import genai

client = genai.Client()

def generate_with_retry(prompt, retries=3):
    for i in range(retries):
        try:
            response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt
            )
            return response.text
        except Exception as e:
            if "RESOURCE_EXHAUSTED" in str(e):
                wait_time = 2 ** i
                time.sleep(wait_time)
            elif "SAFETY" in str(e):
                return "Content filtered for safety"
            else:
                if i == retries - 1:
                    raise
                time.sleep(2 ** i)
```

### JavaScript
```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

async function generateWithRetry(prompt, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const result = await ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: prompt
      });
      return result.text;
    } catch (error) {
      if (error.message.includes('429')) {
        await new Promise(r => setTimeout(r, 2 ** i * 1000));
      } else if (error.message.includes('SAFETY')) {
        return 'Content filtered for safety';
      } else if (i === retries - 1) {
        throw error;
      } else {
        await new Promise(r => setTimeout(r, 2 ** i * 1000));
      }
    }
  }
}
```

## Finish Reasons

| Reason | Description |
|--------|-------------|
| STOP | Normal completion |
| MAX_TOKENS | Hit token limit |
| SAFETY | Content filtered |
| RECITATION | Copyright concern |
| OTHER | Unknown reason |

## Debugging Tips

### 1. Enable Logging
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### 2. Check Response Headers
```python
print(response.headers)
# X-RateLimit-Remaining: 950
```

### 3. Validate JSON
```bash
curl ... | jq .
```

### 4. Test with Minimal Request
```json
{
  "contents": [{
    "parts": [{
      "text": "Hello"
    }]
  }]
}
```

## Best Practices

1. **Always handle safety blocks**
   - Provide user feedback
   - Log for analysis

2. **Implement retry logic**
   - Exponential backoff
   - Max retry limit

3. **Monitor error rates**
   - Track by error type
   - Alert on spikes

4. **Graceful degradation**
   - Fallback responses
   - Cache successful results

## Error Recovery

### For 429 Errors
1. Check rate limit headers
2. Implement backoff
3. Consider caching
4. Upgrade tier if needed

### For 5xx Errors
1. Retry with backoff
2. Check service status
3. Have fallback ready
4. Contact support if persistent

## See Also
- [Rate Limits](rate-limits.md)
- [Safety Settings](safety.md)