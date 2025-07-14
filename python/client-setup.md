# Python Client Setup

*Advanced configuration for google-genai SDK*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/get-started/python
SDK: google-genai (Python)
Verified: 2025-01-14
Key Changes: New Client() initialization, config per request
-->

## Quick Reference
- **Regional endpoints**: Data residency control
- **Proxy support**: Corporate proxy configuration
- **Retry logic**: Built-in exponential backoff
- **Transport options**: Custom HTTP client

## Basic Configuration

```python
from google import genai

# Simple configuration (uses GEMINI_API_KEY env var)
client = genai.Client()

# With explicit API key
client = genai.Client(api_key="your-api-key")

# With client options
client = genai.Client(
    client_options={
        'api_endpoint': 'europe-west4-generativelanguage.googleapis.com'
    }
)
```

## Generation Configuration

```python
# Configuration is passed with each request
config = genai.GenerateContentConfig(
    temperature=0.9,
    top_p=0.95,
    top_k=40,
    max_output_tokens=8192,
    stop_sequences=['END'],
    system_instruction="You are a helpful coding assistant."
)

safety_settings = [
    {
        "category": "HARM_CATEGORY_HATE_SPEECH",
        "threshold": "BLOCK_LOW_AND_ABOVE"
    },
    {
        "category": "HARM_CATEGORY_HARASSMENT",
        "threshold": "BLOCK_MEDIUM_AND_ABOVE"
    }
]

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Your prompt",
    config=config,
    safety_settings=safety_settings
)
```

## Regional Endpoints

```python
# Europe endpoint
client_eu = genai.Client(
    client_options={
        'api_endpoint': 'europe-west4-generativelanguage.googleapis.com'
    }
)

# US endpoint
client_us = genai.Client(
    client_options={
        'api_endpoint': 'us-central1-generativelanguage.googleapis.com'
    }
)

# Asia endpoint
client_asia = genai.Client(
    client_options={
        'api_endpoint': 'asia-northeast1-generativelanguage.googleapis.com'
    }
)
```

## Proxy Configuration

```python
from google import genai
import httpx

# Configure proxy
proxies = {
    "http://": "http://proxy.company.com:8080",
    "https://": "http://proxy.company.com:8080"
}

# Create custom transport
transport = httpx.HTTPTransport(proxy=proxies["https://"])
httpx_client = httpx.Client(transport=transport)

# Create Gemini client with custom transport
client = genai.Client(
    transport=httpx_client
)
```

## Custom Headers

```python
from google import genai

# Custom headers can be set via client options
client = genai.Client(
    client_options={
        'api_endpoint': 'generativelanguage.googleapis.com',
        'headers': {
            'x-custom-header': 'value',
            'user-agent': 'MyApp/1.0'
        }
    }
)
```

## Retry Configuration

```python
import time
from typing import Optional

class RetryClient:
    def __init__(self, max_retries: int = 3):
        self.client = genai.Client()
        self.max_retries = max_retries
    
    def generate_with_retry(
        self, 
        model: str, 
        prompt: str,
        config: Optional[genai.GenerateContentConfig] = None
    ):
        for i in range(self.max_retries):
            try:
                return self.client.models.generate_content(
                    model=model,
                    contents=prompt,
                    config=config
                )
            except Exception as e:
                if i == self.max_retries - 1:
                    raise
                wait_time = min(2 ** i, 60)
                time.sleep(wait_time)
```

## Timeout Configuration

```python
import asyncio
from google import genai

# Per-request timeout using asyncio
async def generate_with_timeout(client, prompt, timeout_seconds=30):
    try:
        response = await asyncio.wait_for(
            client.aio.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt
            ),
            timeout=timeout_seconds
        )
        return response
    except asyncio.TimeoutError:
        raise TimeoutError(f"Request timed out after {timeout_seconds} seconds")
```

## Environment-based Config

```python
import os
from dataclasses import dataclass
from typing import Dict, Any

@dataclass
class ModelConfig:
    model_name: str
    temperature: float
    max_output_tokens: int
    safety_settings: Dict[str, str]

class ConfigManager:
    CONFIGS = {
        'development': ModelConfig(
            model_name='gemini-2.5-flash',
            temperature=1.0,
            max_output_tokens=2048,
            safety_settings={'HARM_CATEGORY_HARASSMENT': 'BLOCK_NONE'}
        ),
        'production': ModelConfig(
            model_name='gemini-2.5-flash',
            temperature=0.7,
            max_output_tokens=4096,
            safety_settings={'HARM_CATEGORY_HARASSMENT': 'BLOCK_MEDIUM_AND_ABOVE'}
        )
    }
    
    @classmethod
    def get_config(cls) -> ModelConfig:
        env = os.getenv('ENVIRONMENT', 'development')
        return cls.CONFIGS[env]

# Usage
config = ConfigManager.get_config()
client = genai.Client()

response = client.models.generate_content(
    model=config.model_name,
    contents="Your prompt",
    config=genai.GenerateContentConfig(
        temperature=config.temperature,
        max_output_tokens=config.max_output_tokens
    ),
    safety_settings=[{"category": k, "threshold": v} for k, v in config.safety_settings.items()]
)
```

## Connection Pooling

```python
import httpx
from google import genai

# Create connection pool
transport = httpx.HTTPTransport(
    limits=httpx.Limits(
        max_keepalive_connections=10,
        max_connections=20,
        keepalive_expiry=30.0
    )
)

httpx_client = httpx.Client(transport=transport)
client = genai.Client(transport=httpx_client)
```

## Configuration Factory Pattern

```python
from typing import Optional, Dict, Any
from google import genai

class ConfigFactory:
    _configs: Dict[str, genai.GenerateContentConfig] = {}
    
    def __init__(self):
        self.client = genai.Client()
    
    def get_config(
        self,
        name: str = 'default',
        **kwargs
    ) -> genai.GenerateContentConfig:
        if name not in self._configs:
            self._configs[name] = genai.GenerateContentConfig(**kwargs)
        return self._configs[name]
    
    def generate(
        self,
        model: str,
        prompt: str,
        config_name: str = 'default'
    ):
        return self.client.models.generate_content(
            model=model,
            contents=prompt,
            config=self.get_config(config_name)
        )

# Usage
factory = ConfigFactory()
factory.get_config('creative', temperature=1.0, max_output_tokens=2048)
response = factory.generate("gemini-2.5-flash", "Write a story", 'creative')
```

## Logging Configuration

```python
import logging
from google import genai

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)
logging.getLogger('google.genai').setLevel(logging.DEBUG)

# Custom logger
logger = logging.getLogger(__name__)

class LoggingClient:
    def __init__(self):
        self.client = genai.Client()
        
    def generate_content(self, model: str, prompt: str) -> str:
        logger.info(f"Generating content for prompt: {prompt[:50]}...")
        
        try:
            response = self.client.models.generate_content(
                model=model,
                contents=prompt
            )
            logger.info(f"Generated {len(response.text)} characters")
            return response.text
        except Exception as e:
            logger.error(f"Generation failed: {e}")
            raise
```

## OAuth Configuration

```python
from google.oauth2 import service_account
from google import genai

# Service account authentication
credentials = service_account.Credentials.from_service_account_file(
    'path/to/service-account-key.json',
    scopes=['https://www.googleapis.com/auth/generative-language']
)

client = genai.Client(credentials=credentials)
```

## See Also
- [Quickstart](quickstart.md)
- [Authentication](../common/authentication.md)
- [Error Handling](error-handling.md)