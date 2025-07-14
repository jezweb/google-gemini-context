# Streaming in REST API

*Real-time streaming responses with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation#streaming
API Version: v1beta
Verified: 2025-01-14
Models: All Gemini models support streaming
Key Features: Server-sent events, real-time output
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/{model}:streamGenerateContent`
- **Method**: POST
- **Format**: Server-sent events (SSE)
- **Content-Type**: `text/event-stream`

## Basic Streaming Request

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Write a story about space exploration"
      }]
    }]
  }'
```

## Response Format

```
data: {"candidates":[{"content":{"parts":[{"text":"In the year"}],"role":"model"}],"finishReason":"STOP"}

data: {"candidates":[{"content":{"parts":[{"text":" 2157, humanity"}],"role":"model"}],"finishReason":"STOP"}

data: {"candidates":[{"content":{"parts":[{"text":" had finally"}],"role":"model"}],"finishReason":"STOP"}

data: [DONE]
```

## Streaming with cURL

```bash
#!/bin/bash
# stream_gemini.sh

PROMPT="$1"

curl -N -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PROMPT"'"
      }]
    }]
  }' | while IFS= read -r line; do
    if [[ $line == data:* ]]; then
      # Extract JSON and parse text
      json_data="${line#data: }"
      if [ "$json_data" != "[DONE]" ]; then
        echo "$json_data" | jq -r '.candidates[0].content.parts[0].text' 2>/dev/null | tr -d '\n'
      fi
    fi
  done
```

## Python Implementation

```python
import requests
import json
import sseclient

def stream_gemini(prompt, api_key):
    """Stream responses from Gemini API"""
    url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent"
    
    headers = {
        "x-goog-api-key": api_key,
        "Content-Type": "application/json"
    }
    
    data = {
        "contents": [{
            "parts": [{
                "text": prompt
            }]
        }]
    }
    
    response = requests.post(url, headers=headers, json=data, stream=True)
    client = sseclient.SSEClient(response)
    
    for event in client.events():
        if event.data != "[DONE]":
            try:
                data = json.loads(event.data)
                text = data["candidates"][0]["content"]["parts"][0]["text"]
                yield text
            except (json.JSONDecodeError, KeyError):
                continue

# Usage
for chunk in stream_gemini("Tell me about AI", "your-api-key"):
    print(chunk, end="", flush=True)
```

## JavaScript Implementation

```javascript
async function streamGemini(prompt, apiKey) {
    const response = await fetch(
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent",
        {
            method: "POST",
            headers: {
                "x-goog-api-key": apiKey,
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                contents: [{
                    parts: [{
                        text: prompt
                    }]
                }]
            })
        }
    );
    
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
                
                try {
                    const parsed = JSON.parse(data);
                    const text = parsed.candidates[0].content.parts[0].text;
                    console.log(text);
                } catch (e) {
                    // Skip invalid JSON
                }
            }
        }
    }
}
```

## Streaming Chat

```bash
#!/bin/bash
# streaming_chat.sh

HISTORY_FILE="/tmp/chat_history.json"

# Initialize empty history
echo '[]' > "$HISTORY_FILE"

add_to_history() {
    local role="$1"
    local text="$2"
    
    # Add message to history
    jq '. += [{
        "role": "'"$role"'",
        "parts": [{
            "text": "'"$text"'"
        }]
    }]' "$HISTORY_FILE" > "/tmp/temp_history.json"
    
    mv "/tmp/temp_history.json" "$HISTORY_FILE"
}

stream_chat() {
    local message="$1"
    
    # Add user message
    add_to_history "user" "$message"
    
    # Get current history
    local history=$(cat "$HISTORY_FILE")
    
    # Stream response
    local response=""
    
    curl -N -X POST \
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
        -H "x-goog-api-key: $GEMINI_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
            "contents": '"$history"'
        }' | while IFS= read -r line; do
        if [[ $line == data:* ]]; then
            json_data="${line#data: }"
            if [ "$json_data" != "[DONE]" ]; then
                text=$(echo "$json_data" | jq -r '.candidates[0].content.parts[0].text' 2>/dev/null)
                if [ "$text" != "null" ]; then
                    echo -n "$text"
                    response+="$text"
                fi
            fi
        fi
    done
    
    # Add assistant response to history
    add_to_history "model" "$response"
}

# Interactive chat loop
while true; do
    echo -n "You: "
    read -r user_message
    
    if [ "$user_message" = "quit" ]; then
        break
    fi
    
    echo -n "AI: "
    stream_chat "$user_message"
    echo ""
done
```

## Advanced Configuration

```json
{
  "contents": [{
    "parts": [{
      "text": "Write a technical article"
    }]
  }],
  "generationConfig": {
    "temperature": 0.7,
    "topK": 40,
    "topP": 0.8,
    "maxOutputTokens": 1024,
    "stopSequences": ["END"]
  },
  "safetySettings": [{
    "category": "HARM_CATEGORY_HARASSMENT",
    "threshold": "BLOCK_MEDIUM_AND_ABOVE"
  }]
}
```

## Multimodal Streaming

```bash
# Stream with image
IMAGE_BASE64=$(base64 -i image.jpg)

curl -N -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "'"$IMAGE_BASE64"'"
          }
        },
        {
          "text": "Describe this image"
        }
      ]
    }]
  }' | while IFS= read -r line; do
    # Process streaming response
    if [[ $line == data:* ]]; then
      json_data="${line#data: }"
      if [ "$json_data" != "[DONE]" ]; then
        echo "$json_data" | jq -r '.candidates[0].content.parts[0].text' 2>/dev/null | tr -d '\n'
      fi
    fi
  done
```

## WebSocket Alternative

```python
import websocket
import json
import ssl

class GeminiWebSocket:
    def __init__(self, api_key):
        self.api_key = api_key
        self.url = "wss://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent"
    
    def on_message(self, ws, message):
        try:
            data = json.loads(message)
            text = data["candidates"][0]["content"]["parts"][0]["text"]
            print(text, end="", flush=True)
        except:
            pass
    
    def on_error(self, ws, error):
        print(f"Error: {error}")
    
    def on_close(self, ws, close_status_code, close_msg):
        print("\nConnection closed")
    
    def stream(self, prompt):
        headers = {
            "x-goog-api-key": self.api_key
        }
        
        ws = websocket.WebSocketApp(
            self.url,
            header=headers,
            on_message=self.on_message,
            on_error=self.on_error,
            on_close=self.on_close
        )
        
        ws.run_forever(sslopt={"cert_reqs": ssl.CERT_NONE})
```

## Server-Sent Events Parser

```python
def parse_sse_stream(response):
    """Parse Server-Sent Events from response"""
    for line in response.iter_lines():
        if line:
            decoded_line = line.decode('utf-8')
            if decoded_line.startswith('data: '):
                data = decoded_line[6:]  # Remove 'data: ' prefix
                if data == '[DONE]':
                    break
                try:
                    parsed = json.loads(data)
                    yield parsed
                except json.JSONDecodeError:
                    continue

# Usage
response = requests.post(url, headers=headers, json=data, stream=True)
for event in parse_sse_stream(response):
    text = event.get("candidates", [{}])[0].get("content", {}).get("parts", [{}])[0].get("text", "")
    print(text, end="", flush=True)
```

## Error Handling

```bash
# Handle streaming errors
curl -N -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Your prompt"
      }]
    }]
  }' | while IFS= read -r line; do
    if [[ $line == data:* ]]; then
      json_data="${line#data: }"
      if [ "$json_data" != "[DONE]" ]; then
        # Check for errors
        if echo "$json_data" | jq -e '.error' > /dev/null; then
          error_msg=$(echo "$json_data" | jq -r '.error.message')
          echo "Error: $error_msg"
          exit 1
        fi
        
        # Extract text
        text=$(echo "$json_data" | jq -r '.candidates[0].content.parts[0].text' 2>/dev/null)
        if [ "$text" != "null" ]; then
          echo -n "$text"
        fi
      fi
    fi
  done
```

## Rate Limiting

```python
import time
import requests

class RateLimitedStream:
    def __init__(self, api_key, requests_per_minute=60):
        self.api_key = api_key
        self.min_interval = 60.0 / requests_per_minute
        self.last_request = 0
    
    def stream(self, prompt):
        # Rate limiting
        elapsed = time.time() - self.last_request
        if elapsed < self.min_interval:
            time.sleep(self.min_interval - elapsed)
        
        self.last_request = time.time()
        
        # Make streaming request
        # ... implementation
```

## Best Practices

1. **Connection Management**: Handle connection drops
2. **Error Recovery**: Implement retry logic
3. **Rate Limiting**: Respect API limits
4. **Buffering**: Buffer chunks for smoother display
5. **Cleanup**: Close streams properly

## See Also
- [Authentication](authentication.md) - API key setup
- [Rate Limits](rate-limits.md) - Usage limits
- [WebSocket](websocket.md) - Real-time connections