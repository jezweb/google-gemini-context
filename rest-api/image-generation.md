# REST API Image Generation

*Generate images using Imagen models via REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/imagen
Verified: 2025-01-14
Models: Imagen 3, Imagen 4, Imagen 4 Ultra
Note: Paid tier only, includes SynthID watermark
-->

## Quick Reference
- **Endpoint**: /models/{model}:generateImages
- **Models**: imagen-3.0-generate-002, imagen-4.0-generate-001, imagen-4.0-ultra-generate-001
- **Pricing**: Imagen 3: $0.03, Imagen 4: $0.04 per image
- **Auth**: API key required (paid tier)

## Basic Image Generation

```bash
# Generate with Imagen 4 (recommended)
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A serene Japanese garden with cherry blossoms",
    "numberOfImages": 1
  }'

# Response includes base64-encoded images
# {
#   "generatedImages": [{
#     "image": {
#       "imageBytes": "iVBORw0KGgoAAAANS..."
#     }
#   }]
# }
```

## Save Generated Image

```bash
# Generate and save image
response=$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A futuristic cityscape at sunset",
    "numberOfImages": 1
  }')

# Extract and decode image
echo $response | jq -r '.generatedImages[0].image.imageBytes' | base64 -d > cityscape.png
```

## Multiple Images

```bash
# Generate multiple variations (max 4 for Imagen 4)
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Abstract geometric patterns in vibrant colors",
    "numberOfImages": 4
  }' | jq -r '.generatedImages[].image.imageBytes' | while IFS= read -r img; do
    echo "$img" | base64 -d > "pattern_$(date +%s%N).png"
    sleep 0.1
  done
```

## Aspect Ratios

```bash
# Generate with specific aspect ratio
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A majestic mountain landscape",
    "numberOfImages": 1,
    "aspectRatio": "16:9"
  }'

# Available aspect ratios:
# - "1:1" (default) - Square
# - "4:3" - Fullscreen
# - "3:4" - Portrait fullscreen
# - "16:9" - Widescreen
# - "9:16" - Portrait widescreen
```

## Imagen 4 Ultra (High Precision)

```bash
# Use Ultra model for precise prompt following
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-ultra-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A steampunk owl wearing brass goggles and a top hat, perched on a Victorian street lamp at twilight, intricate mechanical details visible",
    "numberOfImages": 1
  }' | jq -r '.generatedImages[0].image.imageBytes' | base64 -d > steampunk_owl_ultra.png
```

## Native Gemini Image Generation

```bash
# Gemini 2.5 Flash can also generate images natively
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Create an image of a robot chef cooking in a modern kitchen"
      }]
    }],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"]
    }
  }'

# Response includes inline image data in parts
```

## Batch Script

```bash
#!/bin/bash
# batch_generate.sh - Generate images from prompts file

API_KEY="your-api-key"
MODEL="imagen-4.0-generate-001"

while IFS= read -r prompt; do
  echo "Generating: $prompt"
  
  response=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/$MODEL:generateImages?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"prompt\": \"$prompt\",
      \"numberOfImages\": 1
    }")
  
  # Save with sanitized filename
  filename=$(echo "$prompt" | tr ' ' '_' | tr -d '/?<>\\:*|"' | cut -c1-50)
  echo "$response" | jq -r '.generatedImages[0].image.imageBytes' | base64 -d > "${filename}.png"
  
  sleep 2  # Rate limiting
done < prompts.txt
```

## Advanced Request

```bash
# Complex prompt with all parameters
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A majestic lion, golden mane flowing in the wind, standing on a rocky outcrop at sunset, photorealistic style, dramatic lighting, high detail, professional wildlife photography",
    "numberOfImages": 2,
    "aspectRatio": "16:9",
    "negativePrompt": "cartoon, illustration, low quality, blurry",
    "personGeneration": "dont_allow"
  }'
```

## Error Handling

```bash
# Function to handle API responses
generate_image() {
  local prompt="$1"
  local output="$2"
  
  response=$(curl -s -w "\n%{http_code}" -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:generateImages?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{\"prompt\": \"$prompt\", \"numberOfImages\": 1}")
  
  http_code=$(echo "$response" | tail -n1)
  body=$(echo "$response" | sed '$d')
  
  if [ "$http_code" -eq 200 ]; then
    echo "$body" | jq -r '.generatedImages[0].image.imageBytes' | base64 -d > "$output"
    echo "Success: Image saved to $output"
  else
    error=$(echo "$body" | jq -r '.error.message // "Unknown error"')
    echo "Error ($http_code): $error"
    return 1
  fi
}

# Usage
generate_image "A peaceful zen garden" "zen_garden.png"
```

## Model Comparison

```bash
# Compare outputs from different models
models=("imagen-3.0-generate-002" "imagen-4.0-generate-001")
prompt="A detailed steampunk clockwork mechanism"

for model in "${models[@]}"; do
  echo "Generating with $model..."
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/$model:generateImages?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{\"prompt\": \"$prompt\", \"numberOfImages\": 1}" | \
    jq -r '.generatedImages[0].image.imageBytes' | \
    base64 -d > "${model}_output.png"
done
```

## Response Format

```json
{
  "generatedImages": [
    {
      "image": {
        "imageBytes": "base64-encoded-png-data"
      }
    }
  ]
}
```

## Common Errors

| Code | Error | Solution |
|------|-------|----------|
| 403 | PERMISSION_DENIED | Paid tier required |
| 400 | INVALID_ARGUMENT | Check prompt/parameters |
| 429 | RESOURCE_EXHAUSTED | Rate limit hit |
| 500 | INTERNAL | Retry request |

## Best Practices

### Prompt Structure
```bash
# Good prompt with details
prompt="A majestic eagle soaring above snow-capped mountains, \
golden hour lighting, ultra-realistic, professional photography, \
8k resolution, shallow depth of field"

# Avoid vague prompts
# bad_prompt="bird picture"
```

### Rate Limiting
```bash
# Add delays between requests
for i in {1..5}; do
  generate_image "Landscape $i" "landscape_$i.png"
  sleep 3
done
```

## Limitations
- Paid tier only (no free tier)
- 480 token prompt limit
- No image editing/inpainting
- All images include SynthID watermark
- Ultra model limited to 1 image

## See Also
- [Vision](vision.md) - Image understanding
- [Endpoints](endpoints.md) - API reference
- [Authentication](authentication.md)