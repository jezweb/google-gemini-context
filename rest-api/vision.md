# REST API Vision (Image Understanding)

*Analyze and understand images using REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision
Verified: 2025-01-14
Models: All Gemini multimodal models
Note: Supports inline base64 and file URIs
-->

## Quick Reference
- **Method**: POST generateContent
- **Formats**: Base64, File API URIs
- **Models**: All Gemini models except Nano
- **Use cases**: Analysis, OCR, understanding

## Inline Base64 Image

```bash
# Convert image to base64
base64_image=$(base64 -w 0 image.jpg)

# Send request
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "What is in this image?"
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "'$base64_image'"
          }
        }
      ]
    }]
  }'
```

## Using File API

```bash
# Step 1: Upload image
upload_response=$(curl -X POST "https://generativelanguage.googleapis.com/upload/v1beta/files?key=$API_KEY" \
  -H "X-Goog-Upload-Protocol: multipart" \
  -F "file=@image.jpg;type=image/jpeg")

# Extract file URI
file_uri=$(echo $upload_response | jq -r '.file.uri')

# Step 2: Use uploaded image
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "Describe this image in detail"
        },
        {
          "fileData": {
            "mimeType": "image/jpeg",
            "fileUri": "'$file_uri'"
          }
        }
      ]
    }]
  }'
```

## Multiple Images

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "Compare these two images"
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "'$base64_image1'"
          }
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "'$base64_image2'"
          }
        }
      ]
    }]
  }'
```

## OCR and Text Extraction

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "Extract all text from this image and format it nicely"
        },
        {
          "inlineData": {
            "mimeType": "image/png",
            "data": "'$base64_document'"
          }
        }
      ]
    }]
  }'
```

## Object Detection

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "List all objects in this image with their approximate locations"
        },
        {
          "fileData": {
            "mimeType": "image/jpeg",
            "fileUri": "'$file_uri'"
          }
        }
      ]
    }],
    "generationConfig": {
      "responseMimeType": "application/json"
    }
  }'
```

## Image Analysis with Schema

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "Analyze this product image"
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "'$base64_product'"
          }
        }
      ]
    }],
    "generationConfig": {
      "responseMimeType": "application/json",
      "responseSchema": {
        "type": "object",
        "properties": {
          "productType": {"type": "string"},
          "brand": {"type": "string"},
          "colors": {
            "type": "array",
            "items": {"type": "string"}
          },
          "description": {"type": "string"},
          "condition": {
            "type": "string",
            "enum": ["new", "used", "damaged"]
          }
        }
      }
    }
  }'
```

## Chart and Graph Analysis

```bash
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "text": "Extract data from this chart and provide the values in a table format"
        },
        {
          "inlineData": {
            "mimeType": "image/png",
            "data": "'$base64_chart'"
          }
        }
      ]
    }]
  }'
```

## Batch Image Processing

```bash
# Upload multiple images first
for img in *.jpg; do
  response=$(curl -X POST "https://generativelanguage.googleapis.com/upload/v1beta/files?key=$API_KEY" \
    -H "X-Goog-Upload-Protocol: multipart" \
    -F "file=@$img;type=image/jpeg")
  echo "$img: $(echo $response | jq -r '.file.uri')"
done

# Process all images
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {"text": "Categorize these images"},
        {"fileData": {"mimeType": "image/jpeg", "fileUri": "file1_uri"}},
        {"fileData": {"mimeType": "image/jpeg", "fileUri": "file2_uri"}},
        {"fileData": {"mimeType": "image/jpeg", "fileUri": "file3_uri"}}
      ]
    }]
  }'
```

## Supported Formats

| Format | MIME Type | Max Size |
|--------|-----------|----------|
| JPEG | image/jpeg | 20MB |
| PNG | image/png | 20MB |
| GIF | image/gif | 20MB |
| WebP | image/webp | 20MB |
| HEIC | image/heic | 20MB |

## Best Practices

### Image Optimization
- Resize large images
- Use appropriate compression
- Remove metadata if not needed
- Consider base64 vs file upload

### Prompting Tips
- Be specific about what to analyze
- Ask for structured output
- Provide context when needed
- Use multi-turn for clarification

### Performance
```bash
# Compress before encoding
convert input.jpg -quality 85 -resize 1024x1024\> compressed.jpg
base64_image=$(base64 -w 0 compressed.jpg)
```

## Common Use Cases

1. **Document Processing**
   - Receipt scanning
   - Form extraction
   - ID verification

2. **E-commerce**
   - Product categorization
   - Visual search
   - Quality inspection

3. **Content Moderation**
   - Image classification
   - Safety checking
   - Compliance verification

## Error Handling

```bash
# Check for image errors
response=$(curl -s -X POST ...)
if [[ $response == *"INVALID_ARGUMENT"* ]]; then
  echo "Image format not supported"
fi
```

## Limitations
- 20MB max file size
- No image generation
- Processing time varies
- Token limits apply

## See Also
- [Multipart Upload](multipart.md)
- [File Handling](file-handling.md)
- [Video Understanding](video.md)