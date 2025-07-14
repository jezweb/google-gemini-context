# Vision in Python

*Image understanding and generation with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New client pattern, updated image handling
-->

## Quick Reference
- **Image Input**: Base64 data, file path, or PIL Image
- **Max Size**: 20MB per image
- **Formats**: JPEG, PNG, WebP, HEIC, HEIF
- **Multiple Images**: Supported in single request

## Basic Image Understanding

```python
from google import genai
import base64

client = genai.Client()

# Read image as base64
with open("image.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

# Create image part
image_part = {
    "inline_data": {
        "mime_type": "image/jpeg",
        "data": image_data
    }
}

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[image_part, {"text": "Describe this image in detail"}]
)

print(response.text)
```

## Using PIL Images

```python
from PIL import Image

# Open image with PIL
img = Image.open("photo.jpg")

# Gemini accepts PIL images directly
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[img, {"text": "What's in this image?"}]
)

print(response.text)
```

## Multiple Images

```python
# Compare multiple images
with open("image1.jpg", "rb") as f:
    image1_data = base64.b64encode(f.read()).decode()
    
with open("image2.jpg", "rb") as f:
    image2_data = base64.b64encode(f.read()).decode()

contents = [
    {"text": "Compare these two images"},
    {
        "inline_data": {
            "mime_type": "image/jpeg",
            "data": image1_data
        }
    },
    {
        "inline_data": {
            "mime_type": "image/jpeg",
            "data": image2_data
        }
    }
]

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=contents
)

print(response.text)
```

## Image from URL

```python
import requests
from PIL import Image
from io import BytesIO

def analyze_image_from_url(url: str, prompt: str):
    # Download image
    response = requests.get(url)
    img = Image.open(BytesIO(response.content))
    
    # Analyze with Gemini
    result = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[img, {"text": prompt}]
    )
    
    return result.text

# Usage
analysis = analyze_image_from_url(
    "https://example.com/image.jpg",
    "What objects are in this image?"
)
```

## Image Processing Utilities

```python
from typing import Union, List
from pathlib import Path
import base64

class ImageProcessor:
    def __init__(self):
        self.client = genai.Client()
    
    def process_image(self, image_path: Union[str, Path]) -> dict:
        """Convert image file to Gemini format"""
        path = Path(image_path)
        mime_type = {
            '.jpg': 'image/jpeg',
            '.jpeg': 'image/jpeg',
            '.png': 'image/png',
            '.webp': 'image/webp',
            '.heic': 'image/heic',
            '.heif': 'image/heif'
        }.get(path.suffix.lower(), 'image/jpeg')
        
        with open(path, "rb") as f:
            data = base64.b64encode(f.read()).decode()
        
        return {
            "inline_data": {
                "mime_type": mime_type,
                "data": data
            }
        }
    
    def analyze_image(self, image_path: str, prompt: str) -> str:
        """Analyze single image"""
        image_part = self.process_image(image_path)
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[image_part, {"text": prompt}]
        )
        return response.text
    
    def compare_images(self, paths: List[str], prompt: str = "Compare these images") -> str:
        """Compare multiple images"""
        contents = [{"text": prompt}]
        for path in paths:
            contents.append(self.process_image(path))
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=contents
        )
        return response.text
    
    def extract_text(self, image_path: str) -> str:
        """OCR text extraction"""
        image_part = self.process_image(image_path)
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                image_part,
                {"text": "Extract all text from this image. Format as plain text."}
            ]
        )
        return response.text
```

## Streaming with Images

```python
# Stream response for image analysis
image = Image.open("complex-diagram.png")

for chunk in client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents=[
        image,
        {"text": "Explain each component in this diagram"}
    ]
):
    print(chunk.text, end='', flush=True)
```

## Common Use Cases

### OCR/Text Extraction
```python
with open("document.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "image/jpeg",
                "data": image_data
            }
        },
        {"text": "Extract all text, maintaining the original formatting"}
    ]
)
print(response.text)
```

### Image Classification
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        image_part,
        {"text": "Classify this image: nature, urban, people, animals, food, or other"}
    ]
)
```

### Visual Q&A
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        image_part,
        {"text": "Is there a red car in this image? Answer yes or no."}
    ]
)
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[image_part, {"text": prompt}]
    )
    return response.text
except Exception as e:
    if "File size exceeds maximum" in str(e):
        print("Image too large, please reduce size")
    elif "Unsupported file type" in str(e):
        print("Please use JPEG, PNG, WebP, HEIC, or HEIF")
    else:
        raise
```

## See Also
- [Audio](audio.md) - Audio processing
- [Video](video.md) - Video understanding
- [Text Generation](text-generation.md) - Basic generation