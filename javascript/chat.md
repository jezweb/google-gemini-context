# Chat Conversations in JavaScript

*Multi-turn conversations with context management*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/chat
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: Manual history management, no session objects
-->

## Quick Reference
- **Method**: `ai.chats.create()` for new chat
- **History**: Pass conversation history in request
- **Streaming**: Available for real-time responses
- **Context**: Maintain history client-side

## Basic Chat

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// For multi-turn, include history in each request
const response1 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Hello! I'm learning JavaScript."
});
console.log(response1.text);

// Include previous conversation in next request
const response2 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    { role: "user", parts: [{ text: "Hello! I'm learning JavaScript." }] },
    { role: "model", parts: [{ text: response1.text }] },
    { role: "user", parts: [{ text: "What should I learn first?" }] }
  ]
});
console.log(response2.text);
```

## Chat with History

```javascript
// Maintain conversation history
const history = [
  {
    role: "user",
    parts: [{ text: "My name is Alice" }],
  },
  {
    role: "model",
    parts: [{ text: "Nice to meet you, Alice! How can I help you?" }],
  },
];

// Add new message to history
const newMessage = { role: "user", parts: [{ text: "What's my name?" }] };

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [...history, newMessage]
});

console.log(response.text); // Will remember "Alice"
```

## Streaming Chat

```javascript
// Stream with conversation history
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: [
    ...history,  // Include previous conversation
    { role: "user", parts: [{ text: "Explain Node.js step by step" }] }
  ]
});

for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Managing Context

```javascript
class ChatSession {
  constructor(ai, model = "gemini-2.5-flash", maxHistory = 10) {
    this.ai = ai;
    this.model = model;
    this.maxHistory = maxHistory;
    this.history = [];
  }

  async sendMessage(message) {
    // Add user message to history
    const userMessage = { role: "user", parts: [{ text: message }] };
    
    // Generate response with full history
    const response = await this.ai.models.generateContent({
      model: this.model,
      contents: [...this.history, userMessage]
    });
    
    // Add to history
    this.history.push(userMessage);
    this.history.push({ 
      role: "model", 
      parts: [{ text: response.text }] 
    });

    // Trim history if too long
    if (this.history.length > this.maxHistory * 2) {
      this.history = this.history.slice(-this.maxHistory * 2);
    }

    return response.text;
  }

  getHistory() {
    return this.history;
  }

  clear() {
    this.history = [];
  }
}
```

## System Instructions in Chat

```javascript
// Include system instruction in config
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    ...history,
    { role: "user", parts: [{ text: "What are promises?" }] }
  ],
  config: {
    systemInstruction: "You are a friendly JavaScript tutor. Keep explanations simple and use examples."
  }
});

console.log(response.text);
```

## Chat with Functions

```javascript
// Define functions
const tools = [{
  functionDeclarations: [{
    name: "getWeather",
    description: "Get weather for a location",
    parameters: {
      type: "object",
      properties: {
        location: { type: "string" }
      },
      required: ["location"]
    }
  }]
}];

// Include tools in request
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    ...history,
    { role: "user", parts: [{ text: "What's the weather in Tokyo?" }] }
  ],
  tools: tools
});

// Check for function calls in response
if (response.candidates?.[0]?.content?.parts?.[0]?.functionCall) {
  // Handle function call
}
```

## Persistent Chat Storage

```javascript
class PersistentChat {
  constructor(ai, storageKey, model = "gemini-2.5-flash") {
    this.ai = ai;
    this.model = model;
    this.storageKey = storageKey;
    this.loadHistory();
  }

  loadHistory() {
    if (typeof localStorage !== 'undefined') {
      const saved = localStorage.getItem(this.storageKey);
      this.history = saved ? JSON.parse(saved) : [];
    } else {
      this.history = [];
    }
  }

  async sendMessage(message) {
    // Add user message
    const userMessage = { role: "user", parts: [{ text: message }] };
    
    // Generate response
    const response = await this.ai.models.generateContent({
      model: this.model,
      contents: [...this.history, userMessage]
    });
    
    // Update history
    this.history.push(userMessage);
    this.history.push({ 
      role: "model", 
      parts: [{ text: response.text }] 
    });
    
    this.saveHistory();
    return response.text;
  }

  saveHistory() {
    if (typeof localStorage !== 'undefined') {
      localStorage.setItem(this.storageKey, JSON.stringify(this.history));
    }
  }

  clearHistory() {
    this.history = [];
    if (typeof localStorage !== 'undefined') {
      localStorage.removeItem(this.storageKey);
    }
  }
}
```

## Token Management

```javascript
class TokenAwareChat {
  constructor(ai, model = "gemini-2.5-flash", maxTokens = 30000) {
    this.ai = ai;
    this.model = model;
    this.maxTokens = maxTokens;
    this.history = [];
    this.tokenCount = 0;
  }

  async sendMessage(message) {
    // Count tokens for current conversation
    const testContents = [
      ...this.history,
      { role: "user", parts: [{ text: message }] }
    ];
    
    const countResult = await this.ai.models.countTokens({
      model: this.model,
      contents: testContents
    });

    // Check if we need to trim history
    if (countResult.totalTokens > this.maxTokens) {
      console.log('Trimming history due to token limit');
      // Keep only recent messages
      this.history = this.history.slice(-10);
    }

    // Send message
    const response = await this.ai.models.generateContent({
      model: this.model,
      contents: testContents
    });
    
    // Update history
    this.history.push({ role: "user", parts: [{ text: message }] });
    this.history.push({ role: "model", parts: [{ text: response.text }] });
    
    return response.text;
  }
}
```

## Advanced Chat Interface

```javascript
import { GoogleGenAI } from "@google/genai";

class ChatInterface {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.sessions = new Map();
    this.defaultConfig = {
      temperature: 0.9,
      maxOutputTokens: 2048,
    };
  }

  createSession(sessionId, systemPrompt = null) {
    this.sessions.set(sessionId, { 
      history: [],
      systemPrompt,
      config: { ...this.defaultConfig }
    });
    return sessionId;
  }

  async sendMessage(sessionId, message, stream = false) {
    const session = this.sessions.get(sessionId);
    if (!session) throw new Error('Session not found');

    // Build contents with history
    const contents = [
      ...session.history,
      { role: 'user', parts: [{ text: message }] }
    ];

    // Build config
    const config = { ...session.config };
    if (session.systemPrompt) {
      config.systemInstruction = session.systemPrompt;
    }

    // Generate response
    if (stream) {
      const response = await this.ai.models.generateContentStream({
        model: "gemini-2.5-flash",
        contents,
        config
      });
      return response;
    }

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents,
      config
    });
    
    // Update history
    session.history.push(
      { role: 'user', parts: [{ text: message }] },
      { role: 'model', parts: [{ text: response.text }] }
    );
    
    return response.text;
  }

  getHistory(sessionId) {
    const session = this.sessions.get(sessionId);
    return session ? session.history : [];
  }

  deleteSession(sessionId) {
    this.sessions.delete(sessionId);
  }
}
```

## See Also
- [Text Generation](text-generation.md) - Single responses
- [Streaming](streaming.md) - Real-time output
- [Function Calling](function-calling.md) - Tool use in chat