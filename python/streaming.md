# Streaming in Python

*Real-time streaming responses with Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation#streaming
SDK: google-genai (Python)
Verified: 2025-01-14
Models: All Gemini models support streaming
Key Features: Real-time output, reduced latency, better UX
-->

## Quick Reference
- **Method**: `generate_content()` with `stream=True`
- **Returns**: Iterator of response chunks
- **Use cases**: Chat, long outputs, real-time display
- **Supports**: Text, chat, multimodal

## Basic Streaming

```python
from google import genai

client = genai.Client()

# Stream text generation
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a detailed story about space exploration",
    config={"stream": True}
)

# Process chunks as they arrive
for chunk in response:
    print(chunk.text, end="", flush=True)
```

## Streaming with Full Response

```python
# Stream and collect full response
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Explain quantum computing",
    config={"stream": True}
)

full_text = ""
for chunk in response:
    chunk_text = chunk.text
    print(chunk_text, end="", flush=True)
    full_text += chunk_text

print(f"\n\nFull response: {full_text}")
```

## Streaming Chat Class

```python
class StreamingChat:
    def __init__(self):
        self.client = genai.Client()
        self.history = []
    
    def send_message(self, message, on_chunk=None):
        """Send message and stream response"""
        # Add user message to history
        self.history.append({"role": "user", "parts": [{"text": message}]})
        
        # Stream response
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=self.history,
            config={"stream": True}
        )
        
        full_response = ""
        for chunk in response:
            chunk_text = chunk.text
            full_response += chunk_text
            
            # Call callback with each chunk
            if on_chunk:
                on_chunk(chunk_text)
        
        # Add assistant response to history
        self.history.append({"role": "model", "parts": [{"text": full_response}]})
        
        return full_response

# Usage
chat = StreamingChat()

chat.send_message(
    "Tell me about the solar system",
    on_chunk=lambda chunk: print(chunk, end="", flush=True)
)
```

## Streaming with Multimodal Input

```python
import base64

# Stream response for image analysis
with open("diagram.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "image/png",
                "data": image_data
            }
        },
        {"text": "Explain this diagram in detail"}
    ],
    config={"stream": True}
)

for chunk in response:
    print(chunk.text, end="", flush=True)
```

## Advanced Streaming Handler

```python
import time
from typing import Iterator, Dict, Any

class StreamHandler:
    def __init__(self):
        self.client = genai.Client()
    
    def stream_with_metadata(self, config: Dict[str, Any]) -> Iterator[Dict[str, Any]]:
        """Stream with timing and metadata"""
        start_time = time.time()
        first_chunk_time = None
        chunk_count = 0
        total_text = ""
        
        try:
            # Add stream config
            config["config"] = config.get("config", {})
            config["config"]["stream"] = True
            
            response = self.client.models.generate_content(**config)
            
            for chunk in response:
                if first_chunk_time is None:
                    first_chunk_time = time.time() - start_time
                
                chunk_text = chunk.text
                total_text += chunk_text
                chunk_count += 1
                
                # Yield chunk with metadata
                yield {
                    "text": chunk_text,
                    "chunk_index": chunk_count,
                    "timestamp": time.time() - start_time
                }
            
            # Final metadata
            yield {
                "type": "complete",
                "total_text": total_text,
                "metrics": {
                    "first_chunk_latency": first_chunk_time,
                    "total_time": time.time() - start_time,
                    "chunk_count": chunk_count
                }
            }
            
        except Exception as error:
            yield {
                "type": "error",
                "error": str(error),
                "timestamp": time.time() - start_time
            }

# Usage
handler = StreamHandler()

stream = handler.stream_with_metadata({
    "model": "gemini-2.5-flash",
    "contents": "Write a technical article about AI"
})

for event in stream:
    if "text" in event:
        print(event["text"], end="", flush=True)
    elif event.get("type") == "complete":
        print(f"\n\nStreaming metrics: {event['metrics']}")
```

## Streaming with Cancellation

```python
import threading
import time

class CancellableStream:
    def __init__(self):
        self.client = genai.Client()
        self.cancelled = False
    
    def stream(self, config):
        """Stream with cancellation support"""
        config["config"] = config.get("config", {})
        config["config"]["stream"] = True
        
        response = self.client.models.generate_content(**config)
        
        for chunk in response:
            if self.cancelled:
                print("\nStream cancelled by user")
                break
            yield chunk.text
    
    def cancel(self):
        """Cancel the streaming"""
        self.cancelled = True

# Usage with timeout
streamer = CancellableStream()

# Cancel after 5 seconds
timer = threading.Timer(5.0, streamer.cancel)
timer.start()

for chunk in streamer.stream({
    "model": "gemini-2.5-flash",
    "contents": "Write a very long essay"
}):
    print(chunk, end="", flush=True)

timer.cancel()  # Cancel timer if completed
```

## WebSocket-like Streaming

```python
import asyncio
import websockets
import json

class WebSocketStreamer:
    def __init__(self):
        self.client = genai.Client()
    
    async def handle_client(self, websocket, path):
        """Handle WebSocket connections"""
        try:
            async for message in websocket:
                data = json.loads(message)
                prompt = data.get("prompt", "")
                
                # Stream response
                response = self.client.models.generate_content(
                    model="gemini-2.5-flash",
                    contents=prompt,
                    config={"stream": True}
                )
                
                for chunk in response:
                    await websocket.send(json.dumps({
                        "type": "chunk",
                        "text": chunk.text
                    }))
                
                await websocket.send(json.dumps({
                    "type": "complete"
                }))
                
        except websockets.exceptions.ConnectionClosed:
            pass
    
    def start_server(self, host="localhost", port=8765):
        """Start WebSocket server"""
        return websockets.serve(self.handle_client, host, port)

# Usage
streamer = WebSocketStreamer()
server = streamer.start_server()

# Run server
asyncio.get_event_loop().run_until_complete(server)
asyncio.get_event_loop().run_forever()
```

## Progress Tracking

```python
class ProgressStream:
    def __init__(self):
        self.client = genai.Client()
    
    def stream_with_progress(self, config, on_progress=None):
        """Stream with progress estimation"""
        config["config"] = config.get("config", {})
        config["config"]["stream"] = True
        
        response = self.client.models.generate_content(**config)
        
        expected_tokens = config.get("config", {}).get("max_output_tokens", 1000)
        current_tokens = 0
        
        for chunk in response:
            chunk_text = chunk.text
            
            # Estimate tokens (rough approximation)
            current_tokens += len(chunk_text.split())
            
            progress = min((current_tokens / expected_tokens) * 100, 99)
            
            if on_progress:
                on_progress({
                    "text": chunk_text,
                    "progress": round(progress),
                    "estimated_tokens": current_tokens
                })
        
        if on_progress:
            on_progress({"progress": 100, "complete": True})

# Usage
def progress_callback(data):
    if data.get("complete"):
        print(f"\n\nComplete! Generated {data.get('estimated_tokens', 0)} tokens")
    else:
        print(f"\rProgress: {data['progress']}%", end="")

streamer = ProgressStream()
streamer.stream_with_progress({
    "model": "gemini-2.5-flash",
    "contents": "Write a comprehensive guide"
}, on_progress=progress_callback)
```

## Error Handling in Streams

```python
def safe_stream(config):
    """Stream with comprehensive error handling"""
    client = genai.Client()
    
    try:
        config["config"] = config.get("config", {})
        config["config"]["stream"] = True
        
        response = client.models.generate_content(**config)
        
        for chunk in response:
            try:
                yield {"type": "chunk", "text": chunk.text}
            except Exception as chunk_error:
                yield {"type": "error", "error": f"Chunk error: {str(chunk_error)}"}
        
        yield {"type": "complete"}
        
    except Exception as error:
        error_msg = str(error)
        if "SAFETY" in error_msg:
            yield {"type": "safety", "message": "Content filtered for safety"}
        elif "429" in error_msg:
            yield {"type": "rate_limit", "message": "Rate limit exceeded"}
        else:
            yield {"type": "error", "error": error_msg}

# Usage
for event in safe_stream({
    "model": "gemini-2.5-flash",
    "contents": "Your prompt"
}):
    if event["type"] == "chunk":
        print(event["text"], end="", flush=True)
    elif event["type"] == "complete":
        print("\nStreaming complete")
    elif event["type"] == "error":
        print(f"\nError: {event['error']}")
```

## Buffered Streaming

```python
import time
from collections import deque

class BufferedStream:
    def __init__(self, buffer_size=10):
        self.client = genai.Client()
        self.buffer_size = buffer_size
        self.buffer = deque()
    
    def stream_buffered(self, config):
        """Stream with buffering for smoother output"""
        config["config"] = config.get("config", {})
        config["config"]["stream"] = True
        
        response = self.client.models.generate_content(**config)
        
        for chunk in response:
            self.buffer.append(chunk.text)
            
            # Output buffered chunks
            if len(self.buffer) >= self.buffer_size:
                while self.buffer:
                    print(self.buffer.popleft(), end="", flush=True)
                    time.sleep(0.01)  # Small delay for smoother output
        
        # Output remaining buffer
        while self.buffer:
            print(self.buffer.popleft(), end="", flush=True)
            time.sleep(0.01)

# Usage
buffered = BufferedStream(buffer_size=5)
buffered.stream_buffered({
    "model": "gemini-2.5-flash",
    "contents": "Write a story"
})
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
- [Async Processing](async-processing.md) - Asynchronous patterns