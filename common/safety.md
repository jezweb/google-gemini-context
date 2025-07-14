# Safety Settings

*Content filtering and safety configuration for Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/safety-settings
Verified: 2025-01-14
Key Info: 5 harm categories, 4 threshold levels
Note: HARM_CATEGORY_DANGEROUS not DANGEROUS_CONTENT
-->

## Quick Reference
- **Five harm categories**: Harassment, Hate, Sexual, Dangerous, Civic Integrity
- **Four threshold levels**: None, Only High, Medium+, Low+
- **Default**: BLOCK_NONE for newer models
- **Adjustable per request**

## Harm Categories

| Category | Description |
|----------|-------------|
| HARM_CATEGORY_HARASSMENT | Negative comments targeting identity |
| HARM_CATEGORY_HATE_SPEECH | Content promoting hatred |
| HARM_CATEGORY_SEXUALLY_EXPLICIT | Sexual content |
| HARM_CATEGORY_DANGEROUS | Harmful instructions |
| HARM_CATEGORY_CIVIC_INTEGRITY | Election/democratic process misinfo |

## Block Thresholds

| Level | Description | Use Case |
|-------|-------------|----------|
| BLOCK_NONE | No blocking | Research, moderation |
| BLOCK_ONLY_HIGH | Block only high probability | Permissive filtering |
| BLOCK_MEDIUM_AND_ABOVE | Block medium+ probability | Balanced filtering |
| BLOCK_LOW_AND_ABOVE | Block low+ probability | Strict filtering |
| HARM_BLOCK_THRESHOLD_UNSPECIFIED | Use model default | Default behavior |

## Configuration Examples

### Python
```python
from google import genai

client = genai.Client()

safety_settings = [
    {
        "category": "HARM_CATEGORY_HARASSMENT",
        "threshold": "BLOCK_MEDIUM_AND_ABOVE"
    },
    {
        "category": "HARM_CATEGORY_HATE_SPEECH",
        "threshold": "BLOCK_LOW_AND_ABOVE"
    }
]

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=prompt,
    safety_settings=safety_settings
)
```

### JavaScript
```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

const safetySettings = [
    {
        category: "HARM_CATEGORY_HARASSMENT",
        threshold: "BLOCK_MEDIUM_AND_ABOVE",
    }
];

const result = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt,
    safetySettings,
});
```

### REST API
```json
{
  "contents": [...],
  "safetySettings": [
    {
      "category": "HARM_CATEGORY_DANGEROUS",
      "threshold": "BLOCK_LOW_AND_ABOVE"
    }
  ]
}
```

## Response Safety Ratings

```json
{
  "candidates": [{
    "safetyRatings": [
      {
        "category": "HARM_CATEGORY_HARASSMENT",
        "probability": "NEGLIGIBLE",
        "probabilityScore": 0.1,
        "severity": "HARM_SEVERITY_NEGLIGIBLE",
        "severityScore": 0.1
      }
    ]
  }]
}
```

## Probability Levels
- **NEGLIGIBLE**: Very unlikely to be harmful
- **LOW**: Unlikely to be harmful
- **MEDIUM**: Possibly harmful
- **HIGH**: Likely harmful

## Blocked Response

```json
{
  "candidates": [{
    "finishReason": "SAFETY",
    "safetyRatings": [...],
    "content": {
      "parts": [],
      "role": "model"
    }
  }],
  "promptFeedback": {
    "blockReason": "SAFETY",
    "safetyRatings": [...]
  }
}
```

## Best Practices

### 1. Adjust for Use Case
- Customer service: BLOCK_MEDIUM_AND_ABOVE
- Content moderation: BLOCK_NONE + review
- Education: BLOCK_LOW_AND_ABOVE

### 2. Handle Blocked Content
```python
if response.prompt_feedback:
    if response.prompt_feedback.block_reason:
        # Handle blocked input
        return "Content filtered for safety"
```

### 3. Log Safety Ratings
- Monitor probability scores
- Track blocked requests
- Adjust thresholds based on data

### 4. User Feedback
- Allow reporting false positives
- Provide safety explanations
- Offer alternative suggestions

## Special Considerations

### Medical/Legal Content
- May trigger HARM_CATEGORY_DANGEROUS
- Consider domain-specific models
- Add disclaimers

### Creative Writing
- Fiction may trigger filters
- Use BLOCK_LOW_AND_ABOVE
- Add context in prompt

### Multi-language
- Safety works across languages
- Same thresholds apply
- Cultural context considered

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Over-blocking | Lower threshold |
| Under-blocking | Raise threshold |
| Inconsistent | Check probability scores |
| All blocked | Review prompt content |

## See Also
- [Error Handling](errors.md)
- [Usage Policies](https://ai.google.dev/gemini-api/docs/usage-policies)