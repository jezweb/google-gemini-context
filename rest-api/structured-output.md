# REST API Structured Output

*Generate JSON responses with schema validation*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/structured-output
Verified: 2025-01-14
Models: All Gemini models
Note: REST API supports JSON mode and schema validation
-->

## Quick Reference
- **Method**: POST generateContent
- **Config**: `responseMimeType` and `responseSchema`
- **Modes**: JSON mode, JSON with schema
- **Use case**: Structured data extraction

## Basic JSON Mode

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "List 3 popular cookies as JSON"
      }]
    }],
    "generationConfig": {
      "responseMimeType": "application/json"
    }
  }'
```

## With JSON Schema

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Extract product information from: The new iPhone 15 Pro costs $999 and comes in titanium finish."
      }]
    }],
    "generationConfig": {
      "responseMimeType": "application/json",
      "responseSchema": {
        "type": "object",
        "properties": {
          "product": {
            "type": "string",
            "description": "Product name"
          },
          "price": {
            "type": "number",
            "description": "Price in USD"
          },
          "features": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": ["product", "price"]
      }
    }
  }'
```

## Complex Schema Example

```bash
# Recipe extraction with nested objects
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Extract recipe: To make chocolate chip cookies, mix 2 cups flour, 1 cup butter, 1 cup sugar, 2 eggs, and 1 cup chocolate chips. Bake at 350F for 12 minutes."
      }]
    }],
    "generationConfig": {
      "responseMimeType": "application/json",
      "responseSchema": {
        "type": "object",
        "properties": {
          "recipeName": {"type": "string"},
          "ingredients": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "name": {"type": "string"},
                "amount": {"type": "string"}
              },
              "required": ["name", "amount"]
            }
          },
          "instructions": {
            "type": "object",
            "properties": {
              "temperature": {"type": "integer"},
              "duration": {"type": "string"},
              "steps": {
                "type": "array",
                "items": {"type": "string"}
              }
            }
          }
        },
        "required": ["recipeName", "ingredients"]
      }
    }
  }'
```

## Array Output

```bash
# Generate array of items
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "List 5 programming languages with their use cases"
      }]
    }],
    "generationConfig": {
      "responseMimeType": "application/json",
      "responseSchema": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "language": {"type": "string"},
            "useCase": {"type": "string"},
            "popularity": {
              "type": "string",
              "enum": ["high", "medium", "low"]
            }
          },
          "required": ["language", "useCase"]
        }
      }
    }
  }'
```

## Enum Constraints

```bash
# Using enum for constrained values
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Classify this text sentiment: I absolutely love this product!"
      }]
    }],
    "generationConfig": {
      "responseMimeType": "application/json",
      "responseSchema": {
        "type": "object",
        "properties": {
          "sentiment": {
            "type": "string",
            "enum": ["positive", "negative", "neutral"]
          },
          "confidence": {
            "type": "number",
            "minimum": 0,
            "maximum": 1
          }
        },
        "required": ["sentiment", "confidence"]
      }
    }
  }'
```

## Best Practices

### Schema Design
- Keep schemas simple and focused
- Use descriptive property names
- Include descriptions for clarity
- Mark required fields appropriately

### Error Handling
```bash
# Check for schema validation errors
if [[ $response == *"error"* ]]; then
  echo "Schema validation failed"
fi
```

### Performance Tips
- Smaller schemas generate faster
- Avoid deeply nested structures
- Use enums for constrained outputs
- Test schemas with various inputs

## Common Use Cases

1. **Data Extraction**
   - Form parsing
   - Invoice processing
   - Resume parsing

2. **Classification**
   - Sentiment analysis
   - Content categorization
   - Intent detection

3. **Transformation**
   - Text to structured data
   - Format conversion
   - Data normalization

## Limitations
- Schema complexity affects performance
- Very large schemas may timeout
- Not all JSON Schema features supported
- Response must fit token limits

## See Also
- [Text Generation](text-generation.md)
- [Function Calling](function-calling.md)
- [Error Responses](error-responses.md)