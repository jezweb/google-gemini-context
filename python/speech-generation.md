# Python Speech Generation (TTS)

*Generate natural speech from text using Gemini TTS models*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/speech-generation
Verified: 2025-01-14
Models: Gemini 2.5 Flash Preview TTS, Gemini 2.5 Pro Preview TTS
Note: Preview feature with controllable speech synthesis
-->

## Quick Reference
- **Models**: gemini-2.5-flash-preview-tts, gemini-2.5-pro-preview-tts
- **Languages**: 24+ languages with auto-detection
- **Voices**: 30 unique voice options
- **Output**: Audio in various formats
- **Context**: 32k token window

## Basic Text-to-Speech

```python
from google import genai
from google.genai import types
import base64

client = genai.Client()

# Generate speech
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-tts",
    contents="Welcome to the future of AI-powered speech synthesis!",
    config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
        speech_config=types.SpeechConfig(
            voice_config=types.VoiceConfig(
                prebuilt_voice_config=types.PrebuiltVoiceConfig(
                    voice_name='Kore',
                )
            )
        ),
    )
)

# Save audio file
for part in response.candidates[0].content.parts:
    if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
        audio_data = base64.b64decode(part.inline_data.data)
        with open('output.mp3', 'wb') as f:
            f.write(audio_data)
```

## Voice Selection

```python
# Available voices with characteristics
voices = {
    # Bright/Upbeat voices
    'Zephyr': 'Bright',
    'Puck': 'Upbeat',
    'Aura': 'Energetic',
    
    # Professional/Clear voices
    'Charon': 'Informative',
    'Kore': 'Engaging',
    'Helios': 'Confident',
    
    # Soft/Warm voices
    'Enceladus': 'Breathy',
    'Vega': 'Warm',
    'Luna': 'Gentle',
    
    # Deep/Authoritative voices
    'Orion': 'Deep',
    'Atlas': 'Authoritative',
    'Titan': 'Resonant'
}

# Generate with specific voice
def generate_with_voice(text, voice_name):
    response = client.models.generate_content(
        model="gemini-2.5-flash-preview-tts",
        contents=text,
        config=types.GenerateContentConfig(
            response_modalities=["AUDIO"],
            speech_config=types.SpeechConfig(
                voice_config=types.VoiceConfig(
                    prebuilt_voice_config=types.PrebuiltVoiceConfig(
                        voice_name=voice_name,
                    )
                )
            ),
        )
    )
    return response
```

## Natural Language Control

```python
# Control speech style through prompts
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-tts",
    contents="Say cheerfully: Have a wonderful day!",
    config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
        speech_config=types.SpeechConfig(
            voice_config=types.VoiceConfig(
                prebuilt_voice_config=types.PrebuiltVoiceConfig(
                    voice_name='Puck',  # Upbeat voice
                )
            )
        ),
    )
)

# More style examples
styles = [
    "Say sadly: I'll miss you",
    "Say excitedly: We won the championship!",
    "Say calmly: Everything will be alright",
    "Say mysteriously: The secret lies within",
    "Say urgently: We need to leave now!"
]
```

## Multi-Speaker Dialog

```python
# Generate conversation with multiple voices
def generate_dialog(script):
    """Generate multi-speaker dialog from script"""
    audio_parts = []
    
    for line in script:
        response = client.models.generate_content(
            model="gemini-2.5-flash-preview-tts",
            contents=line['text'],
            config=types.GenerateContentConfig(
                response_modalities=["AUDIO"],
                speech_config=types.SpeechConfig(
                    voice_config=types.VoiceConfig(
                        prebuilt_voice_config=types.PrebuiltVoiceConfig(
                            voice_name=line['speaker'],
                        )
                    )
                ),
            )
        )
        
        # Extract audio
        for part in response.candidates[0].content.parts:
            if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
                audio_parts.append(base64.b64decode(part.inline_data.data))
    
    return audio_parts

# Example dialog
script = [
    {'speaker': 'Kore', 'text': 'Welcome to our podcast!'},
    {'speaker': 'Orion', 'text': 'Thanks for having me.'},
    {'speaker': 'Kore', 'text': 'Let\'s dive into today\'s topic.'}
]

audio_segments = generate_dialog(script)
```

## Language Support

```python
# Multi-language generation (auto-detected)
languages = {
    'en': "Hello, how are you today?",
    'es': "Hola, ¿cómo estás hoy?",
    'fr': "Bonjour, comment allez-vous aujourd'hui?",
    'de': "Hallo, wie geht es dir heute?",
    'ja': "こんにちは、今日はどうですか？",
    'zh': "你好，你今天怎么样？",
    'hi': "नमस्ते, आज आप कैसे हैं?",
    'ar': "مرحبا، كيف حالك اليوم؟"
}

def generate_multilingual(texts, voice_name='Kore'):
    results = {}
    
    for lang, text in texts.items():
        response = client.models.generate_content(
            model="gemini-2.5-flash-preview-tts",
            contents=text,  # Language auto-detected
            config=types.GenerateContentConfig(
                response_modalities=["AUDIO"],
                speech_config=types.SpeechConfig(
                    voice_config=types.VoiceConfig(
                        prebuilt_voice_config=types.PrebuiltVoiceConfig(
                            voice_name=voice_name,
                        )
                    )
                ),
            )
        )
        results[lang] = response
    
    return results
```

## Advanced Speech Control

```python
# Control pace, tone, and emphasis
def generate_expressive_speech(text, instructions):
    """Generate speech with specific instructions"""
    
    # Combine instructions with text
    prompt = f"{instructions}: {text}"
    
    response = client.models.generate_content(
        model="gemini-2.5-pro-preview-tts",  # Pro for better control
        contents=prompt,
        config=types.GenerateContentConfig(
            response_modalities=["AUDIO"],
            speech_config=types.SpeechConfig(
                voice_config=types.VoiceConfig(
                    prebuilt_voice_config=types.PrebuiltVoiceConfig(
                        voice_name='Helios',
                    )
                )
            ),
        )
    )
    
    return response

# Examples
examples = [
    ("The quick brown fox", "Say slowly and clearly"),
    ("Breaking news!", "Say urgently with emphasis"),
    ("Once upon a time", "Say in a storytelling voice"),
    ("Thank you for your patience", "Say politely and warmly")
]
```

## Batch Processing

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def process_batch(texts, voice_name='Kore', max_workers=5):
    """Process multiple texts efficiently"""
    
    def generate_single(text):
        try:
            response = client.models.generate_content(
                model="gemini-2.5-flash-preview-tts",
                contents=text,
                config=types.GenerateContentConfig(
                    response_modalities=["AUDIO"],
                    speech_config=types.SpeechConfig(
                        voice_config=types.VoiceConfig(
                            prebuilt_voice_config=types.PrebuiltVoiceConfig(
                                voice_name=voice_name,
                            )
                        )
                    ),
                )
            )
            return text, response
        except Exception as e:
            return text, None
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(generate_single, texts))
    
    return results

# Process multiple texts
texts = [
    "Welcome to chapter one",
    "This is the introduction",
    "Let's begin our journey"
]

results = process_batch(texts)
```

## Save in Different Formats

```python
import wave
import io

def save_audio(response, filename, format='mp3'):
    """Save audio response in specified format"""
    
    for part in response.candidates[0].content.parts:
        if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
            audio_data = base64.b64decode(part.inline_data.data)
            
            if format == 'mp3':
                with open(f'{filename}.mp3', 'wb') as f:
                    f.write(audio_data)
            elif format == 'wav':
                # Convert if needed (example with pydub)
                from pydub import AudioSegment
                audio = AudioSegment.from_mp3(io.BytesIO(audio_data))
                audio.export(f'{filename}.wav', format='wav')
            
            return f'{filename}.{format}'
```

## Error Handling

```python
def safe_generate_speech(text, voice_name='Kore', retry=3):
    """Generate speech with error handling"""
    
    for attempt in range(retry):
        try:
            response = client.models.generate_content(
                model="gemini-2.5-flash-preview-tts",
                contents=text,
                config=types.GenerateContentConfig(
                    response_modalities=["AUDIO"],
                    speech_config=types.SpeechConfig(
                        voice_config=types.VoiceConfig(
                            prebuilt_voice_config=types.PrebuiltVoiceConfig(
                                voice_name=voice_name,
                            )
                        )
                    ),
                )
            )
            return response
        
        except Exception as e:
            if "INVALID_ARGUMENT" in str(e):
                print(f"Invalid voice or text: {e}")
                break
            elif "RESOURCE_EXHAUSTED" in str(e):
                print(f"Rate limit hit, waiting...")
                time.sleep(2 ** attempt)
            else:
                print(f"Error: {e}")
                
    return None
```

## Best Practices

### Voice Selection
```python
def select_voice_for_content(content_type):
    """Choose appropriate voice for content"""
    voice_map = {
        'news': 'Charon',        # Informative
        'story': 'Luna',         # Gentle, narrative
        'tutorial': 'Kore',      # Engaging
        'announcement': 'Helios', # Confident
        'meditation': 'Enceladus' # Breathy, calm
    }
    return voice_map.get(content_type, 'Kore')
```

### Text Preparation
```python
def prepare_text_for_tts(text):
    """Clean and format text for optimal TTS"""
    # Add pauses
    text = text.replace('.', '. ')
    text = text.replace(',', ', ')
    
    # Handle abbreviations
    text = text.replace('Dr.', 'Doctor')
    text = text.replace('Mr.', 'Mister')
    
    # Add emphasis markers if needed
    # text = text.replace('important', '*important*')
    
    return text
```

## Limitations
- Preview feature (may change)
- 32k token context limit
- Limited audio format options
- No direct SSML support
- Rate limits apply

## See Also
- [Audio Understanding](audio.md)
- [Text Generation](text-generation.md)
- [Native Audio (Live API)](live-api.md)