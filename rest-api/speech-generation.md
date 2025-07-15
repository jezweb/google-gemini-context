# REST API Speech Generation (TTS)

*Generate natural speech from text using Gemini TTS models*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/speech-generation
Verified: 2025-01-14
Models: Gemini 2.5 Flash Preview TTS, Gemini 2.5 Pro Preview TTS
Note: Preview feature with controllable speech synthesis
-->

## Quick Reference
- **Endpoint**: /models/{model}:generateContent
- **Models**: gemini-2.5-flash-preview-tts, gemini-2.5-pro-preview-tts
- **Response**: Base64 encoded audio
- **Languages**: 24+ with auto-detection
- **Voices**: 30 unique options

## Basic Text-to-Speech

```bash
# Generate speech with default voice
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Welcome to the future of AI-powered speech synthesis!"
      }]
    }],
    "generationConfig": {
      "responseModalities": ["AUDIO"],
      "speechConfig": {
        "voiceConfig": {
          "prebuiltVoiceConfig": {
            "voiceName": "Kore"
          }
        }
      }
    }
  }'
```

## Save Audio Output

```bash
# Generate and save as MP3
response=$(curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Hello world, this is a test of speech synthesis."
      }]
    }],
    "generationConfig": {
      "responseModalities": ["AUDIO"],
      "speechConfig": {
        "voiceConfig": {
          "prebuiltVoiceConfig": {
            "voiceName": "Kore"
          }
        }
      }
    }
  }')

# Extract and decode audio
echo "$response" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > output.mp3
```

## Voice Selection

```bash
# Available voices
voices=(
  "Zephyr"    # Bright
  "Puck"      # Upbeat
  "Aura"      # Energetic
  "Charon"    # Informative
  "Kore"      # Engaging
  "Helios"    # Confident
  "Enceladus" # Breathy
  "Vega"      # Warm
  "Luna"      # Gentle
  "Orion"     # Deep
  "Atlas"     # Authoritative
  "Titan"     # Resonant
)

# Generate with specific voice
for voice in "${voices[@]}"; do
  curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "Testing voice: '"$voice"'"
        }]
      }],
      "generationConfig": {
        "responseModalities": ["AUDIO"],
        "speechConfig": {
          "voiceConfig": {
            "prebuiltVoiceConfig": {
              "voiceName": "'$voice'"
            }
          }
        }
      }
    }' | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "voice_${voice}.mp3"
done
```

## Natural Language Control

```bash
# Control speech style through prompts
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Say cheerfully: Have a wonderful day!"
      }]
    }],
    "generationConfig": {
      "responseModalities": ["AUDIO"],
      "speechConfig": {
        "voiceConfig": {
          "prebuiltVoiceConfig": {
            "voiceName": "Puck"
          }
        }
      }
    }
  }'

# More style examples
styles=(
  "Say sadly: I'll miss you"
  "Say excitedly: We won the championship!"
  "Say calmly: Everything will be alright"
  "Say mysteriously: The secret lies within"
  "Say urgently: We need to leave now!"
)
```

## Multi-Language Support

```bash
# Generate speech in multiple languages (auto-detected)
languages=(
  "Hello, how are you today?"              # English
  "Hola, ¿cómo estás hoy?"                # Spanish
  "Bonjour, comment allez-vous?"          # French
  "Hallo, wie geht es dir heute?"         # German
  "こんにちは、今日はどうですか？"            # Japanese
  "你好，你今天怎么样？"                    # Chinese
  "नमस्ते, आज आप कैसे हैं?"              # Hindi
)

for text in "${languages[@]}"; do
  curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$text"'"
        }]
      }],
      "generationConfig": {
        "responseModalities": ["AUDIO"],
        "speechConfig": {
          "voiceConfig": {
            "prebuiltVoiceConfig": {
              "voiceName": "Kore"
            }
          }
        }
      }
    }' | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "lang_$(date +%s).mp3"
  
  sleep 1
done
```

## Batch Processing Script

```bash
#!/bin/bash
# batch_tts.sh - Convert text files to speech

API_KEY="your-api-key"
VOICE="Kore"
MODEL="gemini-2.5-flash-preview-tts"

process_text_file() {
  local input_file="$1"
  local output_file="${input_file%.txt}.mp3"
  local text=$(cat "$input_file" | jq -Rs .)
  
  echo "Processing: $input_file"
  
  response=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/$MODEL:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": '"$text"'
        }]
      }],
      "generationConfig": {
        "responseModalities": ["AUDIO"],
        "speechConfig": {
          "voiceConfig": {
            "prebuiltVoiceConfig": {
              "voiceName": "'"$VOICE"'"
            }
          }
        }
      }
    }')
  
  if [[ $response == *"error"* ]]; then
    echo "Error processing $input_file"
    echo "$response" | jq '.error'
  else
    echo "$response" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output_file"
    echo "Saved: $output_file"
  fi
}

# Process all text files
for file in *.txt; do
  process_text_file "$file"
  sleep 2  # Rate limiting
done
```

## Multi-Speaker Dialog

```bash
# Generate dialog with multiple speakers
generate_dialog() {
  local speaker="$1"
  local text="$2"
  local output="$3"
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$text"'"
        }]
      }],
      "generationConfig": {
        "responseModalities": ["AUDIO"],
        "speechConfig": {
          "voiceConfig": {
            "prebuiltVoiceConfig": {
              "voiceName": "'"$speaker"'"
            }
          }
        }
      }
    }' | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output"
}

# Create conversation
generate_dialog "Kore" "Welcome to our podcast!" "dialog_1.mp3"
generate_dialog "Orion" "Thanks for having me." "dialog_2.mp3"
generate_dialog "Kore" "Let's dive into today's topic." "dialog_3.mp3"

# Concatenate audio files (requires ffmpeg)
ffmpeg -i "concat:dialog_1.mp3|dialog_2.mp3|dialog_3.mp3" -acodec copy conversation.mp3
```

## Advanced Speech Control

```bash
# Function for expressive speech
generate_expressive() {
  local instructions="$1"
  local text="$2"
  local voice="${3:-Helios}"
  
  curl -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro-preview-tts:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$instructions: $text"'"
        }]
      }],
      "generationConfig": {
        "responseModalities": ["AUDIO"],
        "speechConfig": {
          "voiceConfig": {
            "prebuiltVoiceConfig": {
              "voiceName": "'"$voice"'"
            }
          }
        }
      }
    }'
}

# Examples
generate_expressive "Say slowly and clearly" "The quick brown fox" "Luna"
generate_expressive "Say urgently with emphasis" "Breaking news!" "Helios"
generate_expressive "Say in a storytelling voice" "Once upon a time" "Luna"
```

## Error Handling

```bash
# Function with error handling
safe_generate_tts() {
  local text="$1"
  local voice="${2:-Kore}"
  local output="${3:-output.mp3}"
  local max_retries=3
  local retry=0
  
  while [ $retry -lt $max_retries ]; do
    response=$(curl -s -w "\n%{http_code}" -X POST \
      "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "contents": [{
          "parts": [{
            "text": "'"$text"'"
          }]
        }],
        "generationConfig": {
          "responseModalities": ["AUDIO"],
          "speechConfig": {
            "voiceConfig": {
              "prebuiltVoiceConfig": {
                "voiceName": "'"$voice"'"
              }
            }
          }
        }
      }')
    
    http_code=$(echo "$response" | tail -n1)
    body=$(echo "$response" | sed '$d')
    
    if [ "$http_code" -eq 200 ]; then
      echo "$body" | jq -r '.candidates[0].content.parts[0].inlineData.data' | base64 -d > "$output"
      echo "Success: Audio saved to $output"
      return 0
    elif [ "$http_code" -eq 429 ]; then
      echo "Rate limit hit, waiting..."
      sleep $((2 ** retry))
      ((retry++))
    else
      echo "Error: $body"
      return 1
    fi
  done
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
            "inlineData": {
              "mimeType": "audio/mp3",
              "data": "base64-encoded-audio-data"
            }
          }
        ],
        "role": "model"
      }
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 10,
    "candidatesTokenCount": 0,
    "totalTokenCount": 10
  }
}
```

## Voice Selection Guide

```bash
# Choose voice based on content type
select_voice() {
  case "$1" in
    "news")        echo "Charon" ;;     # Informative
    "story")       echo "Luna" ;;       # Gentle
    "tutorial")    echo "Kore" ;;       # Engaging
    "announcement") echo "Helios" ;;    # Confident
    "meditation")  echo "Enceladus" ;;  # Calm
    *)            echo "Kore" ;;        # Default
  esac
}

# Usage
voice=$(select_voice "news")
safe_generate_tts "Breaking news update..." "$voice" "news.mp3"
```

## Limitations
- Preview feature (may change)
- 32k token context limit
- Base64 encoding increases payload size
- No direct SSML support
- Rate limits apply

## See Also
- [Audio Understanding](audio.md)
- [Text Generation](text-generation.md)
- [Endpoints](endpoints.md)