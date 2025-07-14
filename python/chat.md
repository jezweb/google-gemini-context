# Chat Conversations in Python

*Multi-turn conversations with context management*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/chat
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: Manual history management, no start_chat() method
-->

## Quick Reference
- **Method**: Include conversation history in each request
- **History**: Manual context tracking via contents array
- **Streaming**: `generate_content_stream()` method
- **Async**: `client.aio` for async operations

## Basic Chat

```python
from google import genai

client = genai.Client()

# For multi-turn conversations, include history in each request
response1 = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Hello! I'm learning Python."
)
print(response1.text)

# Include previous conversation in next request
response2 = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"role": "user", "parts": [{"text": "Hello! I'm learning Python."}]},
        {"role": "model", "parts": [{"text": response1.text}]},
        {"role": "user", "parts": [{"text": "What should I learn first?"}]}
    ]
)
print(response2.text)
```

## Chat with History

```python
# Maintain conversation history
history = [
    {"role": "user", "parts": [{"text": "My name is Alice"}]},
    {"role": "model", "parts": [{"text": "Nice to meet you, Alice! How can I help?"}]},
]

# Add new message to history
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        *history,
        {"role": "user", "parts": [{"text": "What's my name?"}]}
    ]
)
print(response.text)  # Will remember "Alice"

# Update history
history.extend([
    {"role": "user", "parts": [{"text": "What's my name?"}]},
    {"role": "model", "parts": [{"text": response.text}]}
])
```

## Streaming Chat

```python
# Stream with conversation history
for chunk in client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents=[
        *history,  # Include previous conversation
        {"role": "user", "parts": [{"text": "Explain Python decorators step by step"}]}
    ]
):
    print(chunk.text, end='', flush=True)
```

## Async Chat

```python
import asyncio
from google import genai

async def chat_async():
    client = genai.Client()
    history = []
    
    # First message
    response = await client.aio.models.generate_content(
        model="gemini-2.5-flash",
        contents="Hello!"
    )
    print(response.text)
    
    # Update history
    history.extend([
        {"role": "user", "parts": [{"text": "Hello!"}]},
        {"role": "model", "parts": [{"text": response.text}]}
    ])
    
    # Streaming async
    async for chunk in client.aio.models.generate_content_stream(
        model="gemini-2.5-flash",
        contents=[
            *history,
            {"role": "user", "parts": [{"text": "Tell me a story"}]}
        ]
    ):
        print(chunk.text, end='')

asyncio.run(chat_async())
```

## Managing Context

```python
class ChatSession:
    def __init__(self, model_name='gemini-2.5-flash', max_history=20):
        self.client = genai.Client()
        self.model_name = model_name
        self.max_history = max_history
        self.history = []
    
    def send_message(self, message: str) -> str:
        # Add user message to history
        user_message = {"role": "user", "parts": [{"text": message}]}
        
        # Generate response with full history
        response = self.client.models.generate_content(
            model=self.model_name,
            contents=[*self.history, user_message]
        )
        response_text = response.text
        
        # Update history
        self.history.append(user_message)
        self.history.append({"role": "model", "parts": [{"text": response_text}]})
        
        # Trim history if needed
        if len(self.history) > self.max_history * 2:
            self.history = self.history[-(self.max_history * 2):]
        
        return response_text
    
    def get_history(self):
        return self.history
    
    def clear(self):
        self.history = []
    
    def save_history(self, filename: str):
        import json
        with open(filename, 'w') as f:
            json.dump(self.history, f)
    
    def load_history(self, filename: str):
        import json
        with open(filename, 'r') as f:
            self.history = json.load(f)
```

## System Instructions in Chat

```python
config = genai.GenerateContentConfig(
    system_instruction="You are a Python tutor. Keep explanations simple with examples."
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are list comprehensions?",
    config=config
)
print(response.text)
```

## Chat with Functions

```python
def get_weather(location: str, unit: str = "celsius") -> dict:
    # Mock weather data
    return {
        "location": location,
        "temperature": 22,
        "unit": unit,
        "conditions": "Partly cloudy"
    }

# Define tools
tools = [{
    "function_declarations": [{
        "name": "get_weather",
        "description": "Get current weather for a location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"},
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"]
                }
            },
            "required": ["location"]
        }
    }]
}]

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What's the weather in Tokyo?",
    tools=tools
)
# Handle function calls in response
```

## Token-Aware Chat

```python
class TokenAwareChat:
    def __init__(self, model_name='gemini-2.5-flash', max_tokens=30000):
        self.client = genai.Client()
        self.model_name = model_name
        self.max_tokens = max_tokens
        self.current_tokens = 0
        self.history = []
    
    def send_message(self, message: str) -> str:
        # Count tokens in current conversation
        test_contents = [
            *self.history,
            {"role": "user", "parts": [{"text": message}]}
        ]
        
        count_result = self.client.models.count_tokens(
            model=self.model_name,
            contents=test_contents
        )
        
        # Check if we need to trim history
        if count_result.total_tokens > self.max_tokens:
            print("Trimming history due to token limit")
            # Keep only recent messages
            self.history = self.history[-10:]
        
        # Send message
        response = self.client.models.generate_content(
            model=self.model_name,
            contents=test_contents
        )
        
        # Update history
        self.history.append({"role": "user", "parts": [{"text": message}]})
        self.history.append({"role": "model", "parts": [{"text": response.text}]})
        
        return response.text
    
    def get_token_usage(self):
        return {
            'current': self.current_tokens,
            'max': self.max_tokens,
            'remaining': self.max_tokens - self.current_tokens
        }
```

## Advanced Chat Manager

```python
from typing import Dict, List, Optional
from datetime import datetime
import json

class ChatManager:
    def __init__(self):
        self.client = genai.Client()
        self.sessions: Dict[str, Dict] = {}
        self.default_model = 'gemini-2.5-flash'
    
    def create_session(
        self,
        session_id: str,
        model_name: Optional[str] = None,
        system_instruction: Optional[str] = None
    ) -> str:
        self.sessions[session_id] = {
            'model_name': model_name or self.default_model,
            'system_instruction': system_instruction,
            'created_at': datetime.now(),
            'history': [],
            'messages': []
        }
        
        return session_id
    
    def send_message(
        self,
        session_id: str,
        message: str,
        stream: bool = False
    ):\n        if session_id not in self.sessions:
            raise ValueError(f"Session {session_id} not found")
        
        session = self.sessions[session_id]
        
        # Record message
        session['messages'].append({
            'role': 'user',
            'content': message,
            'timestamp': datetime.now().isoformat()
        })
        
        # Prepare config
        config = genai.GenerateContentConfig()
        if session['system_instruction']:
            config.system_instruction = session['system_instruction']
        
        # Prepare contents
        contents = [
            *session['history'],
            {"role": "user", "parts": [{"text": message}]}
        ]
        
        # Send to model
        if stream:
            response = self.client.models.generate_content_stream(
                model=session['model_name'],
                contents=contents,
                config=config
            )
            return self._stream_response(session, response, message)
        
        response = self.client.models.generate_content(
            model=session['model_name'],
            contents=contents,
            config=config
        )
        
        # Update history
        session['history'].append({"role": "user", "parts": [{"text": message}]})
        session['history'].append({"role": "model", "parts": [{"text": response.text}]})
        
        # Record response
        response_text = response.text
        session['messages'].append({
            'role': 'assistant',
            'content': response_text,
            'timestamp': datetime.now().isoformat()
        })
        
        return response_text
    
    def _stream_response(self, session, response, user_message):
        full_response = ""
        for chunk in response:
            full_response += chunk.text
            yield chunk.text
        
        # Update history
        session['history'].append({"role": "user", "parts": [{"text": user_message}]})
        session['history'].append({"role": "model", "parts": [{"text": full_response}]})
        
        # Record complete response
        session['messages'].append({
            'role': 'assistant',
            'content': full_response,
            'timestamp': datetime.now().isoformat()
        })
    
    def get_history(self, session_id: str) -> List[Dict]:
        if session_id not in self.sessions:
            return []
        return self.sessions[session_id]['messages']
    
    def export_session(self, session_id: str) -> str:
        if session_id not in self.sessions:
            raise ValueError(f"Session {session_id} not found")
        
        session_data = {
            'session_id': session_id,
            'created_at': self.sessions[session_id]['created_at'].isoformat(),
            'messages': self.sessions[session_id]['messages']
        }
        
        return json.dumps(session_data, indent=2)
    
    def delete_session(self, session_id: str):
        if session_id in self.sessions:
            del self.sessions[session_id]

# Usage
manager = ChatManager()
session_id = manager.create_session(
    "user123",
    system_instruction="You are a helpful Python tutor"
)

response = manager.send_message(session_id, "Explain generators")
print(response)

# Stream response
for chunk in manager.send_message(session_id, "Show me an example", stream=True):
    print(chunk, end='')
```

## Persistent Chat with SQLite

```python
import sqlite3
import json
from datetime import datetime

class PersistentChat:
    def __init__(self, db_path: str = "chat_history.db"):
        self.db_path = db_path
        self.client = genai.Client()
        self.model_name = 'gemini-2.5-flash'
        self._init_db()
        self.history = []
    
    def _init_db(self):
        conn = sqlite3.connect(self.db_path)
        conn.execute('''
            CREATE TABLE IF NOT EXISTS chat_sessions (
                session_id TEXT PRIMARY KEY,
                created_at TIMESTAMP,
                history TEXT
            )
        ''')
        conn.commit()
        conn.close()
    
    def load_or_create_session(self, session_id: str):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute(
            "SELECT history FROM chat_sessions WHERE session_id = ?",
            (session_id,)
        )
        result = cursor.fetchone()
        
        if result:
            self.history = json.loads(result[0])
        else:
            self.history = []
            cursor.execute(
                "INSERT INTO chat_sessions VALUES (?, ?, ?)",
                (session_id, datetime.now(), "[]")
            )
            conn.commit()
        
        conn.close()
        self.session_id = session_id
    
    def send_message(self, message: str) -> str:
        # Add user message
        user_message = {"role": "user", "parts": [{"text": message}]}
        
        # Generate response
        response = self.client.models.generate_content(
            model=self.model_name,
            contents=[*self.history, user_message]
        )
        
        # Update history
        self.history.append(user_message)
        self.history.append({"role": "model", "parts": [{"text": response.text}]})
        
        self._save_history()
        return response.text
    
    def _save_history(self):
        conn = sqlite3.connect(self.db_path)
        history_json = json.dumps(self.history)
        
        conn.execute(
            "UPDATE chat_sessions SET history = ? WHERE session_id = ?",
            (history_json, self.session_id)
        )
        conn.commit()
        conn.close()
```

## See Also
- [Text Generation](text-generation.md) - Single responses
- [Async](async.md) - Async operations
- [Function Calling](function-calling.md) - Tool use in chat