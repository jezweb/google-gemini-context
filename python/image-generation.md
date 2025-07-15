# Python Image Generation

*Generate images using Imagen models via Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/imagen
Verified: 2025-01-14
Models: Imagen 3, Imagen 4, Imagen 4 Ultra
Note: Paid tier only, includes SynthID watermark
-->

## Quick Reference
- **Models**: imagen-3.0-generate-002, imagen-4.0-generate-001, imagen-4.0-ultra-generate-001
- **Pricing**: Imagen 3: $0.03, Imagen 4: $0.04 per image
- **Max prompt**: 480 tokens
- **Output**: PNG images with invisible watermark

## Basic Image Generation

```python
from google import genai
from google.genai import types
from PIL import Image
from io import BytesIO

client = genai.Client()

# Generate with Imagen 4 (recommended)
response = client.models.generate_images(
    model="imagen-4.0-generate-001",
    prompt="A serene Japanese garden with cherry blossoms",
    config=types.GenerateImagesConfig(
        number_of_images=1,
    )
)

# Save image
for generated_image in response.generated_images:
    image = Image.open(BytesIO(generated_image.image.image_bytes))
    image.save("japanese_garden.png")
    image.show()
```

## Multiple Images

```python
# Generate multiple variations
response = client.models.generate_images(
    model="imagen-4.0-generate-001",
    prompt="A futuristic cityscape at sunset",
    config=types.GenerateImagesConfig(
        number_of_images=4,  # Max 4 for Imagen 4
    )
)

# Save all images
for i, generated_image in enumerate(response.generated_images):
    image = Image.open(BytesIO(generated_image.image.image_bytes))
    image.save(f"cityscape_{i+1}.png")
```

## Aspect Ratios

```python
# Available aspect ratios
response = client.models.generate_images(
    model="imagen-4.0-generate-001",
    prompt="A majestic mountain landscape",
    config=types.GenerateImagesConfig(
        number_of_images=1,
        aspect_ratio="16:9"  # Widescreen
    )
)

# Aspect ratio options:
# - "1:1" (default) - Square
# - "4:3" - Fullscreen
# - "3:4" - Portrait fullscreen
# - "16:9" - Widescreen
# - "9:16" - Portrait widescreen
```

## Imagen 4 Ultra (High Precision)

```python
# Use Ultra for precise prompt following
response = client.models.generate_images(
    model="imagen-4.0-ultra-generate-001",
    prompt="""A steampunk owl wearing brass goggles and a top hat,
              perched on a Victorian street lamp at twilight,
              intricate mechanical details visible""",
    config=types.GenerateImagesConfig(
        number_of_images=1,  # Ultra only supports 1 image
    )
)

image = Image.open(BytesIO(response.generated_images[0].image.image_bytes))
image.save("steampunk_owl_ultra.png")
```

## Advanced Prompting

```python
# Detailed prompt structure
def generate_with_style(subject, context, style, modifiers):
    prompt = f"{subject} in {context}, {style} style, {', '.join(modifiers)}"
    
    response = client.models.generate_images(
        model="imagen-4.0-generate-001",
        prompt=prompt,
        config=types.GenerateImagesConfig(
            number_of_images=2,
        )
    )
    
    return response

# Example usage
result = generate_with_style(
    subject="a wise wizard",
    context="an ancient library",
    style="oil painting",
    modifiers=["dramatic lighting", "rich colors", "detailed brushwork"]
)
```

## Native Gemini Image Generation

```python
# Gemini 2.5 Flash can also generate images natively
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Create an image of a robot chef cooking in a modern kitchen",
    config=types.GenerateContentConfig(
        response_modalities=["TEXT", "IMAGE"],
    )
)

# Extract generated images
for part in response.candidates[0].content.parts:
    if hasattr(part, 'inline_data') and part.inline_data.mime_type.startswith('image/'):
        image_data = part.inline_data.data
        image = Image.open(BytesIO(base64.b64decode(image_data)))
        image.save("robot_chef.png")
```

## Batch Generation

```python
import asyncio

async def batch_generate(prompts):
    """Generate images for multiple prompts efficiently"""
    tasks = []
    
    async def generate_single(prompt, index):
        response = await client.models.generate_images_async(
            model="imagen-4.0-generate-001",
            prompt=prompt,
            config=types.GenerateImagesConfig(
                number_of_images=1,
            )
        )
        return index, response
    
    for i, prompt in enumerate(prompts):
        tasks.append(generate_single(prompt, i))
    
    results = await asyncio.gather(*tasks)
    return results

# Usage
prompts = [
    "A peaceful zen garden",
    "A bustling market street",
    "A cozy reading nook"
]

results = asyncio.run(batch_generate(prompts))
```

## Error Handling

```python
try:
    response = client.models.generate_images(
        model="imagen-4.0-generate-001",
        prompt="Generate an image",
        config=types.GenerateImagesConfig(
            number_of_images=1,
        )
    )
except Exception as e:
    if "INVALID_ARGUMENT" in str(e):
        print("Invalid prompt or parameters")
    elif "RESOURCE_EXHAUSTED" in str(e):
        print("Rate limit exceeded")
    else:
        print(f"Generation failed: {e}")
```

## Best Practices

### Prompt Engineering
```python
# Good prompt structure
good_prompt = """
A majestic lion, golden mane flowing in the wind,
standing on a rocky outcrop at sunset,
photorealistic style, dramatic lighting,
high detail, professional wildlife photography
"""

# Avoid vague prompts
# bad_prompt = "nice picture"
```

### Model Selection
```python
def choose_model(requirements):
    """Select appropriate model based on needs"""
    if requirements.get("ultra_precision"):
        return "imagen-4.0-ultra-generate-001"
    elif requirements.get("text_rendering"):
        return "imagen-4.0-generate-001"  # Better text
    else:
        return "imagen-3.0-generate-002"  # Lower cost
```

## Limitations
- Paid tier only (no free tier)
- 480 token prompt limit
- No image editing/inpainting
- All images include SynthID watermark
- Ultra model limited to 1 image

## See Also
- [Vision](vision.md) - Image understanding
- [Text Generation](text-generation.md)
- [Client Setup](client-setup.md)