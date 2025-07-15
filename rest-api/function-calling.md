# REST API Function Calling

*Enable models to call external functions via REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/function-calling
Verified: 2025-01-14
Models: All Gemini models
Note: REST API supports full function calling with tool definitions
-->

## Quick Reference
- **Method**: POST generateContent
- **Config**: `tools` array with function declarations
- **Response**: Contains `functionCall` objects
- **Modes**: AUTO, ANY, NONE, or specific function

## Basic Function Declaration

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "What is the weather in San Francisco?"
      }]
    }],
    "tools": [{
      "functionDeclarations": [{
        "name": "getWeather",
        "description": "Get weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City name"
            },
            "unit": {
              "type": "string",
              "enum": ["celsius", "fahrenheit"],
              "description": "Temperature unit"
            }
          },
          "required": ["location"]
        }
      }]
    }]
  }'
```

## Multiple Functions

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "What is 25 celsius in fahrenheit?"
      }]
    }],
    "tools": [{
      "functionDeclarations": [
        {
          "name": "convertTemperature",
          "description": "Convert temperature between units",
          "parameters": {
            "type": "object",
            "properties": {
              "value": {"type": "number"},
              "fromUnit": {"type": "string", "enum": ["celsius", "fahrenheit", "kelvin"]},
              "toUnit": {"type": "string", "enum": ["celsius", "fahrenheit", "kelvin"]}
            },
            "required": ["value", "fromUnit", "toUnit"]
          }
        },
        {
          "name": "calculateAverage",
          "description": "Calculate average of numbers",
          "parameters": {
            "type": "object",
            "properties": {
              "numbers": {
                "type": "array",
                "items": {"type": "number"}
              }
            },
            "required": ["numbers"]
          }
        }
      ]
    }]
  }'
```

## Handling Function Calls

```bash
# Step 1: Get function call from model
response=$(curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Get the stock price of AAPL"
      }]
    }],
    "tools": [{
      "functionDeclarations": [{
        "name": "getStockPrice",
        "description": "Get current stock price",
        "parameters": {
          "type": "object",
          "properties": {
            "symbol": {"type": "string", "description": "Stock symbol"}
          },
          "required": ["symbol"]
        }
      }]
    }]
  }')

# Extract function call (pseudo-code)
# functionCall = response.candidates[0].content.parts[0].functionCall

# Step 2: Execute function and send result back
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      {
        "parts": [{
          "text": "Get the stock price of AAPL"
        }]
      },
      {
        "role": "model",
        "parts": [{
          "functionCall": {
            "name": "getStockPrice",
            "args": {"symbol": "AAPL"}
          }
        }]
      },
      {
        "role": "user",
        "parts": [{
          "functionResponse": {
            "name": "getStockPrice",
            "response": {
              "symbol": "AAPL",
              "price": 195.89,
              "currency": "USD"
            }
          }
        }]
      }
    ]
  }'
```

## Function Calling Modes

```bash
# ANY mode - must call one function
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Calculate something"
      }]
    }],
    "tools": [{
      "functionDeclarations": [...]
    }],
    "toolConfig": {
      "functionCallingConfig": {
        "mode": "ANY"
      }
    }
  }'

# Specific function only
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [...],
    "tools": [...],
    "toolConfig": {
      "functionCallingConfig": {
        "mode": "ANY",
        "allowedFunctionNames": ["calculateSum"]
      }
    }
  }'
```

## Complex Example: Multi-turn

```bash
# Database query assistant
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "How many users signed up last month?"
      }]
    }],
    "tools": [{
      "functionDeclarations": [
        {
          "name": "queryDatabase",
          "description": "Execute SQL query",
          "parameters": {
            "type": "object",
            "properties": {
              "query": {"type": "string", "description": "SQL query"},
              "database": {"type": "string", "default": "users"}
            },
            "required": ["query"]
          }
        },
        {
          "name": "formatDate",
          "description": "Format date for queries",
          "parameters": {
            "type": "object",
            "properties": {
              "date": {"type": "string"},
              "format": {"type": "string", "default": "YYYY-MM-DD"}
            },
            "required": ["date"]
          }
        }
      ]
    }]
  }'
```

## Parallel Function Calls

```bash
# Model can call multiple functions
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Get weather for NYC and London"
      }]
    }],
    "tools": [{
      "functionDeclarations": [{
        "name": "getWeather",
        "description": "Get weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {"type": "string"}
          },
          "required": ["location"]
        }
      }]
    }]
  }'

# Response may contain multiple function calls:
# {
#   "candidates": [{
#     "content": {
#       "parts": [
#         {"functionCall": {"name": "getWeather", "args": {"location": "NYC"}}},
#         {"functionCall": {"name": "getWeather", "args": {"location": "London"}}}
#       ]
#     }
#   }]
# }
```

## Best Practices

### Function Design
- Clear, descriptive names
- Comprehensive descriptions
- Proper parameter types
- Sensible defaults

### Error Handling
```json
{
  "functionResponse": {
    "name": "getWeather",
    "response": {
      "error": "Location not found",
      "code": "INVALID_LOCATION"
    }
  }
}
```

### Response Format
```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "functionCall": {
          "name": "functionName",
          "args": {
            "param1": "value1",
            "param2": 123
          }
        }
      }]
    }
  }]
}
```

## Common Patterns

1. **Data Retrieval**
   - Database queries
   - API calls
   - File operations

2. **Calculations**
   - Math operations
   - Data analysis
   - Unit conversions

3. **Actions**
   - Send emails
   - Create tasks
   - Update records

## Limitations
- No actual function execution
- Client must handle function calls
- Function results must be sent back
- Token limits apply to all turns

## See Also
- [Structured Output](structured-output.md)
- [Text Generation](text-generation.md)
- [Chat](chat.md)