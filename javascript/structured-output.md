# Structured Output in JavaScript

*Generate JSON and structured data with type safety*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/structured-output
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: responseMimeType in config, responseSchema support
-->

## Quick Reference
- **JSON mode**: Force valid JSON output
- **Response schema**: Define exact structure
- **Type validation**: Ensure correct types
- **Enum constraints**: Limit to specific values

## Basic JSON Mode

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "List 3 programming languages with their year of creation",
  config: {
    responseMimeType: "application/json",
  }
});

const data = JSON.parse(result.text);
console.log(data);
// Output: { languages: [...] }
```

## With Response Schema

```javascript
const schema = {
  type: "object",
  properties: {
    name: { type: "string" },
    age: { type: "number" },
    email: { type: "string" },
    interests: {
      type: "array",
      items: { type: "string" }
    }
  },
  required: ["name", "age"]
};

const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Generate a user profile for a software developer",
  config: {
    responseMimeType: "application/json",
    responseSchema: schema,
  }
});

const profile = JSON.parse(result.text);
// Guaranteed to match schema
```

## Complex Schema Example

```javascript
const productSchema = {
  type: "object",
  properties: {
    products: {
      type: "array",
      items: {
        type: "object",
        properties: {
          id: { type: "string" },
          name: { type: "string" },
          price: { type: "number" },
          currency: { 
            type: "string",
            enum: ["USD", "EUR", "GBP"]
          },
          inStock: { type: "boolean" },
          categories: {
            type: "array",
            items: { type: "string" }
          },
          specifications: {
            type: "object",
            properties: {
              weight: { type: "number" },
              dimensions: {
                type: "object",
                properties: {
                  length: { type: "number" },
                  width: { type: "number" },
                  height: { type: "number" }
                }
              }
            }
          }
        },
        required: ["id", "name", "price"]
      }
    },
    totalCount: { type: "number" },
    lastUpdated: { type: "string" }
  }
};

// Use in generateContent request
const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Generate a product catalog",
  config: {
    responseMimeType: "application/json",
    responseSchema: productSchema,
  }
});
```

## Extraction Tasks

```javascript
// Extract structured data from text
const extractionSchema = {
  type: "object",
  properties: {
    people: {
      type: "array",
      items: {
        type: "object",
        properties: {
          name: { type: "string" },
          role: { type: "string" },
          company: { type: "string" }
        }
      }
    },
    companies: {
      type: "array",
      items: {
        type: "object",
        properties: {
          name: { type: "string" },
          industry: { type: "string" }
        }
      }
    },
    relationships: {
      type: "array",
      items: {
        type: "object",
        properties: {
          person: { type: "string" },
          company: { type: "string" },
          relationship: { type: "string" }
        }
      }
    }
  }
};

const text = `
John Smith is the CEO of TechCorp, a software company.
Jane Doe works as CTO at DataSystems, a data analytics firm.
`;

const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: `Extract all people, companies, and relationships from this text:\n${text}`,
  config: {
    responseMimeType: "application/json",
    responseSchema: extractionSchema,
  }
});
```

## Classification with Enums

```javascript
const classificationSchema = {
  type: "object",
  properties: {
    sentiment: {
      type: "string",
      enum: ["positive", "negative", "neutral"]
    },
    confidence: {
      type: "number",
      minimum: 0,
      maximum: 1
    },
    topics: {
      type: "array",
      items: {
        type: "string",
        enum: ["technology", "business", "health", "sports", "entertainment", "other"]
      }
    },
    language: {
      type: "string",
      enum: ["en", "es", "fr", "de", "ja", "other"]
    }
  },
  required: ["sentiment", "confidence"]
};

// Use for classification tasks
const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Classify this text: 'The new product launch was incredibly successful!'",
  config: {
    responseMimeType: "application/json",
    responseSchema: classificationSchema,
  }
});
```

## Structured Output Handler

```javascript
class StructuredGenerator {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async generate(prompt, schema, options = {}) {
    try {
      const result = await this.ai.models.generateContent({
        model: options.model || "gemini-2.5-flash",
        contents: prompt,
        config: {
          responseMimeType: "application/json",
          responseSchema: schema,
          temperature: options.temperature || 0.1,
          ...options.config
        }
      });
      
      // Parse and validate
      const data = JSON.parse(result.text);
      
      // Additional validation if needed
      if (options.validate) {
        this.validateSchema(data, schema);
      }
      
      return data;
    } catch (error) {
      if (error instanceof SyntaxError) {
        throw new Error('Invalid JSON response');
      }
      throw error;
    }
  }

  validateSchema(data, schema) {
    // Simple validation example
    if (schema.required) {
      for (const field of schema.required) {
        if (!(field in data)) {
          throw new Error(`Missing required field: ${field}`);
        }
      }
    }
  }
}
```

## Streaming Structured Output

```javascript
// Note: Streaming with JSON requires careful handling
async function streamJSON(ai, prompt, schema) {
  const result = await ai.models.generateContentStream({
    model: "gemini-2.5-flash",
    contents: prompt,
    config: {
      responseMimeType: "application/json",
      responseSchema: schema
    }
  });
  
  let buffer = '';
  for await (const chunk of result) {
    buffer += chunk.text;
  }
  
  try {
    return JSON.parse(buffer);
  } catch (error) {
    console.error('Failed to parse JSON:', buffer);
    throw error;
  }
}
```

## Common Patterns

### Form Generation
```javascript
const formSchema = {
  type: "object",
  properties: {
    fields: {
      type: "array",
      items: {
        type: "object",
        properties: {
          name: { type: "string" },
          type: { 
            type: "string",
            enum: ["text", "number", "email", "select", "checkbox"]
          },
          label: { type: "string" },
          required: { type: "boolean" },
          options: {
            type: "array",
            items: { type: "string" }
          }
        }
      }
    }
  }
};
```

### API Response Simulation
```javascript
const apiSchema = {
  type: "object",
  properties: {
    status: { type: "number" },
    data: { type: "object" },
    error: { 
      type: "object",
      properties: {
        code: { type: "string" },
        message: { type: "string" }
      }
    },
    metadata: {
      type: "object",
      properties: {
        page: { type: "number" },
        totalPages: { type: "number" },
        itemsPerPage: { type: "number" }
      }
    }
  }
};
```

## Best Practices

1. **Keep schemas simple**: Complex schemas may reduce quality
2. **Use enums**: Constrain outputs to valid values
3. **Set low temperature**: 0.1-0.3 for consistency
4. **Validate output**: Always validate parsed JSON
5. **Handle errors**: Gracefully handle parsing failures

## See Also
- [Text Generation](text-generation.md) - Basic generation
- [Function Calling](function-calling.md) - Tool responses
- [Error Handling](error-handling.md) - Error patterns