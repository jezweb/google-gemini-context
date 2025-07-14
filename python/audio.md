# Audio in Python

*Audio understanding and generation with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/audio
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Transcription, understanding, multimodal with audio
-->

## Quick Reference
- **Input formats**: MP3, WAV, FLAC, AAC, OGG, OPUS
- **Max duration**: 8.4 hours per prompt
- **Max file size**: 20MB
- **Features**: Transcription, Q&A, summarization

## Basic Audio Understanding

```python
from google import genai
import base64

client = genai.Client()

# Read audio file
with open("speech.mp3", "rb") as f:
    audio_data = base64.b64encode(f.read()).decode()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": audio_data
            }
        },
        {"text": "Transcribe this audio and summarize the main points"}
    ]
)

print(response.text)
```

## Audio Transcription

```python
# Pure transcription
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "audio/wav",
                "data": audio_data
            }
        },
        {"text": "Transcribe this audio verbatim"}
    ]
)

print("Transcript:", response.text)
```

## Audio Analysis

```python
# Analyze audio content
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": audio_data
            }
        },
        {
            "text": """Analyze this audio and provide:
            1. Speaker identification
            2. Emotion/tone analysis
            3. Key topics discussed
            4. Audio quality assessment"""
        }
    ]
)

print(response.text)
```

## Multiple Audio Files

```python
# Compare multiple audio files
with open("interview1.mp3", "rb") as f:
    audio1_data = base64.b64encode(f.read()).decode()
    
with open("interview2.mp3", "rb") as f:
    audio2_data = base64.b64encode(f.read()).decode()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"text": "Compare these two interviews"},
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": audio1_data
            }
        },
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": audio2_data
            }
        }
    ]
)
```

## Audio with Images

```python
# Multimodal: audio + image
with open("presentation.mp3", "rb") as f:
    audio_data = base64.b64encode(f.read()).decode()
    
with open("slide.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": audio_data
            }
        },
        {
            "inline_data": {
                "mime_type": "image/jpeg",
                "data": image_data
            }
        },
        {"text": "How does the audio narration relate to this slide?"}
    ]
)
```

## Streaming Audio Analysis

```python
# Stream response for long audio
for chunk in client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents=[
        {
            "inline_data": {
                "mime_type": "audio/mp3",
                "data": long_audio_data
            }
        },
        {"text": "Provide a detailed transcript with timestamps"}
    ]
):
    print(chunk.text, end='', flush=True)
```

## Audio Processing Utilities

```python
from pathlib import Path
from typing import Optional, Dict, Any
import base64

class AudioProcessor:
    def __init__(self):
        self.client = genai.Client()
        self.mime_types = {
            '.mp3': 'audio/mp3',
            '.wav': 'audio/wav',
            '.flac': 'audio/flac',
            '.aac': 'audio/aac',
            '.ogg': 'audio/ogg',
            '.opus': 'audio/opus',
        }
    
    def load_audio(self, audio_path: str) -> Dict[str, Any]:
        """Load audio file and return inline data format"""
        path = Path(audio_path)
        mime_type = self.mime_types.get(path.suffix.lower(), 'audio/mp3')
        
        with open(audio_path, "rb") as f:
            data = base64.b64encode(f.read()).decode()
        
        return {
            "inline_data": {
                "mime_type": mime_type,
                "data": data
            }
        }
    
    def transcribe(
        self, 
        audio_path: str, 
        prompt: Optional[str] = None,
        model: str = "gemini-2.5-flash"
    ) -> str:
        """Transcribe audio file"""
        audio_content = self.load_audio(audio_path)
        
        contents = [
            audio_content,
            {"text": prompt or "Transcribe this audio"}
        ]
        
        response = self.client.models.generate_content(
            model=model,
            contents=contents
        )
        
        return response.text
    
    def analyze_podcast(self, audio_path: str) -> Dict[str, Any]:
        """Analyze podcast episode"""
        audio_content = self.load_audio(audio_path)
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                audio_content,
                {
                    "text": """Analyze this podcast episode:
                    - Title and topic
                    - Guest speakers
                    - Key discussion points
                    - Notable quotes
                    - Episode summary
                    
                    Format as JSON."""
                }
            ],
            config=genai.GenerateContentConfig(
                response_mime_type="application/json"
            )
        )
        
        import json
        return json.loads(response.text)
    
    def extract_meeting_notes(self, audio_path: str) -> str:
        """Extract structured meeting notes"""
        audio_content = self.load_audio(audio_path)
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                audio_content,
                {
                    "text": """Transcribe this meeting and provide:
                    - Attendee list
                    - Agenda items discussed
                    - Action items with owners
                    - Key decisions made
                    - Next steps and deadlines"""
                }
            ]
        )
        
        return response.text
```

## Common Use Cases

### Meeting Transcription
```python
processor = AudioProcessor()
meeting_notes = processor.extract_meeting_notes("team_meeting.mp3")
print(meeting_notes)
```

### Language Detection
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        audio_content,
        {"text": "What language is being spoken in this audio?"}
    ]
)
print(f"Language: {response.text}")
```

### Audio Summarization
```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        audio_content,
        {"text": "Provide a 3-paragraph summary of this audio"}
    ]
)
```

## Batch Processing

```python
def process_audio_batch(audio_files: list) -> list:
    """Process multiple audio files"""
    results = []
    
    for audio_path in audio_files:
        try:
            processor = AudioProcessor()
            transcript = processor.transcribe(audio_path)
            results.append({
                "file": audio_path,
                "transcript": transcript,
                "status": "success"
            })
        except Exception as e:
            results.append({
                "file": audio_path,
                "error": str(e),
                "status": "failed"
            })
    
    return results
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[audio_content, {"text": prompt}]
    )
    return response.text
except Exception as e:
    if "File size exceeds" in str(e):
        print("Audio file too large (max 20MB)")
    elif "Unsupported audio format" in str(e):
        print("Use MP3, WAV, FLAC, AAC, OGG, or OPUS")
    else:
        raise
```

## Best Practices

1. **File Size**: Keep under 20MB, compress if needed
2. **Quality**: Higher quality improves transcription
3. **Format**: MP3 and WAV most reliable
4. **Prompts**: Be specific about desired output format
5. **Long Audio**: Use streaming for better UX

## See Also
- [Vision](vision.md) - Multimodal with images
- [Video](video.md) - Video with audio tracks
- [Streaming](streaming.md) - Real-time responses