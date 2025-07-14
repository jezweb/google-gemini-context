# Function Calling in JavaScript

*Enable Gemini to use tools and external functions*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/function-calling
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: Tools passed in generateContent, new response structure
-->

## Quick Reference
- **Define tools**: Describe functions for model to call
- **Automatic mode**: Model decides when to call
- **Manual mode**: You control function execution
- **Native support**: Direct tool use with type safety

## Basic Function Definition

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Define function
const getWeather = async ({ location, unit = "celsius" }) => {
  // Simulate API call
  return {
    location,
    temperature: 22,
    unit,
    conditions: "Partly cloudy"
  };
};

// Define tool
const tools = [{
  functionDeclarations: [{
    name: "getWeather",
    description: "Get current weather for a location",
    parameters: {
      type: "object",
      properties: {
        location: {
          type: "string",
          description: "City name or coordinates"
        },
        unit: {
          type: "string",
          enum: ["celsius", "fahrenheit"],
          description: "Temperature unit"
        }
      },
      required: ["location"]
    }
  }]
}];

// Use with generateContent
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "What's the weather like in Paris?",
  tools: tools
});
```

## Manual Function Calling

```javascript
// Send message that might trigger function
const result = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "What's the weather in Tokyo?",
  tools: tools
});

// Check for function calls
const functionCall = result.response.candidates?.[0]?.content?.parts?.[0]?.functionCall;
if (functionCall) {
  // Execute function
  const apiResponse = await getWeather(functionCall.args);
  
  // Send function response back
  const result2 = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: [
      { role: "user", parts: [{ text: "What's the weather in Tokyo?" }] },
      { role: "model", parts: [{ functionCall: functionCall }] },
      { role: "function", parts: [{
        functionResponse: {
          name: functionCall.name,
          response: apiResponse,
        }
      }] }
    ],
    tools: tools
  });
  
  console.log(result2.text);
}
```

## Multiple Functions

```javascript
const functions = {
  getWeather: async ({ location }) => ({
    temperature: 22,
    conditions: "Sunny"
  }),
  
  getTime: async ({ timezone }) => ({
    time: new Date().toLocaleString('en-US', { timeZone: timezone })
  }),
  
  calculate: async ({ expression }) => ({
    result: eval(expression) // Note: eval is unsafe in production
  })
};

const tools = [{
  functionDeclarations: [
    {
      name: "getWeather",
      description: "Get weather for a location",
      parameters: {
        type: "object",
        properties: {
          location: { type: "string" }
        },
        required: ["location"]
      }
    },
    {
      name: "getTime",
      description: "Get current time in timezone",
      parameters: {
        type: "object",
        properties: {
          timezone: { type: "string" }
        },
        required: ["timezone"]
      }
    },
    {
      name: "calculate",
      description: "Calculate math expressions",
      parameters: {
        type: "object",
        properties: {
          expression: { type: "string" }
        },
        required: ["expression"]
      }
    }
  ]
}];
```

## Function Calling Handler

```javascript
class FunctionHandler {
  constructor(ai, functions, tools) {
    this.ai = ai;
    this.functions = functions;
    this.tools = tools;
    this.history = [];
  }

  async sendMessage(message) {
    // Add user message to history
    this.history.push({ role: "user", parts: [{ text: message }] });
    
    let response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: this.history,
      tools: this.tools
    });
    
    // Handle function calls
    let functionCall = response.response.candidates?.[0]?.content?.parts?.[0]?.functionCall;
    while (functionCall) {
      // Add model's function call to history
      this.history.push({ role: "model", parts: [{ functionCall }] });
      
      // Execute function
      const functionResponse = await this.executeFunction(functionCall);
      
      // Add function response to history
      this.history.push({ role: "function", parts: [functionResponse] });
      
      // Get next response
      response = await this.ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: this.history,
        tools: this.tools
      });
      
      functionCall = response.response.candidates?.[0]?.content?.parts?.[0]?.functionCall;
    }
    
    // Add final response to history
    const finalText = response.text;
    this.history.push({ role: "model", parts: [{ text: finalText }] });
    
    return finalText;
  }

  async executeFunction(functionCall) {
    try {
      const func = this.functions[functionCall.name];
      if (!func) {
        throw new Error(`Function ${functionCall.name} not found`);
      }
      
      const result = await func(functionCall.args);
      
      return {
        functionResponse: {
          name: functionCall.name,
          response: result,
        },
      };
    } catch (error) {
      return {
        functionResponse: {
          name: functionCall.name,
          response: { error: error.message },
        },
      };
    }
  }
}

// Usage
const handler = new FunctionHandler(ai, functions, tools);
const response = await handler.sendMessage(
  "What's the weather in Tokyo and what time is it there?"
);
```

## Real-World Example: Database Query

```javascript
const dbFunctions = {
  queryDatabase: async ({ query, table }) => {
    // Simulate database query
    console.log(`Executing: ${query} on table ${table}`);
    
    if (table === "users" && query.includes("COUNT")) {
      return { count: 1234 };
    }
    
    return { error: "Query not supported in demo" };
  },
  
  getSchema: async ({ table }) => {
    const schemas = {
      users: ["id", "name", "email", "created_at"],
      orders: ["id", "user_id", "total", "status"]
    };
    
    return { columns: schemas[table] || [] };
  }
};

const dbTools = [{
  functionDeclarations: [
    {
      name: "queryDatabase",
      description: "Execute SQL query on database",
      parameters: {
        type: "object",
        properties: {
          query: { type: "string", description: "SQL query" },
          table: { type: "string", description: "Table name" }
        },
        required: ["query", "table"]
      }
    },
    {
      name: "getSchema",
      description: "Get table schema",
      parameters: {
        type: "object",
        properties: {
          table: { type: "string" }
        },
        required: ["table"]
      }
    }
  ]
}];
```

## Streaming with Functions

```javascript
const result = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: "Check weather in Paris",
  tools: tools
});

for await (const chunk of result) {
  const part = chunk.candidates?.[0]?.content?.parts?.[0];
  if (part?.functionCall) {
    // Handle function call
    console.log('Function call:', part.functionCall);
    // Note: With streaming, you'll need to accumulate the response
    // and handle function calls after stream completes
  } else if (part?.text) {
    process.stdout.write(part.text);
  }
}
```

## Error Handling

```javascript
const safeFunction = async (functionCall) => {
  try {
    const func = functions[functionCall.name];
    if (!func) {
      return {
        functionResponse: {
          name: functionCall.name,
          response: { error: "Function not found" }
        }
      };
    }
    
    // Validate parameters
    const result = await func(functionCall.args);
    
    return {
      functionResponse: {
        name: functionCall.name,
        response: result
      }
    };
  } catch (error) {
    console.error(`Function ${functionCall.name} error:`, error);
    
    return {
      functionResponse: {
        name: functionCall.name,
        response: { 
          error: error.message,
          status: "failed"
        }
      }
    };
  }
};
```

## Best Practices

1. **Clear descriptions**: Help model understand when to use functions
2. **Type safety**: Validate parameters before execution
3. **Error handling**: Always catch and report errors gracefully
4. **Async functions**: All functions should be async
5. **Security**: Never execute untrusted code

## See Also
- [Chat](chat.md) - Multi-turn with functions
- [Structured Output](structured-output.md) - JSON responses
- [Examples](../examples/javascript/agent.md) - Full agent