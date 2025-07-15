# Python Music Generation

*Generate instrumental music with Lyria RealTime*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs
Verified: 2025-01-14
Models: Lyria RealTime
Note: Available in Google AI Studio and Gemini API
-->

## Quick Reference
- **Model**: Lyria RealTime
- **Type**: Live instrumental music generation
- **Output**: Continuous audio stream
- **Control**: Text prompts for style/mood
- **Access**: Google AI Studio, Gemini API

## Basic Music Generation

```python
from google import genai
from google.genai import types

client = genai.Client()

# Generate instrumental music
response = client.models.generate_content(
    model="lyria-realtime",
    contents="Create upbeat electronic dance music",
    config=types.GenerateContentConfig(
        response_modalities=["AUDIO"],
    )
)

# Save generated music
for part in response.candidates[0].content.parts:
    if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
        audio_data = base64.b64decode(part.inline_data.data)
        with open('generated_music.mp3', 'wb') as f:
            f.write(audio_data)
```

## Style Control

```python
# Different music styles
music_prompts = {
    'ambient': "Calm ambient music with soft synthesizers",
    'jazz': "Smooth jazz with piano and saxophone feel",
    'classical': "Classical orchestral piece in major key",
    'electronic': "Energetic electronic music with strong bass",
    'lofi': "Lo-fi hip hop beats for studying",
    'cinematic': "Epic cinematic soundtrack with orchestral elements"
}

def generate_music_style(style, duration_hint=None):
    prompt = music_prompts.get(style, style)
    if duration_hint:
        prompt += f" for approximately {duration_hint}"
    
    response = client.models.generate_content(
        model="lyria-realtime",
        contents=prompt,
        config=types.GenerateContentConfig(
            response_modalities=["AUDIO"],
        )
    )
    
    return response
```

## Mood-Based Generation

```python
def generate_by_mood(mood, tempo=None, instruments=None):
    """Generate music based on mood and parameters"""
    
    prompt_parts = [f"{mood} instrumental music"]
    
    if tempo:
        prompt_parts.append(f"at {tempo} tempo")
    
    if instruments:
        prompt_parts.append(f"featuring {', '.join(instruments)}")
    
    prompt = " ".join(prompt_parts)
    
    response = client.models.generate_content(
        model="lyria-realtime",
        contents=prompt,
        config=types.GenerateContentConfig(
            response_modalities=["AUDIO"],
        )
    )
    
    return response

# Examples
happy_music = generate_by_mood("happy", tempo="upbeat", instruments=["piano", "strings"])
relaxing_music = generate_by_mood("relaxing", tempo="slow", instruments=["ambient pads"])
```

## Interactive Generation

```python
class MusicGenerator:
    def __init__(self, client):
        self.client = client
        self.current_style = None
        
    def generate_variation(self, base_prompt, variation_type):
        """Generate variations of a musical theme"""
        variations = {
            'faster': f"{base_prompt} but with increased tempo",
            'slower': f"{base_prompt} but more relaxed and slower",
            'intense': f"{base_prompt} with more energy and intensity",
            'minimal': f"{base_prompt} but stripped down and minimal",
            'complex': f"{base_prompt} with additional layers and complexity"
        }
        
        prompt = variations.get(variation_type, base_prompt)
        
        response = self.client.models.generate_content(
            model="lyria-realtime",
            contents=prompt,
            config=types.GenerateContentConfig(
                response_modalities=["AUDIO"],
            )
        )
        
        return response
    
    def generate_sequence(self, prompts):
        """Generate a sequence of musical pieces"""
        results = []
        
        for i, prompt in enumerate(prompts):
            print(f"Generating part {i+1}: {prompt}")
            response = self.client.models.generate_content(
                model="lyria-realtime",
                contents=prompt,
                config=types.GenerateContentConfig(
                    response_modalities=["AUDIO"],
                )
            )
            results.append(response)
            
        return results

# Usage
generator = MusicGenerator(client)

# Create variations
base = "Peaceful piano melody"
variations = [
    generator.generate_variation(base, 'faster'),
    generator.generate_variation(base, 'intense'),
    generator.generate_variation(base, 'minimal')
]
```

## Prompt Engineering

```python
def create_detailed_prompt(genre, mood, tempo, key=None, time_signature=None, elements=[]):
    """Build detailed music generation prompt"""
    
    prompt_parts = []
    
    # Basic style
    prompt_parts.append(f"{mood} {genre} instrumental music")
    
    # Technical details
    if tempo:
        prompt_parts.append(f"at {tempo} BPM")
    
    if key:
        prompt_parts.append(f"in {key}")
        
    if time_signature:
        prompt_parts.append(f"in {time_signature} time")
    
    # Musical elements
    if elements:
        element_str = ", ".join(elements)
        prompt_parts.append(f"featuring {element_str}")
    
    return " ".join(prompt_parts)

# Examples
prompt1 = create_detailed_prompt(
    genre="electronic",
    mood="energetic",
    tempo="128",
    elements=["synthesizers", "drum machine", "bass drops"]
)

prompt2 = create_detailed_prompt(
    genre="classical",
    mood="melancholic",
    tempo="60",
    key="D minor",
    elements=["strings", "piano", "subtle percussion"]
)
```

## Batch Generation

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def batch_generate_music(prompts, max_workers=3):
    """Generate multiple music pieces in parallel"""
    
    def generate_single(prompt, index):
        try:
            response = client.models.generate_content(
                model="lyria-realtime",
                contents=prompt,
                config=types.GenerateContentConfig(
                    response_modalities=["AUDIO"],
                )
            )
            return index, prompt, response
        except Exception as e:
            return index, prompt, None
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [executor.submit(generate_single, prompt, i) 
                  for i, prompt in enumerate(prompts)]
        results = [future.result() for future in futures]
    
    return results

# Generate music for different scenes
scene_prompts = [
    "Mysterious ambient music for a dark forest scene",
    "Uplifting orchestral music for a victory moment",
    "Tense electronic music for a chase sequence",
    "Romantic piano melody for an emotional scene"
]

results = batch_generate_music(scene_prompts)
```

## Save and Organize

```python
import os
from datetime import datetime

class MusicLibrary:
    def __init__(self, base_path="generated_music"):
        self.base_path = base_path
        os.makedirs(base_path, exist_ok=True)
        
    def save_music(self, response, style, tags=[]):
        """Save generated music with metadata"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        filename = f"{style}_{timestamp}.mp3"
        filepath = os.path.join(self.base_path, filename)
        
        # Extract and save audio
        for part in response.candidates[0].content.parts:
            if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
                audio_data = base64.b64decode(part.inline_data.data)
                with open(filepath, 'wb') as f:
                    f.write(audio_data)
                
                # Save metadata
                metadata = {
                    'filename': filename,
                    'style': style,
                    'tags': tags,
                    'created': timestamp,
                    'prompt': response.candidates[0].content.parts[0].text if hasattr(response.candidates[0].content.parts[0], 'text') else None
                }
                
                import json
                with open(filepath.replace('.mp3', '_metadata.json'), 'w') as f:
                    json.dump(metadata, f, indent=2)
                
                return filepath
        
        return None

# Usage
library = MusicLibrary()
response = generate_music_style('ambient')
library.save_music(response, 'ambient', tags=['relaxing', 'meditation'])
```

## Error Handling

```python
def safe_generate_music(prompt, retries=3):
    """Generate music with error handling"""
    
    for attempt in range(retries):
        try:
            response = client.models.generate_content(
                model="lyria-realtime",
                contents=prompt,
                config=types.GenerateContentConfig(
                    response_modalities=["AUDIO"],
                )
            )
            
            # Verify audio was generated
            has_audio = False
            for part in response.candidates[0].content.parts:
                if hasattr(part, 'inline_data') and 'audio' in part.inline_data.mime_type:
                    has_audio = True
                    break
            
            if has_audio:
                return response
            else:
                print(f"No audio generated for prompt: {prompt}")
                
        except Exception as e:
            if "RESOURCE_EXHAUSTED" in str(e):
                print(f"Rate limit hit, waiting...")
                time.sleep(2 ** attempt)
            else:
                print(f"Error generating music: {e}")
                
    return None
```

## Use Cases

### Background Music
```python
def generate_background_music(duration_hint, activity):
    """Generate appropriate background music"""
    prompts = {
        'work': f"Focused instrumental music for productivity, {duration_hint}",
        'meditation': f"Calm meditation music with nature sounds, {duration_hint}",
        'exercise': f"High-energy workout music with strong beat, {duration_hint}",
        'sleep': f"Gentle sleep music with soft ambient tones, {duration_hint}"
    }
    
    prompt = prompts.get(activity, f"Background music for {activity}, {duration_hint}")
    return safe_generate_music(prompt)
```

### Game Music
```python
def generate_game_music(scene_type):
    """Generate music for game scenes"""
    scenes = {
        'menu': "Ambient menu music with subtle melody",
        'battle': "Intense battle music with drums and orchestra",
        'exploration': "Adventurous exploration music with wonder",
        'boss': "Epic boss battle music with dramatic tension",
        'victory': "Triumphant victory fanfare"
    }
    
    prompt = scenes.get(scene_type, f"Game music for {scene_type}")
    return safe_generate_music(prompt)
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
- [Text Generation](text-generation.md)