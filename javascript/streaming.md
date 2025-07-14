# Streaming in JavaScript

*Real-time streaming responses with Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation#streaming
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: All Gemini models support streaming
Key Features: Real-time output, reduced latency, better UX
-->

## Quick Reference
- **Method**: `generateContentStream()`
- **Returns**: Async iterable
- **Use cases**: Chat, long outputs, real-time display
- **Supports**: Text, chat, multimodal

## Basic Streaming

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Stream text generation
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: "Write a detailed story about space exploration"
});

// Process chunks as they arrive
for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Streaming with Full Response

```javascript
// Stream and collect full response
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: "Explain quantum computing"
});

let fullText = '';
for await (const chunk of response) {
  const chunkText = chunk.text;
  process.stdout.write(chunkText);
  fullText += chunkText;
}

console.log('\n\nFull response:', fullText);

// Access aggregated response data
const aggregatedResponse = await response.response;
console.log('Token usage:', aggregatedResponse.usageMetadata);
```

## Streaming Chat Conversations

```javascript
class StreamingChat {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.history = [];
  }

  async sendMessage(message, onChunk) {
    // Add user message to history
    this.history.push({
      role: "user",
      parts: [{ text: message }]
    });

    // Stream response
    const response = await this.ai.models.generateContentStream({
      model: "gemini-2.5-flash",
      contents: this.history
    });

    let fullResponse = '';
    for await (const chunk of response) {
      const chunkText = chunk.text;
      fullResponse += chunkText;
      
      // Call callback with each chunk
      if (onChunk) {
        onChunk(chunkText);
      }
    }

    // Add assistant response to history
    this.history.push({
      role: "model",
      parts: [{ text: fullResponse }]
    });

    return fullResponse;
  }
}

// Usage
const chat = new StreamingChat();

await chat.sendMessage(
  "Tell me about the solar system",
  (chunk) => process.stdout.write(chunk)
);
```

## Streaming with Multimodal Input

```javascript
import * as fs from "node:fs";

// Stream response for image analysis
const imageData = fs.readFileSync("diagram.png", { encoding: "base64" });

const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/png",
        data: imageData,
      },
    },
    { text: "Explain this diagram in detail" }
  ]
});

for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Advanced Streaming Handler

```javascript
class StreamHandler {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async streamWithMetadata(config) {
    const startTime = Date.now();
    let firstChunkTime = null;
    let chunkCount = 0;
    let totalText = '';

    try {
      const response = await this.ai.models.generateContentStream(config);

      for await (const chunk of response) {
        if (firstChunkTime === null) {
          firstChunkTime = Date.now() - startTime;
        }

        const chunkText = chunk.text;
        totalText += chunkText;
        chunkCount++;

        // Yield chunk with metadata
        yield {
          text: chunkText,
          chunkIndex: chunkCount,
          timestamp: Date.now() - startTime
        };
      }

      // Final metadata
      const aggregated = await response.response;
      yield {
        type: 'complete',
        totalText,
        metrics: {
          firstChunkLatency: firstChunkTime,
          totalTime: Date.now() - startTime,
          chunkCount,
          tokenCount: aggregated.usageMetadata?.totalTokenCount
        }
      };

    } catch (error) {
      yield {
        type: 'error',
        error: error.message,
        timestamp: Date.now() - startTime
      };
    }
  }
}

// Usage
const handler = new StreamHandler();

const stream = handler.streamWithMetadata({
  model: "gemini-2.5-flash",
  contents: "Write a technical article about AI"
});

for await (const event of stream) {
  if (event.text) {
    process.stdout.write(event.text);
  } else if (event.type === 'complete') {
    console.log('\n\nStreaming metrics:', event.metrics);
  }
}
```

## Streaming with Cancellation

```javascript
class CancellableStream {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.cancelled = false;
  }

  async *stream(config) {
    const response = await this.ai.models.generateContentStream(config);

    for await (const chunk of response) {
      if (this.cancelled) {
        console.log('\nStream cancelled by user');
        break;
      }
      yield chunk.text;
    }
  }

  cancel() {
    this.cancelled = true;
  }
}

// Usage with timeout
const streamer = new CancellableStream();

// Cancel after 5 seconds
setTimeout(() => streamer.cancel(), 5000);

const stream = streamer.stream({
  model: "gemini-2.5-flash",
  contents: "Write a very long essay"
});

for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

## Streaming for Web Applications

```javascript
// Express.js streaming endpoint
import express from 'express';

const app = express();

app.get('/api/stream', async (req, res) => {
  const { prompt } = req.query;

  // Set headers for SSE
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    const response = await ai.models.generateContentStream({
      model: "gemini-2.5-flash",
      contents: prompt
    });

    for await (const chunk of response) {
      // Send as Server-Sent Event
      res.write(`data: ${JSON.stringify({ text: chunk.text })}\n\n`);
    }

    res.write('data: [DONE]\n\n');
    res.end();

  } catch (error) {
    res.write(`data: ${JSON.stringify({ error: error.message })}\n\n`);
    res.end();
  }
});

// Client-side consumption
async function streamFromAPI(prompt) {
  const response = await fetch(`/api/stream?prompt=${encodeURIComponent(prompt)}`);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') return;
        
        const parsed = JSON.parse(data);
        if (parsed.text) {
          document.getElementById('output').textContent += parsed.text;
        }
      }
    }
  }
}
```

## Streaming with Progress Tracking

```javascript
class ProgressStream {
  async streamWithProgress(config, onProgress) {
    const response = await this.ai.models.generateContentStream(config);
    
    const expectedTokens = config.config?.maxOutputTokens || 1000;
    let currentTokens = 0;

    for await (const chunk of response) {
      const chunkText = chunk.text;
      
      // Estimate tokens (rough approximation)
      currentTokens += chunkText.split(/\s+/).length;
      
      const progress = Math.min(
        (currentTokens / expectedTokens) * 100,
        99
      );

      onProgress({
        text: chunkText,
        progress: Math.round(progress),
        estimatedTokens: currentTokens
      });
    }

    onProgress({ progress: 100, complete: true });
  }
}
```

## Error Handling in Streams

```javascript
async function safeStream(config) {
  try {
    const response = await ai.models.generateContentStream(config);
    
    for await (const chunk of response) {
      try {
        yield { type: 'chunk', text: chunk.text };
      } catch (chunkError) {
        yield { type: 'error', error: `Chunk error: ${chunkError.message}` };
      }
    }
    
    yield { type: 'complete' };
    
  } catch (error) {
    if (error.message.includes('SAFETY')) {
      yield { type: 'safety', message: 'Content filtered for safety' };
    } else if (error.message.includes('429')) {
      yield { type: 'rateLimit', message: 'Rate limit exceeded' };
    } else {
      yield { type: 'error', error: error.message };
    }
  }
}

// Usage
const stream = safeStream({
  model: "gemini-2.5-flash",
  contents: "Your prompt"
});

for await (const event of stream) {
  switch (event.type) {
    case 'chunk':
      process.stdout.write(event.text);
      break;
    case 'complete':
      console.log('\nStreaming complete');
      break;
    case 'error':
      console.error('\nError:', event.error);
      break;
  }
}
```

## Best Practices

1. **Error Handling**: Always handle stream errors
2. **Cancellation**: Implement ability to stop streams
3. **Progress**: Show progress for long outputs
4. **Buffering**: Consider buffering for smoother display
5. **Timeouts**: Set reasonable timeouts
6. **Memory**: Clean up streams properly

## See Also
- [Text Generation](text-generation.md) - Non-streaming generation
- [Chat](chat.md) - Chat conversations
- [Server-Sent Events](../examples/javascript/sse.md)