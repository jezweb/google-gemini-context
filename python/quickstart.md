# Python Quickstart

*Get started with Gemini API in Python in 5 minutes*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/quickstart?lang=python
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New SDK package, Client() initialization
-->

## Quick Reference
- **Package**: `google-genai`
- **Python**: 3.9+ required
- **Async**: Full async/await support
- **Type hints**: Comprehensive typing

## Installation

```bash
pip install -q -U google-genai
```

## Basic Setup

```python
from google import genai

# Initialize client (API key from GEMINI_API_KEY env var)
client = genai.Client()

# Basic generation
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Explain quantum computing in one paragraph"
)
print(response.text)
```

## First Request

```python
from google import genai
import os

# Client automatically uses GEMINI_API_KEY env var
client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Explain quantum computing in one paragraph"
)
print(response.text)
```

## Environment Setup

```bash
# .env file
GEMINI_API_KEY=your-api-key-here

# Load with python-dotenv
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os

load_dotenv()
genai.configure(api_key=os.getenv("GEMINI_API_KEY"))
```

## Async Support

```python
import asyncio
from google import genai

async def generate_async():
    client = genai.Client()
    response = await client.models.generate_content_async(
        model="gemini-2.5-flash",
        contents="Write a haiku about programming"
    )
    return response.text

# Run async
async def main():
    result = await generate_async()
    print(result)

asyncio.run(main())
```

## Streaming Response

```python
client = genai.Client()
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a story",
    config={"stream": True}
)

for chunk in response:
    print(chunk.text, end='')
```

## Error Handling

```python
try:
    response = model.generate_content(prompt)
    print(response.text)
except Exception as e:
    if "SAFETY" in str(e):
        print("Content blocked for safety reasons")
    elif "429" in str(e):
        print("Rate limit exceeded, please retry later")
    else:
        print(f"Error: {e}")
```

## Complete Example

```python
from google import genai
from google.genai import types
from dotenv import load_dotenv
import os

def main():
    # Load environment variables
    load_dotenv()
    
    # Initialize client
    client = genai.Client()
    
    # Generate with configuration
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents="Hello! How are you today?",
        config=types.GenerateContentConfig(
            temperature=0.9,
            top_p=0.95,
            top_k=40,
            max_output_tokens=2048,
            thinking_config=types.ThinkingConfig(thinking_budget=0)
        )
    )
    
    print(response.text)
    
    # For chat conversations
    chat = client.chats.create(model="gemini-2.5-flash")
    response = chat.send_message("Can you help me learn Python?")
    print(response.text)

if __name__ == "__main__":
    main()
```

## Type Hints Example

```python
from typing import Optional, List
from google import genai
from google.genai import types

def generate_text(
    prompt: str,
    model_name: str = "gemini-2.5-flash",
    temperature: float = 0.7,
    max_tokens: int = 2048
) -> Optional[str]:
    """Generate text using Gemini API with type safety."""
    
    try:
        client = genai.Client()
        response = client.models.generate_content(
            model=model_name,
            contents=prompt,
            config=types.GenerateContentConfig(
                temperature=temperature,
                max_output_tokens=max_tokens
            )
        )
        return response.text
    
    except Exception as e:
        print(f"Generation failed: {e}")
        return None
```

## Requirements.txt

```text
google-genai
python-dotenv>=1.0.0
```

## Next Steps
- [Client Setup](client-setup.md) - Advanced configuration
- [Text Generation](text-generation.md) - Generation options
- [Chat](chat.md) - Multi-turn conversations

## Common Issues

| Problem | Solution |
|---------|----------|
| `ModuleNotFoundError` | Install with `pip install google-generativeai` |
| `API key not valid` | Check environment variable |
| `AttributeError` | Update to latest SDK version |
| `Rate limit` | Implement exponential backoff |

## Virtual Environment Setup

```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install dependencies
pip install google-generativeai python-dotenv
```

## See Also
- [Official SDK Docs](https://ai.google.dev/tutorials/python_quickstart)
- [Async Operations](async.md)
- [Error Handling](error-handling.md)