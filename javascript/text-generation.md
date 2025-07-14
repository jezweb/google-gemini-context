# Text Generation in JavaScript

*Generate text completions with Gemini models*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New SDK package, new initialization pattern
-->

## Quick Reference
- **Method**: `ai.models.generateContent()` for single response
- **Streaming**: `ai.models.generateContentStream()` for real-time
- **Config**: Pass options in `config` object
- **Temperature**: 0.0 (deterministic) to 2.0 (creative)

## Basic Generation

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Write a haiku about coding"
});

console.log(response.text);
```

## Generation Parameters

```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Write a creative story",
  config: {
    temperature: 0.9,        // Creativity (0.0-2.0)
    topK: 40,               // Top K sampling
    topP: 0.95,             // Nucleus sampling
    maxOutputTokens: 2048,   // Max response length
    stopSequences: ["END"], // Stop generation triggers
  }
});
```

## System Instructions

```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "How do I sort an array?",
  config: {
    systemInstruction: "You are a helpful coding assistant. Always provide code examples."
  }
});

console.log(response.text);
```

## Streaming Responses

```javascript
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: "Explain machine learning"
});

let fullText = '';
for await (const chunk of response) {
  process.stdout.write(chunk.text); // Print as it arrives
  fullText += chunk.text;
}

console.log('Complete response:', fullText);
```

## Multiple Candidates

```javascript
// Note: candidateCount may not be supported in all models
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Write a tagline",
  config: {
    candidateCount: 3,  // Generate 3 variations
    temperature: 0.8,
  }
});

// Access candidates if available
if (response.candidates) {
  response.candidates.forEach((candidate, i) => {
    console.log(`Option ${i + 1}: ${candidate.content.parts[0].text}`);
  });
}
```

## Token Counting

```javascript
// Count tokens before generating
const countResponse = await ai.models.countTokens({
  model: "gemini-2.5-flash",
  contents: "Your prompt here"
});
console.log('Input tokens:', countResponse.totalTokens);

// Get usage after generation
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Your prompt"
});

// Usage metadata may be in response
if (response.usageMetadata) {
  console.log('Total tokens:', response.usageMetadata.totalTokenCount);
}
```

## Advanced Example

```javascript
import { GoogleGenAI } from "@google/genai";

class TextGenerator {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async generate(prompt, options = {}) {
    const config = {
      model: options.model || "gemini-2.5-flash",
      contents: prompt,
      config: {
        temperature: options.temperature || 0.7,
        maxOutputTokens: options.maxTokens || 2048,
        topP: options.topP || 0.95,
        topK: options.topK || 40,
      }
    };

    try {
      if (options.stream) {
        return this.streamGenerate(config);
      }
      
      const response = await this.ai.models.generateContent(config);
      return {
        text: response.text,
        safe: true
      };
    } catch (error) {
      if (error.message.includes('SAFETY')) {
        return { text: 'Content filtered for safety', safe: false };
      }
      throw error;
    }
  }

  async streamGenerate(config) {
    const response = await this.ai.models.generateContentStream(config);
    return response;
  }
}

// Usage
const generator = new TextGenerator();
const response = await generator.generate(
  "Write a function to reverse a string",
  { temperature: 0.3, maxTokens: 500 }
);
console.log(response.text);
```

## Common Patterns

### Retry with Backoff
```javascript
async function generateWithRetry(ai, prompt, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: prompt
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
}
```

### Template System
```javascript
function fillTemplate(template, vars) {
  return template.replace(/{{(\w+)}}/g, (_, key) => vars[key] || '');
}

const template = "Write a {{style}} story about {{topic}}";
const prompt = fillTemplate(template, {
  style: "funny",
  topic: "a programmer's first bug"
});
```

## See Also
- [Chat](chat.md) - Multi-turn conversations
- [Streaming](streaming.md) - Real-time responses
- [Structured Output](structured-output.md) - JSON responses