# Regional Availability

*Geographic availability and data residency for Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/available-regions
Verified: 2025-01-14
Key Info: Global endpoint default, regional for data residency
Note: Free tier limited to global endpoint
-->

## Quick Reference
- **Global endpoint**: Default, routes to nearest region
- **Regional endpoints**: For data residency requirements
- **Free tier**: Global endpoint only
- **Paid tier**: All regions available

## Available Regions

### Global (Default)
- **Endpoint**: `generativelanguage.googleapis.com`
- **Routing**: Automatic nearest region
- **Best for**: Most use cases
- **Latency**: Optimized globally

### United States
- **Endpoint**: `us-central1-generativelanguage.googleapis.com`
- **Location**: Iowa, USA
- **Data residency**: US only

### Europe
- **Endpoint**: `europe-west4-generativelanguage.googleapis.com`
- **Location**: Netherlands
- **Data residency**: EU only
- **Compliance**: GDPR

### Asia Pacific
- **Endpoint**: `asia-northeast1-generativelanguage.googleapis.com`
- **Location**: Tokyo, Japan
- **Data residency**: Asia only

## Configuration

### Python
```python
from google import genai

# Regional endpoint
client = genai.Client(
    client_options={
        'api_endpoint': 'us-central1-generativelanguage.googleapis.com'
    }
)
```

### JavaScript
```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({
    apiEndpoint: 'europe-west4-generativelanguage.googleapis.com'
});
```

### REST API
```bash
# Regional URL
curl https://us-central1-generativelanguage.googleapis.com/v1beta/models
```

## Data Residency

### What Stays Regional
- API requests and responses
- Uploaded files (File API)
- Cached content
- Processing and inference

### What's Global
- Model weights
- API keys
- Usage metrics
- Billing data

## Latency Considerations

| User Location | Recommended Endpoint |
|---------------|---------------------|
| North America | us-central1 or global |
| South America | global |
| Europe | europe-west4 |
| Asia | asia-northeast1 |
| Africa | global |
| Oceania | global |

## Feature Availability

### All Regions Support
- All Gemini models
- All features (chat, vision, etc.)
- Same pricing
- Same rate limits

### Regional Differences
- Network latency
- Data residency laws
- Maintenance windows

## Compliance

### GDPR (Europe)
- Use `europe-west4` endpoint
- Data stays in EU
- Right to deletion supported

### CCPA (California)
- Use `us-central1` endpoint
- Data stays in US
- Privacy rights supported

### Other Regulations
- Check local requirements
- Contact support for guidance
- Enterprise agreements available

## Best Practices

1. **Start with global endpoint**
   - Best performance for most
   - Automatic optimization

2. **Use regional for compliance**
   - Legal requirements
   - Data sovereignty

3. **Test latency**
   ```python
   import time
   start = time.time()
   response = client.models.generate_content(
       model="gemini-2.5-flash",
       contents="test"
   )
   latency = time.time() - start
   ```

4. **Implement fallback**
   ```python
   endpoints = [
       'generativelanguage.googleapis.com',
       'us-central1-generativelanguage.googleapis.com'
   ]
   ```

## Migration Between Regions

1. Update endpoint configuration
2. No code changes needed
3. Same API interface
4. Billing continues normally

## See Also
- [Authentication](authentication.md)
- [Rate Limits](rate-limits.md)