# JavaScript Quickstart

*Get started with Gemini API in Node.js in 5 minutes*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/quickstart?lang=node
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New SDK package, simplified initialization
-->

## Quick Reference
- **Package**: `@google/genai`
- **Node**: 18+ required
- **TypeScript**: Fully supported
- **Streaming**: Built-in support

## Installation

```bash
npm install @google/genai
```

## Basic Setup

```javascript
import { GoogleGenAI } from "@google/genai";

// Initialize (API key from GEMINI_API_KEY env var)
const ai = new GoogleGenAI({});

// Basic generation
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Explain quantum computing in one paragraph"
});

console.log(response.text);
```

## First Request

```javascript
async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: "Explain quantum computing in one paragraph"
  });
  
  console.log(response.text);
}

main();
```

## Environment Setup

```bash
# .env file
GEMINI_API_KEY=your-api-key-here

# Load in Node.js
require('dotenv').config();
```

## TypeScript Setup

```typescript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

async function generate(prompt: string): Promise<string> {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt
  });
  return response.text;
}
```

## Streaming Response

```javascript
async function streamResponse() {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: "Write a story",
    config: { stream: true }
  });
  
  let text = '';
  for await (const chunk of response.stream) {
    console.log(chunk.text);
    text += chunk.text;
  }
  
  return text;
}
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt
  });
  console.log(response.text);
} catch (error) {
  if (error.message.includes('SAFETY')) {
    console.log('Content blocked for safety');
  } else if (error.message.includes('429')) {
    console.log('Rate limit hit, retry later');
  } else {
    console.error('Error:', error.message);
  }
}
```

## Complete Example

```javascript
import { GoogleGenAI } from "@google/genai";
import dotenv from 'dotenv';

dotenv.config();

async function main() {
  // Initialize
  const ai = new GoogleGenAI({});
  
  // Generate with config
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: "Hello! How are you today?",
    config: {
      temperature: 0.9,
      topK: 40,
      topP: 0.95,
      maxOutputTokens: 2048,
    }
  });
  
  console.log(response.text);
}

main().catch(console.error);
```

## Package.json Example

```json
{
  "name": "gemini-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google/genai": "latest",
    "dotenv": "^16.0.0"
  }
}
```

## Next Steps
- [Client Setup](client-setup.md) - Advanced configuration
- [Text Generation](text-generation.md) - Generation options
- [Chat](chat.md) - Multi-turn conversations

## Common Issues

| Problem | Solution |
|---------|----------|
| `Cannot find module` | Use Node 18+, check package.json |
| `API key not valid` | Check env variable loading |
| `429 Error` | Implement rate limit handling |

## See Also
- [Official SDK Docs](https://ai.google.dev/tutorials/node_quickstart)
- [Error Handling](error-handling.md)