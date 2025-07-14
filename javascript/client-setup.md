# JavaScript Client Setup

*Advanced configuration for Google Generative AI SDK*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/get-started/node
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Key Changes: New SDK initialization, configuration in request
-->

## Quick Reference
- **Custom endpoints**: Regional support
- **Proxy support**: HTTP/HTTPS proxies
- **Timeout control**: Request timeouts
- **Custom headers**: Additional headers

## Basic Configuration

```javascript
import { GoogleGenAI } from "@google/genai";

// Basic initialization (uses GEMINI_API_KEY env var)
const ai = new GoogleGenAI({});

// With explicit API key
const ai = new GoogleGenAI({
  apiKey: 'your-api-key'
});

// With custom endpoint
const ai = new GoogleGenAI({
  baseUrl: 'https://europe-west4-generativelanguage.googleapis.com'
});
```

## Generation Configuration

```javascript
// Configuration is passed with each request
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Your prompt here",
  config: {
    temperature: 0.9,
    topK: 40,
    topP: 0.95,
    maxOutputTokens: 8192,
    stopSequences: ["END"],
  },
  safetySettings: [
    {
      category: "HARM_CATEGORY_HARASSMENT",
      threshold: "BLOCK_MEDIUM_AND_ABOVE",
    },
  ],
});
```

## Regional Endpoints

```javascript
// Europe endpoint
const ai = new GoogleGenAI({
  baseUrl: 'https://europe-west4-generativelanguage.googleapis.com'
});

// US endpoint
const ai = new GoogleGenAI({
  baseUrl: 'https://us-central1-generativelanguage.googleapis.com'
});

// Asia endpoint
const ai = new GoogleGenAI({
  baseUrl: 'https://asia-northeast1-generativelanguage.googleapis.com'
});
```

## Proxy Configuration

```javascript
import { GoogleGenAI } from "@google/genai";
import { ProxyAgent } from 'undici';

const agent = new ProxyAgent('http://proxy.company.com:8080');

const ai = new GoogleGenAI({
  fetchOptions: {
    dispatcher: agent
  }
});
```

## Custom Headers

```javascript
const ai = new GoogleGenAI({
  fetchOptions: {
    headers: {
      'X-Custom-Header': 'value',
      'User-Agent': 'MyApp/1.0'
    }
  }
});
```

## Timeout Configuration

```javascript
const ai = new GoogleGenAI({
  fetchOptions: {
    signal: AbortSignal.timeout(30000) // 30 seconds
  }
});

// Or per-request timeout
const controller = new AbortController();
setTimeout(() => controller.abort(), 30000);

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Your prompt",
  signal: controller.signal
});
```

## Retry Configuration

```javascript
class RetryableClient {
  constructor(options = {}, maxRetries = 3) {
    this.ai = new GoogleGenAI(options);
    this.maxRetries = maxRetries;
  }

  async generateWithRetry(prompt, modelConfig) {
    for (let i = 0; i < this.maxRetries; i++) {
      try {
        const result = await this.ai.models.generateContent({
          model: modelConfig.model || "gemini-2.5-flash",
          contents: prompt,
          config: modelConfig.config
        });
        return result;
      } catch (error) {
        if (i === this.maxRetries - 1) throw error;
        
        const delay = Math.min(1000 * Math.pow(2, i), 10000);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
}
```

## Environment-based Config

```javascript
const config = {
  development: {
    model: "gemini-2.5-flash",
    temperature: 1.0,
    maxOutputTokens: 2048
  },
  production: {
    model: "gemini-2.5-flash",
    temperature: 0.7,
    maxOutputTokens: 4096
  }
};

const env = process.env.NODE_ENV || 'development';

// Use config when making requests
const response = await ai.models.generateContent({
  model: config[env].model,
  contents: "Your prompt",
  config: config[env]
});
```

## Custom Fetch Implementation

```javascript
const customFetch = async (url, options) => {
  console.log(`Fetching: ${url}`);
  const start = Date.now();
  
  const response = await fetch(url, options);
  
  console.log(`Response time: ${Date.now() - start}ms`);
  return response;
};

const ai = new GoogleGenAI({
  fetch: customFetch
});
```

## Request Configuration Cache

```javascript
class ConfigCache {
  constructor(options = {}) {
    this.ai = new GoogleGenAI(options);
    this.configs = new Map();
  }

  async generateContent(modelName, prompt, configName = 'default') {
    const config = this.configs.get(configName) || {};
    
    return await this.ai.models.generateContent({
      model: modelName,
      contents: prompt,
      config: config
    });
  }
  
  setConfig(name, config) {
    this.configs.set(name, config);
  }
}
```

## Error Handling Wrapper

```javascript
class GeminiWithErrorHandling {
  constructor(options = {}) {
    this.ai = new GoogleGenAI(options);
  }
  
  async generateContent(config) {
    try {
      return await this.ai.models.generateContent(config);
    } catch (error) {
      console.error('Gemini API Error:', error);
      // Send to monitoring service
      // Handle specific error types
      if (error.message.includes('429')) {
        throw new Error('Rate limit exceeded');
      }
      throw error;
    }
  }
}
```

## TypeScript Types

```typescript
import { GoogleGenAI } from "@google/genai";

interface AppConfig {
  apiKey?: string;
  model: string;
  generation: GenerationConfig;
  safety: SafetySetting[];
}

interface GenerationConfig {
  temperature?: number;
  topK?: number;
  topP?: number;
  maxOutputTokens?: number;
}

interface SafetySetting {
  category: string;
  threshold: string;
}

class GeminiClient {
  private client: GoogleGenAI;
  private defaultConfig: AppConfig;

  constructor(config: AppConfig) {
    this.client = new GoogleGenAI({ apiKey: config.apiKey });
    this.defaultConfig = config;
  }
  
  async generate(prompt: string) {
    return await this.client.models.generateContent({
      model: this.defaultConfig.model,
      contents: prompt,
      config: this.defaultConfig.generation,
      safetySettings: this.defaultConfig.safety
    });
  }
}
```

## See Also
- [Quickstart](quickstart.md)
- [Error Handling](error-handling.md)
- [Authentication](../common/authentication.md)