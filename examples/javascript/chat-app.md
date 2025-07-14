# JavaScript Chat App Example

*Build a simple chat application with Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Note: Complete rewrite for new SDK - no session objects, manual history management
-->

## Quick Reference
- **Framework**: Express.js
- **Features**: Streaming, history, system prompts
- **Frontend**: Basic HTML + JavaScript
- **Key Change**: No session objects - manual history management required

## Server Implementation

```javascript
// server.js
import express from 'express';
import { GoogleGenAI } from '@google/genai';

const app = express();
app.use(express.json());
app.use(express.static('public'));

const ai = new GoogleGenAI(); // Auto-reads GEMINI_API_KEY

// Store chat sessions with manual history
const sessions = new Map();

// Create new chat session
app.post('/api/chat/new', (req, res) => {
  const sessionId = Date.now().toString();
  const { systemPrompt } = req.body;
  
  sessions.set(sessionId, {
    history: [], // Manual history management
    systemPrompt: systemPrompt || "You are a helpful assistant.",
    created: new Date()
  });
  
  res.json({ sessionId });
});

// Send message
app.post('/api/chat/:sessionId/message', async (req, res) => {
  const { sessionId } = req.params;
  const { message } = req.body;
  
  const session = sessions.get(sessionId);
  if (!session) {
    return res.status(404).json({ error: 'Session not found' });
  }
  
  try {
    // Build contents array with history + new message
    const contents = [
      ...session.history,
      { role: "user", parts: [{ text: message }] }
    ];
    
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents,
      systemInstruction: session.systemPrompt
    });
    
    const responseText = response.candidates[0].content.parts[0].text;
    
    // Update history
    session.history.push(
      { role: "user", parts: [{ text: message }] },
      { role: "model", parts: [{ text: responseText }] }
    );
    
    res.json({ 
      response: responseText,
      usage: response.usageMetadata
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Stream message
app.post('/api/chat/:sessionId/stream', async (req, res) => {
  const { sessionId } = req.params;
  const { message } = req.body;
  
  const session = sessions.get(sessionId);
  if (!session) {
    return res.status(404).json({ error: 'Session not found' });
  }
  
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  try {
    // Build contents array with history + new message
    const contents = [
      ...session.history,
      { role: "user", parts: [{ text: message }] }
    ];
    
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents,
      systemInstruction: session.systemPrompt,
      config: { stream: true }
    });
    
    let fullResponse = '';
    
    for await (const chunk of response) {
      const text = chunk.candidates?.[0]?.content?.parts?.[0]?.text || '';
      if (text) {
        fullResponse += text;
        res.write(`data: ${JSON.stringify({ text })}\n\n`);
      }
    }
    
    // Update history after streaming completes
    session.history.push(
      { role: "user", parts: [{ text: message }] },
      { role: "model", parts: [{ text: fullResponse }] }
    );
    
    res.write('data: [DONE]\n\n');
    res.end();
  } catch (error) {
    res.write(`data: ${JSON.stringify({ error: error.message })}\n\n`);
    res.end();
  }
});

app.listen(3000, () => {
  console.log('Chat server running on http://localhost:3000');
});
```

## Frontend Implementation

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Gemini Chat</title>
  <style>
    #chat { height: 400px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; }
    .message { margin: 10px 0; }
    .user { color: blue; }
    .assistant { color: green; }
    .error { color: red; }
  </style>
</head>
<body>
  <h1>Gemini Chat</h1>
  
  <div id="chat"></div>
  
  <input type="text" id="message" placeholder="Type a message..." style="width: 80%">
  <button onclick="sendMessage()">Send</button>
  <button onclick="sendStreamMessage()">Stream</button>
  
  <script>
    let sessionId = null;
    
    // Initialize chat
    async function initChat() {
      const response = await fetch('/api/chat/new', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          systemPrompt: "You are a friendly chat assistant."
        })
      });
      
      const data = await response.json();
      sessionId = data.sessionId;
    }
    
    // Send regular message
    async function sendMessage() {
      const input = document.getElementById('message');
      const message = input.value;
      if (!message) return;
      
      addMessage('user', message);
      input.value = '';
      
      try {
        const response = await fetch(`/api/chat/${sessionId}/message`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message })
        });
        
        const data = await response.json();
        if (data.error) {
          addMessage('error', data.error);
        } else {
          addMessage('assistant', data.response);
        }
      } catch (error) {
        addMessage('error', error.message);
      }
    }
    
    // Send streaming message
    async function sendStreamMessage() {
      const input = document.getElementById('message');
      const message = input.value;
      if (!message) return;
      
      addMessage('user', message);
      input.value = '';
      
      const messageDiv = addMessage('assistant', '');
      
      try {
        const response = await fetch(`/api/chat/${sessionId}/stream`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message })
        });
        
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
              if (data === '[DONE]') break;
              
              try {
                const parsed = JSON.parse(data);
                if (parsed.text) {
                  messageDiv.textContent += parsed.text;
                }
              } catch (e) {}
            }
          }
        }
      } catch (error) {
        addMessage('error', error.message);
      }
    }
    
    // Add message to chat
    function addMessage(role, text) {
      const chat = document.getElementById('chat');
      const div = document.createElement('div');
      div.className = `message ${role}`;
      div.textContent = `${role}: ${text}`;
      chat.appendChild(div);
      chat.scrollTop = chat.scrollHeight;
      return div;
    }
    
    // Enter key to send
    document.getElementById('message').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') sendMessage();
    });
    
    // Initialize on load
    initChat();
  </script>
</body>
</html>
```

## Enhanced Features

### With TypeScript
```typescript
interface ChatSession {
  id: string;
  chat: any;
  created: Date;
  userId?: string;
  metadata?: Record<string, any>;
}

class ChatManager {
  private sessions = new Map<string, ChatSession>();
  
  createSession(userId?: string): string {
    // Implementation
  }
  
  async sendMessage(
    sessionId: string,
    message: string
  ): Promise<ChatResponse> {
    // Implementation
  }
}
```

### With Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 15, // 15 requests per minute
  message: 'Too many requests'
});

app.use('/api/chat', limiter);
```

### With Persistence
```javascript
const sqlite3 = require('sqlite3');
const db = new sqlite3.Database('./chat.db');

// Save messages
function saveMessage(sessionId, role, content) {
  db.run(
    'INSERT INTO messages (session_id, role, content) VALUES (?, ?, ?)',
    [sessionId, role, content]
  );
}
```

## Deployment

### Environment Variables
```bash
# .env
GEMINI_API_KEY=your-api-key
PORT=3000
NODE_ENV=production
```

### Package.json
```json
{
  "name": "gemini-chat-app",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "@google/genai": "^0.3.0",
    "express": "^4.18.0",
    "dotenv": "^16.0.0"
  }
}
```

## See Also
- [Chat Documentation](../../javascript/chat.md)
- [Streaming](../../javascript/streaming.md)
- [Error Handling](../../javascript/error-handling.md)