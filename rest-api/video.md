# Video in REST API

*Video understanding with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision#video
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Video analysis via File API
-->

## Quick Reference
- **Upload endpoint**: `/upload/v1beta/files`
- **Content endpoint**: `/v1beta/models/{model}:generateContent`
- **Max size**: 2GB
- **Max duration**: 1 hour
- **Formats**: MP4, AVI, MOV, WebM, MPEG, 3GPP

## Upload Video File

```bash
# Step 1: Upload video
curl -X POST "https://generativelanguage.googleapis.com/upload/v1beta/files" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "X-Goog-Upload-Command: start" \
  -H "X-Goog-Upload-Header-Content-Length: $(stat -f%z video.mp4)" \
  -H "X-Goog-Upload-Header-Content-Type: video/mp4" \
  -H "Content-Type: application/json" \
  -d '{"file": {"display_name": "My Video"}}'

# Returns upload URL in X-Goog-Upload-URL header
UPLOAD_URL="<from-response-header>"

# Step 2: Upload video data
curl -X POST "$UPLOAD_URL" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Length: $(stat -f%z video.mp4)" \
  -H "X-Goog-Upload-Offset: 0" \
  -H "X-Goog-Upload-Command: upload, finalize" \
  --data-binary @video.mp4
```

## Check Video Status

```bash
# Get file name from upload response
FILE_NAME="files/abc123"

# Check processing status
curl "https://generativelanguage.googleapis.com/v1beta/$FILE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"

# Response includes state: PROCESSING, ACTIVE, or FAILED
```

## Analyze Video

```bash
# Use uploaded video
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "file_data": {
            "file_uri": "https://generativelanguage.googleapis.com/v1beta/files/abc123",
            "mime_type": "video/mp4"
          }
        },
        {
          "text": "Analyze this video in detail"
        }
      ]
    }]
  }'
```

## Complete Upload Script

```bash
#!/bin/bash
# upload_video.sh

VIDEO_FILE="$1"
PROMPT="${2:-Describe this video}"

# Get file size and mime type
FILE_SIZE=$(stat -f%z "$VIDEO_FILE" 2>/dev/null || stat -c%s "$VIDEO_FILE")
MIME_TYPE="video/mp4"

# Start upload
echo "Starting upload..."
INIT_RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/upload/v1beta/files" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "X-Goog-Upload-Command: start" \
  -H "X-Goog-Upload-Header-Content-Length: $FILE_SIZE" \
  -H "X-Goog-Upload-Header-Content-Type: $MIME_TYPE" \
  -H "Content-Type: application/json" \
  -d '{"file": {"display_name": "'"$(basename "$VIDEO_FILE")"'"}}' \
  -D -)

# Extract upload URL
UPLOAD_URL=$(echo "$INIT_RESPONSE" | grep -i "x-goog-upload-url:" | cut -d' ' -f2 | tr -d '\r')

# Upload file
echo "Uploading video data..."
UPLOAD_RESPONSE=$(curl -s -X POST "$UPLOAD_URL" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Length: $FILE_SIZE" \
  -H "X-Goog-Upload-Offset: 0" \
  -H "X-Goog-Upload-Command: upload, finalize" \
  --data-binary @"$VIDEO_FILE")

# Extract file name
FILE_NAME=$(echo "$UPLOAD_RESPONSE" | jq -r '.file.name')
FILE_URI=$(echo "$UPLOAD_RESPONSE" | jq -r '.file.uri')

echo "File uploaded: $FILE_NAME"

# Wait for processing
echo "Processing video..."
while true; do
  STATUS=$(curl -s \
    "https://generativelanguage.googleapis.com/v1beta/$FILE_NAME" \
    -H "x-goog-api-key: $GEMINI_API_KEY" | jq -r '.state')
  
  if [ "$STATUS" = "ACTIVE" ]; then
    echo "Video ready!"
    break
  elif [ "$STATUS" = "FAILED" ]; then
    echo "Processing failed!"
    exit 1
  fi
  
  echo -n "."
  sleep 5
done

# Analyze video
echo "Analyzing video..."
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "file_data": {
            "file_uri": "'"$FILE_URI"'",
            "mime_type": "'"$MIME_TYPE"'"
          }
        },
        {
          "text": "'"$PROMPT"'"
        }
      ]
    }]
  }' | jq -r '.candidates[0].content.parts[0].text'
```

## Python Example

```python
import requests
import time
import os

def upload_video(video_path):
    """Upload video to Gemini File API"""
    api_key = os.environ["GEMINI_API_KEY"]
    file_size = os.path.getsize(video_path)
    
    # Start upload
    headers = {
        "x-goog-api-key": api_key,
        "X-Goog-Upload-Command": "start",
        "X-Goog-Upload-Header-Content-Length": str(file_size),
        "X-Goog-Upload-Header-Content-Type": "video/mp4",
        "Content-Type": "application/json"
    }
    
    data = {
        "file": {
            "display_name": os.path.basename(video_path)
        }
    }
    
    response = requests.post(
        "https://generativelanguage.googleapis.com/upload/v1beta/files",
        headers=headers,
        json=data
    )
    
    # Get upload URL
    upload_url = response.headers.get("X-Goog-Upload-URL")
    
    # Upload video data
    with open(video_path, "rb") as f:
        upload_headers = {
            "x-goog-api-key": api_key,
            "Content-Length": str(file_size),
            "X-Goog-Upload-Offset": "0",
            "X-Goog-Upload-Command": "upload, finalize"
        }
        
        upload_response = requests.post(
            upload_url,
            headers=upload_headers,
            data=f
        )
    
    file_info = upload_response.json()
    return file_info["file"]

def wait_for_processing(file_name, api_key):
    """Wait for video to be processed"""
    while True:
        response = requests.get(
            f"https://generativelanguage.googleapis.com/v1beta/{file_name}",
            headers={"x-goog-api-key": api_key}
        )
        
        file_info = response.json()
        if file_info["state"] == "ACTIVE":
            return file_info
        elif file_info["state"] == "FAILED":
            raise Exception("Video processing failed")
        
        time.sleep(5)

def analyze_video(file_uri, prompt, api_key):
    """Analyze uploaded video"""
    url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"
    
    headers = {
        "x-goog-api-key": api_key,
        "Content-Type": "application/json"
    }
    
    data = {
        "contents": [{
            "parts": [
                {
                    "file_data": {
                        "file_uri": file_uri,
                        "mime_type": "video/mp4"
                    }
                },
                {
                    "text": prompt
                }
            ]
        }]
    }
    
    response = requests.post(url, headers=headers, json=data)
    result = response.json()
    
    return result["candidates"][0]["content"]["parts"][0]["text"]
```

## Video Analysis Examples

### Scene Detection
```json
{
  "contents": [{
    "parts": [
      {
        "file_data": {
          "file_uri": "https://generativelanguage.googleapis.com/v1beta/files/abc123",
          "mime_type": "video/mp4"
        }
      },
      {
        "text": "Provide scene-by-scene breakdown with timestamps"
      }
    ]
  }]
}
```

### Content Moderation
```json
{
  "contents": [{
    "parts": [
      {
        "file_data": {
          "file_uri": "FILE_URI",
          "mime_type": "video/mp4"
        }
      },
      {
        "text": "Check for inappropriate content or policy violations"
      }
    ]
  }],
  "safetySettings": [{
    "category": "HARM_CATEGORY_DANGEROUS",
    "threshold": "BLOCK_LOW_AND_ABOVE"
  }]
}
```

## Delete Video

```bash
# Clean up after processing
curl -X DELETE \
  "https://generativelanguage.googleapis.com/v1beta/files/abc123" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Error Handling

```bash
# Check for upload errors
if [ -z "$FILE_NAME" ]; then
    echo "Upload failed"
    exit 1
fi

# Check processing errors
if [ "$STATUS" = "FAILED" ]; then
    ERROR=$(curl -s \
      "https://generativelanguage.googleapis.com/v1beta/$FILE_NAME" \
      -H "x-goog-api-key: $GEMINI_API_KEY" | jq -r '.error.message')
    echo "Error: $ERROR"
    exit 1
fi
```

## Best Practices

1. **Resumable Uploads**: Use for large files
2. **Status Checks**: Poll processing status
3. **Cleanup**: Delete files after use
4. **Error Handling**: Check each step
5. **Compression**: Pre-compress if needed

## See Also
- [File API](file-api.md) - File upload details
- [Vision](vision.md) - Image processing
- [Audio](audio.md) - Audio with video