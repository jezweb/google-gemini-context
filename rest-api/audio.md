# Audio in REST API

*Audio understanding with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/audio
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Transcription, understanding, multimodal with audio
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/{model}:generateContent`
- **Input formats**: MP3, WAV, FLAC, AAC, OGG, OPUS
- **Max size**: 20MB
- **Encoding**: Base64 in JSON

## Basic Audio Request

```bash
# Encode audio file to base64
AUDIO_BASE64=$(base64 -i speech.mp3)

# Send request
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"contents\": [
      {
        \"parts\": [
          {
            \"inline_data\": {
              \"mime_type\": \"audio/mp3\",
              \"data\": \"$AUDIO_BASE64\"
            }
          },
          {
            \"text\": \"Transcribe this audio\"
          }
        ]
      }
    ]
  }"
```

## Audio Transcription

```json
{
  "contents": [
    {
      "parts": [
        {
          "inline_data": {
            "mime_type": "audio/wav",
            "data": "BASE64_ENCODED_AUDIO"
          }
        },
        {
          "text": "Transcribe this audio verbatim"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.1,
    "maxOutputTokens": 8192
  }
}
```

## Audio Analysis

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "inline_data": {
            "mime_type": "audio/mp3",
            "data": "'$AUDIO_BASE64'"
          }
        },
        {
          "text": "Analyze this audio for: speakers, emotions, topics, and quality"
        }
      ]
    }]
  }'
```

## Multiple Audio Files

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Compare these two audio recordings"
        },
        {
          "inline_data": {
            "mime_type": "audio/mp3",
            "data": "FIRST_AUDIO_BASE64"
          }
        },
        {
          "inline_data": {
            "mime_type": "audio/mp3",
            "data": "SECOND_AUDIO_BASE64"
          }
        }
      ]
    }
  ]
}
```

## Audio with Image

```json
{
  "contents": [
    {
      "parts": [
        {
          "inline_data": {
            "mime_type": "audio/mp3",
            "data": "AUDIO_BASE64"
          }
        },
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "IMAGE_BASE64"
          }
        },
        {
          "text": "How does the audio relate to this image?"
        }
      ]
    }
  ]
}
```

## Shell Script for Audio Processing

```bash
#!/bin/bash

# audio_to_gemini.sh
# Usage: ./audio_to_gemini.sh audio.mp3 "Your prompt"

AUDIO_FILE="$1"
PROMPT="$2"
API_KEY="$GEMINI_API_KEY"

# Check file exists
if [ ! -f "$AUDIO_FILE" ]; then
    echo "Error: Audio file not found"
    exit 1
fi

# Get MIME type
MIME_TYPE="audio/mp3"
case "${AUDIO_FILE##*.}" in
    wav) MIME_TYPE="audio/wav" ;;
    flac) MIME_TYPE="audio/flac" ;;
    aac) MIME_TYPE="audio/aac" ;;
    ogg) MIME_TYPE="audio/ogg" ;;
    opus) MIME_TYPE="audio/opus" ;;
esac

# Encode to base64
AUDIO_BASE64=$(base64 -i "$AUDIO_FILE" | tr -d '\n')

# Create request
REQUEST_JSON=$(cat <<EOF
{
  "contents": [{
    "parts": [
      {
        "inline_data": {
          "mime_type": "$MIME_TYPE",
          "data": "$AUDIO_BASE64"
        }
      },
      {
        "text": "$PROMPT"
      }
    ]
  }]
}
EOF
)

# Send request
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d "$REQUEST_JSON" | jq -r '.candidates[0].content.parts[0].text'
```

## Python Example

```python
import requests
import base64
import json

def transcribe_audio(audio_path, prompt="Transcribe this audio"):
    # Read and encode audio
    with open(audio_path, "rb") as f:
        audio_base64 = base64.b64encode(f.read()).decode()
    
    # Determine MIME type
    mime_types = {
        '.mp3': 'audio/mp3',
        '.wav': 'audio/wav',
        '.flac': 'audio/flac'
    }
    ext = audio_path.lower().split('.')[-1]
    mime_type = mime_types.get(f'.{ext}', 'audio/mp3')
    
    # Prepare request
    url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"
    headers = {
        "x-goog-api-key": os.environ["GEMINI_API_KEY"],
        "Content-Type": "application/json"
    }
    
    data = {
        "contents": [{
            "parts": [
                {
                    "inline_data": {
                        "mime_type": mime_type,
                        "data": audio_base64
                    }
                },
                {
                    "text": prompt
                }
            ]
        }]
    }
    
    # Send request
    response = requests.post(url, headers=headers, json=data)
    result = response.json()
    
    return result['candidates'][0]['content']['parts'][0]['text']
```

## Common Use Cases

### Meeting Notes
```json
{
  "contents": [{
    "parts": [
      {
        "inline_data": {
          "mime_type": "audio/mp3",
          "data": "MEETING_AUDIO_BASE64"
        }
      },
      {
        "text": "Extract meeting notes with: attendees, action items, decisions, and next steps"
      }
    ]
  }],
  "generationConfig": {
    "temperature": 0.3,
    "maxOutputTokens": 4096
  }
}
```

### Language Detection
```json
{
  "contents": [{
    "parts": [
      {
        "inline_data": {
          "mime_type": "audio/mp3",
          "data": "AUDIO_BASE64"
        }
      },
      {
        "text": "What language is being spoken?"
      }
    ]
  }]
}
```

## Response Format

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Transcription: Hello, this is a test audio file..."
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP"
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 2500,
    "candidatesTokenCount": 150,
    "totalTokenCount": 2650
  }
}
```

## Error Handling

```bash
# Check response for errors
RESPONSE=$(curl -s -X POST ... )

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    echo "Error: $ERROR_MSG"
    exit 1
fi

# Extract text
TEXT=$(echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text')
echo "$TEXT"
```

## Best Practices

1. **Compression**: Pre-compress large audio files
2. **Chunking**: Split very long audio into segments
3. **Error Handling**: Check for size/format errors
4. **Caching**: Store transcriptions to avoid re-processing
5. **Batch Processing**: Use scripts for multiple files

## See Also
- [Vision](vision.md) - Image processing
- [Video](video.md) - Video with audio
- [Multipart](multipart.md) - File uploads