# Structured Output in Python

*Generate JSON and structured data with type safety*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/structured-output
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New client pattern, config-based JSON mode
-->

## Quick Reference
- **JSON mode**: Force valid JSON output
- **Response schema**: Define exact structure
- **Type validation**: Ensure correct types
- **Enum constraints**: Limit to specific values

## Basic JSON Mode

```python
from google import genai

client = genai.Client()

# Configure for JSON output
config = genai.GenerateContentConfig(
    response_mime_type="application/json"
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="List 3 programming languages with their year of creation",
    config=config
)

# Parse JSON response
import json
data = json.loads(response.text)
print(data)
# Output: {"languages": [...]}
```

## With Response Schema

```python
# Define schema
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "number"},
        "email": {"type": "string"},
        "interests": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["name", "age"]
}

# Configure with schema
config = genai.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=schema
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Generate a user profile for a software developer",
    config=config
)

profile = json.loads(response.text)
# Guaranteed to match schema
```

## Complex Schema Example

```python
# Product catalog schema
product_schema = {
    "type": "object",
    "properties": {
        "products": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "id": {"type": "string"},
                    "name": {"type": "string"},
                    "price": {"type": "number"},
                    "currency": {
                        "type": "string",
                        "enum": ["USD", "EUR", "GBP"]
                    },
                    "in_stock": {"type": "boolean"},
                    "categories": {
                        "type": "array",
                        "items": {"type": "string"}
                    },
                    "specifications": {
                        "type": "object",
                        "properties": {
                            "weight": {"type": "number"},
                            "dimensions": {
                                "type": "object",
                                "properties": {
                                    "length": {"type": "number"},
                                    "width": {"type": "number"},
                                    "height": {"type": "number"}
                                }
                            }
                        }
                    }
                },
                "required": ["id", "name", "price"]
            }
        },
        "total_count": {"type": "number"},
        "last_updated": {"type": "string"}
    }
}

# Use with generation
config = genai.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=product_schema
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Generate a product catalog for electronics",
    config=config
)

catalog = json.loads(response.text)
```

## Extraction Tasks

```python
# Extract structured data from text
extraction_schema = {
    "type": "object",
    "properties": {
        "people": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "role": {"type": "string"},
                    "company": {"type": "string"}
                }
            }
        },
        "companies": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "industry": {"type": "string"}
                }
            }
        },
        "relationships": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "person": {"type": "string"},
                    "company": {"type": "string"},
                    "relationship": {"type": "string"}
                }
            }
        }
    }
}

text = """
John Smith is the CEO of TechCorp, a software company.
Jane Doe works as CTO at DataSystems, a data analytics firm.
"""

config = genai.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=extraction_schema
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=f"Extract all people, companies, and relationships from this text:\n{text}",
    config=config
)

extracted_data = json.loads(response.text)
```

## Classification with Enums

```python
# Classification schema with enums
classification_schema = {
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
        },
        "topics": {
            "type": "array",
            "items": {
                "type": "string",
                "enum": ["technology", "business", "health", "sports", "entertainment", "other"]
            }
        },
        "language": {
            "type": "string",
            "enum": ["en", "es", "fr", "de", "ja", "other"]
        }
    },
    "required": ["sentiment", "confidence"]
}

# Classify text
config = genai.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=classification_schema
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Classify this text: 'The new product launch was incredibly successful!'",
    config=config
)

classification = json.loads(response.text)
print(f"Sentiment: {classification['sentiment']} ({classification['confidence']:.2f})")
```

## Structured Output Handler

```python
from typing import Dict, Any, Optional
import json
import jsonschema

class StructuredGenerator:
    def __init__(self):
        self.client = genai.Client()
    
    def generate(
        self,
        prompt: str,
        schema: Dict[str, Any],
        model: str = "gemini-2.5-flash",
        temperature: float = 0.1,
        validate: bool = True
    ) -> Dict[str, Any]:
        """Generate structured output with schema"""
        config = genai.GenerateContentConfig(
            response_mime_type="application/json",
            response_schema=schema,
            temperature=temperature
        )
        
        try:
            response = self.client.models.generate_content(
                model=model,
                contents=prompt,
                config=config
            )
            
            # Parse JSON
            data = json.loads(response.text)
            
            # Additional validation if requested
            if validate:
                self.validate_schema(data, schema)
            
            return data
            
        except json.JSONDecodeError as e:
            raise ValueError(f"Invalid JSON response: {e}")
        except Exception as e:
            raise RuntimeError(f"Generation failed: {e}")
    
    def validate_schema(self, data: Dict, schema: Dict):
        """Validate data against schema"""
        try:
            jsonschema.validate(instance=data, schema=schema)
        except jsonschema.exceptions.ValidationError as e:
            raise ValueError(f"Schema validation failed: {e}")
    
    def batch_generate(
        self,
        prompts: List[str],
        schema: Dict[str, Any],
        **kwargs
    ) -> List[Dict[str, Any]]:
        """Generate structured output for multiple prompts"""
        results = []
        for prompt in prompts:
            try:
                result = self.generate(prompt, schema, **kwargs)
                results.append(result)
            except Exception as e:
                results.append({"error": str(e)})
        return results

# Usage
generator = StructuredGenerator()
result = generator.generate(
    "Generate a product listing",
    product_schema,
    temperature=0.3
)
```

## Streaming Structured Output

```python
# Note: Streaming with JSON requires careful handling
async def stream_json(prompt: str, schema: Dict):
    config = genai.GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=schema
    )
    
    buffer = ""
    async for chunk in client.aio.models.generate_content_stream(
        model="gemini-2.5-flash",
        contents=prompt,
        config=config
    ):
        buffer += chunk.text
    
    try:
        return json.loads(buffer)
    except json.JSONDecodeError as e:
        print(f"Failed to parse JSON: {buffer}")
        raise e
```

## Common Patterns

### Form Generation
```python
form_schema = {
    "type": "object",
    "properties": {
        "fields": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "type": {
                        "type": "string",
                        "enum": ["text", "number", "email", "select", "checkbox"]
                    },
                    "label": {"type": "string"},
                    "required": {"type": "boolean"},
                    "options": {
                        "type": "array",
                        "items": {"type": "string"}
                    }
                }
            }
        }
    }
}

# Generate dynamic form
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Create a user registration form",
    config=genai.GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=form_schema
    )
)
```

### API Response Simulation
```python
api_schema = {
    "type": "object",
    "properties": {
        "status": {"type": "number"},
        "data": {"type": "object"},
        "error": {
            "type": "object",
            "properties": {
                "code": {"type": "string"},
                "message": {"type": "string"}
            }
        },
        "metadata": {
            "type": "object",
            "properties": {
                "page": {"type": "number"},
                "total_pages": {"type": "number"},
                "items_per_page": {"type": "number"}
            }
        }
    }
}

# Simulate API response
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Generate a paginated API response for products",
    config=genai.GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=api_schema
    )
)
```

## Best Practices

1. **Keep schemas simple**: Complex schemas may reduce quality
2. **Use enums**: Constrain outputs to valid values
3. **Set low temperature**: 0.1-0.3 for consistency
4. **Validate output**: Always validate parsed JSON
5. **Handle errors**: Gracefully handle parsing failures
6. **Test schemas**: Verify schemas produce expected results

## See Also
- [Text Generation](text-generation.md) - Basic generation
- [Function Calling](function-calling.md) - Tool responses
- [Chat](chat.md) - Structured chat responses