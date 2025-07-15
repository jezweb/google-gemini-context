# REST API Music Generation

*Generate instrumental music with Lyria RealTime*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs
Verified: 2025-01-14
Models: Lyria RealTime
Note: Available in Google AI Studio and Gemini API
-->

## Quick Reference
- **Endpoint**: /models/lyria-realtime:generateContent
- **Type**: Instrumental music generation
- **Response**: Base64 encoded audio
- **Control**: Text prompts for style/mood
- **Access**: API key required

## Basic Music Generation

```bash
# Generate instrumental music
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Create upbeat electronic dance music"
      }]
    }],
    "generationConfig": {
      "responseModalities": ["AUDIO"]
    }
  }'

# Response contains base64 audio data
```

## Save Generated Music

```bash
# Generate and save music
response=$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Calm ambient music with soft synthesizers"
      }]
    }],
    "generationConfig": {
      "responseModalities": ["AUDIO"]
    }
  }')

# Extract and save audio
echo "$response" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > ambient_music.mp3
```

## Style Examples

```bash
# Function to generate different styles
generate_music_style() {
  local style="$1"
  local output="$2"
  
  local prompts=(
    "ambient:Calm ambient music with soft synthesizers"
    "jazz:Smooth jazz with piano and saxophone feel"
    "classical:Classical orchestral piece in major key"
    "electronic:Energetic electronic music with strong bass"
    "lofi:Lo-fi hip hop beats for studying"
    "cinematic:Epic cinematic soundtrack with orchestral elements"
  )
  
  # Find matching prompt
  local prompt=""
  for p in "${prompts[@]}"; do
    if [[ $p == "$style:"* ]]; then
      prompt="${p#*:}"
      break
    fi
  done
  
  if [ -z "$prompt" ]; then
    prompt="$style instrumental music"
  fi
  
  curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [{
          \"text\": \"$prompt\"
        }]
      }],
      \"generationConfig\": {
        \"responseModalities\": [\"AUDIO\"]
      }
    }" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output"
}

# Generate different styles
generate_music_style "ambient" "ambient.mp3"
generate_music_style "jazz" "jazz.mp3"
generate_music_style "electronic" "electronic.mp3"
```

## Mood-Based Generation

```bash
# Generate music by mood
generate_by_mood() {
  local mood="$1"
  local tempo="$2"
  local instruments="$3"
  local output="$4"
  
  # Build prompt
  local prompt="$mood instrumental music"
  [ -n "$tempo" ] && prompt="$prompt at $tempo tempo"
  [ -n "$instruments" ] && prompt="$prompt featuring $instruments"
  
  curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [{
          \"text\": \"$prompt\"
        }]
      }],
      \"generationConfig\": {
        \"responseModalities\": [\"AUDIO\"]
      }
    }" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output"
  
  echo "Generated: $output - $prompt"
}

# Examples
generate_by_mood "happy" "upbeat" "piano, strings" "happy_music.mp3"
generate_by_mood "relaxing" "slow" "ambient pads" "relaxing_music.mp3"
generate_by_mood "mysterious" "moderate" "synthesizers, subtle percussion" "mysterious.mp3"
```

## Detailed Prompts

```bash
# Create detailed music prompts
create_detailed_prompt() {
  local genre="$1"
  local mood="$2"
  local tempo="$3"
  local key="$4"
  local elements="$5"
  
  local prompt="$mood $genre instrumental music"
  
  [ -n "$tempo" ] && prompt="$prompt at $tempo BPM"
  [ -n "$key" ] && prompt="$prompt in $key"
  [ -n "$elements" ] && prompt="$prompt featuring $elements"
  
  echo "$prompt"
}

# Generate with detailed prompt
prompt=$(create_detailed_prompt "electronic" "energetic" "128" "" "synthesizers, drum machine, bass drops")

curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"contents\": [{
      \"parts\": [{
        \"text\": \"$prompt\"
      }]
    }],
    \"generationConfig\": {
      \"responseModalities\": [\"AUDIO\"]
    }
  }" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "electronic_128bpm.mp3"
```

## Batch Processing

```bash
#!/bin/bash
# batch_music_generation.sh

API_KEY="your-api-key"
OUTPUT_DIR="generated_music"
mkdir -p "$OUTPUT_DIR"

# List of music prompts
prompts=(
  "Peaceful morning meditation music"
  "Energetic workout motivation music"
  "Focused concentration music for studying"
  "Relaxing evening wind-down music"
  "Uplifting celebratory party music"
)

# Generate each prompt
for i in "${!prompts[@]}"; do
  prompt="${prompts[$i]}"
  output_file="$OUTPUT_DIR/music_$(printf %02d $((i+1))).mp3"
  
  echo "Generating: $prompt"
  
  response=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [{
          \"text\": \"$prompt\"
        }]
      }],
      \"generationConfig\": {
        \"responseModalities\": [\"AUDIO\"]
      }
    }")
  
  # Check for errors
  if echo "$response" | jq -e '.error' > /dev/null; then
    echo "Error: $(echo "$response" | jq -r '.error.message')"
  else
    echo "$response" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output_file"
    echo "Saved: $output_file"
  fi
  
  # Rate limiting
  sleep 3
done
```

## Scene-Based Music

```bash
# Generate music for different scenes
generate_scene_music() {
  local scene_type="$1"
  
  case "$scene_type" in
    "menu")
      prompt="Ambient menu music with subtle melody"
      ;;
    "battle")
      prompt="Intense battle music with drums and orchestra"
      ;;
    "exploration")
      prompt="Adventurous exploration music with wonder"
      ;;
    "boss")
      prompt="Epic boss battle music with dramatic tension"
      ;;
    "victory")
      prompt="Triumphant victory fanfare"
      ;;
    "peaceful")
      prompt="Peaceful village music with folk instruments"
      ;;
    *)
      prompt="Game music for $scene_type scene"
      ;;
  esac
  
  curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [{
          \"text\": \"$prompt\"
        }]
      }],
      \"generationConfig\": {
        \"responseModalities\": [\"AUDIO\"]
      }
    }" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "scene_${scene_type}.mp3"
}

# Generate music for all scenes
scenes=("menu" "battle" "exploration" "boss" "victory" "peaceful")
for scene in "${scenes[@]}"; do
  echo "Generating music for: $scene"
  generate_scene_music "$scene"
  sleep 2
done
```

## Error Handling

```bash
# Safe music generation with retries
safe_generate_music() {
  local prompt="$1"
  local output="$2"
  local max_retries=3
  local retry=0
  
  while [ $retry -lt $max_retries ]; do
    response=$(curl -s -w "\n%{http_code}" -X POST \
      "https://generativelanguage.googleapis.com/v1beta/models/lyria-realtime:generateContent?key=$API_KEY" \
      -H "Content-Type: application/json" \
      -d "{
        \"contents\": [{
          \"parts\": [{
            \"text\": \"$prompt\"
          }]
        }],
        \"generationConfig\": {
          \"responseModalities\": [\"AUDIO\"]
        }
      }")
    
    http_code=$(echo "$response" | tail -n1)
    body=$(echo "$response" | sed '$d')
    
    if [ "$http_code" -eq 200 ]; then
      # Check if audio was generated
      if echo "$body" | jq -e '.candidates[0].content.parts[0].inlineData.data' > /dev/null; then
        echo "$body" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output"
        echo "Success: Music saved to $output"
        return 0
      else
        echo "No audio generated"
      fi
    elif [ "$http_code" -eq 429 ]; then
      echo "Rate limit hit, waiting..."
      sleep $((2 ** retry))
      ((retry++))
    else
      echo "Error ($http_code): $(echo "$body" | jq -r '.error.message // "Unknown error"')"
      return 1
    fi
  done
  
  echo "Failed after $max_retries retries"
  return 1
}

# Usage
safe_generate_music "Uplifting orchestral music" "orchestral.mp3"
```

## Music Library Script

```bash
#!/bin/bash
# music_library.sh - Organize generated music

LIBRARY_DIR="music_library"
mkdir -p "$LIBRARY_DIR"

# Save with metadata
save_music_with_metadata() {
  local prompt="$1"
  local style="$2"
  local tags="$3"
  
  local timestamp=$(date +%Y%m%d_%H%M%S)
  local filename="${style}_${timestamp}.mp3"
  local filepath="$LIBRARY_DIR/$filename"
  
  # Generate music
  if safe_generate_music "$prompt" "$filepath"; then
    # Save metadata
    cat > "${filepath%.mp3}_metadata.json" <<EOF
{
  "filename": "$filename",
  "prompt": "$prompt",
  "style": "$style",
  "tags": [$tags],
  "created": "$timestamp",
  "size": $(stat -f%z "$filepath" 2>/dev/null || stat -c%s "$filepath")
}
EOF
    echo "Saved with metadata: $filename"
  fi
}

# Generate library
save_music_with_metadata "Calm meditation music" "meditation" '"relaxing","ambient","calm"'
save_music_with_metadata "Energetic workout music" "workout" '"energetic","motivational","upbeat"'
save_music_with_metadata "Focus music for productivity" "focus" '"concentration","work","study"'
```

## Response Format

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "inlineData": {
              "mimeType": "audio/mp3",
              "data": "base64-encoded-audio-data"
            }
          }
        ],
        "role": "model"
      }
    }
  ]
}
```

## Use Cases

### Background Music Generator
```bash
generate_background() {
  local activity="$1"
  local duration_hint="$2"
  
  case "$activity" in
    "work")
      prompt="Focused instrumental music for productivity, $duration_hint"
      ;;
    "meditation")
      prompt="Calm meditation music with nature sounds, $duration_hint"
      ;;
    "exercise")
      prompt="High-energy workout music with strong beat, $duration_hint"
      ;;
    "sleep")
      prompt="Gentle sleep music with soft ambient tones, $duration_hint"
      ;;
    *)
      prompt="Background music for $activity, $duration_hint"
      ;;
  esac
  
  safe_generate_music "$prompt" "${activity}_background.mp3"
}
```

## Limitations
- Instrumental only (no vocals)
- Limited direct control over duration
- No precise BPM/key control
- Generated as complete pieces
- API availability may vary

## See Also
- [Audio Understanding](audio.md)
- [Speech Generation](speech-generation.md)
- [Endpoints](endpoints.md)