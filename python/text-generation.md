# Text Generation in Python

*Generate text completions with Gemini models*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/text-generation
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New SDK package, Client() initialization
-->

## Quick Reference
- **Method**: `client.models.generate_content()` for single response
- **Streaming**: `client.models.generate_content_stream()` for streaming
- **Async**: `client.aio` for async operations
- **Batch**: Process multiple prompts efficiently

## Basic Generation

```python
from google import genai

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a haiku about coding"
)
print(response.text)
```

## Generation Parameters

```python
config = genai.GenerateContentConfig(
    temperature=0.9,           # Creativity (0.0-2.0)
    top_p=0.95,               # Nucleus sampling
    top_k=40,                 # Top K sampling  
    max_output_tokens=2048,    # Max response length
    stop_sequences=['END'],    # Stop triggers
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=prompt,
    config=config
)
```

## System Instructions

```python
config = genai.GenerateContentConfig(
    system_instruction="You are a helpful coding assistant. Always provide code examples."
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="How do I sort a list in Python?",
    config=config
)
print(response.text)
```

## Streaming Responses

```python
for chunk in client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents="Explain machine learning step by step"
):
    print(chunk.text, end='', flush=True)

# Note: Token counting may need separate request
# or accumulation during streaming
```

## Multiple Candidates

```python
# Note: candidate_count may have limitations
config = genai.GenerateContentConfig(
    candidate_count=3,  # Generate 3 variations
    temperature=0.8,
)

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a tagline for a coffee shop",
    config=config
)

# Access candidates if available
if hasattr(response, 'candidates'):
    for i, candidate in enumerate(response.candidates):
        print(f"Option {i+1}: {candidate.text}")
```

## Token Counting

```python
# Count tokens before generating
count = client.models.count_tokens(
    model="gemini-2.5-flash",
    contents="Your prompt here"
)
print(f"Input tokens: {count.total_tokens}")

# Get usage after generation
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Your prompt"
)
# Usage metadata may be in response attributes
if hasattr(response, 'usage_metadata'):
    print(f"Total tokens: {response.usage_metadata.total_token_count}")
```

## Async Generation

```python
import asyncio
from google import genai

async def generate_async(client, prompt: str) -> str:
    response = await client.aio.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt
    )
    return response.text

# Run multiple async requests
async def main():
    client = genai.Client()
    prompts = [
        "Write a function to reverse a string",
        "Explain recursion with an example",
        "What is a decorator in Python?"
    ]
    
    tasks = [generate_async(client, p) for p in prompts]
    results = await asyncio.gather(*tasks)
    
    for prompt, result in zip(prompts, results):
        print(f"Prompt: {prompt}")
        print(f"Response: {result[:100]}...\n")

asyncio.run(main())
```

## Advanced Generator Class

```python
from typing import Optional, List, Dict, Any
from google import genai

class TextGenerator:
    def __init__(self, model_name: str = 'gemini-2.5-flash'):
        self.client = genai.Client()
        self.model_name = model_name
    
    def generate(
        self,
        prompt: str,
        temperature: float = 0.7,
        max_tokens: int = 2048,
        stream: bool = False,
        **kwargs
    ) -> Dict[str, Any]:
        """Generate text with options."""
        config = {
            'temperature': temperature,
            'max_output_tokens': max_tokens,
            **kwargs
        }
        
        try:
            if stream:
                stream_response = self.client.models.generate_content_stream(
                    model=self.model_name,
                    contents=prompt,
                    config=genai.GenerateContentConfig(**config)
                )
                return {'stream': stream_response}
            
            response = self.client.models.generate_content(
                model=self.model_name,
                contents=prompt,
                config=genai.GenerateContentConfig(**config)
            )
            
            result = {
                'text': response.text,
                'safe': True
            }
            
            # Add usage metadata if available
            if hasattr(response, 'usage_metadata'):
                result['usage'] = {
                    'total_tokens': response.usage_metadata.total_token_count
                }
            
            return result
            
        except Exception as e:
            if 'SAFETY' in str(e):
                return {'text': 'Content filtered for safety', 'safe': False}
            raise
    
    def batch_generate(
        self,
        prompts: List[str],
        **kwargs
    ) -> List[Dict[str, Any]]:
        """Generate for multiple prompts."""
        return [self.generate(p, **kwargs) for p in prompts]

# Usage
generator = TextGenerator()
result = generator.generate(
    "Write a Python function to find prime numbers",
    temperature=0.3,
    max_tokens=500
)
print(result['text'])
```

## Common Patterns

### Retry with Backoff
```python
import time
from typing import Optional

def generate_with_retry(
    client: genai.Client,
    prompt: str,
    max_retries: int = 3
) -> Optional[str]:
    for i in range(max_retries):
        try:
            response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt
            )
            return response.text
        except Exception as e:
            if i == max_retries - 1:
                raise
            wait_time = 2 ** i
            print(f"Retry {i+1} after {wait_time}s...")
            time.sleep(wait_time)
```

### Template System
```python
from string import Template

class PromptTemplate:
    def __init__(self, template: str):
        self.template = Template(template)
    
    def format(self, **kwargs) -> str:
        return self.template.substitute(**kwargs)

# Usage
template = PromptTemplate(
    "Write a $style story about $topic in $words words"
)

prompt = template.format(
    style="funny",
    topic="a programmer's first bug",
    words=100
)
```

### Safety Handling
```python
def safe_generate(client, prompt, fallback="Content unavailable"):
    try:
        response = client.models.generate_content(
            model="gemini-2.5-flash",
            contents=prompt
        )
        return response.text
    except Exception as e:
        if any(word in str(e) for word in ['SAFETY', 'BLOCKED']):
            return fallback
        raise
```

## See Also
- [Chat](chat.md) - Multi-turn conversations
- [Async](async.md) - Async operations
- [Structured Output](structured-output.md) - JSON responses