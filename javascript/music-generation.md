# JavaScript Music Generation

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

```javascript
import { GoogleGenAI } from "@google/genai";
import fs from "fs";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

// Generate instrumental music
const response = await ai.models.generateContent({
  model: "lyria-realtime",
  contents: "Create upbeat electronic dance music",
  config: {
    responseModalities: ["AUDIO"],
  }
});

// Save generated music
response.candidates[0].content.parts.forEach((part) => {
  if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
    const audioBuffer = Buffer.from(part.inlineData.data, 'base64');
    fs.writeFileSync('generated_music.mp3', audioBuffer);
  }
});
```

## Style Control

```javascript
// Different music styles
const musicPrompts = {
  ambient: "Calm ambient music with soft synthesizers",
  jazz: "Smooth jazz with piano and saxophone feel",
  classical: "Classical orchestral piece in major key",
  electronic: "Energetic electronic music with strong bass",
  lofi: "Lo-fi hip hop beats for studying",
  cinematic: "Epic cinematic soundtrack with orchestral elements"
};

async function generateMusicStyle(style, durationHint = null) {
  let prompt = musicPrompts[style] || style;
  if (durationHint) {
    prompt += ` for approximately ${durationHint}`;
  }
  
  const response = await ai.models.generateContent({
    model: "lyria-realtime",
    contents: prompt,
    config: {
      responseModalities: ["AUDIO"],
    }
  });
  
  return response;
}

// Generate different styles
const ambientMusic = await generateMusicStyle('ambient');
const jazzMusic = await generateMusicStyle('jazz', '2 minutes');
```

## Mood-Based Generation

```javascript
async function generateByMood(mood, tempo = null, instruments = null) {
  const promptParts = [`${mood} instrumental music`];
  
  if (tempo) {
    promptParts.push(`at ${tempo} tempo`);
  }
  
  if (instruments) {
    promptParts.push(`featuring ${instruments.join(', ')}`);
  }
  
  const prompt = promptParts.join(' ');
  
  const response = await ai.models.generateContent({
    model: "lyria-realtime",
    contents: prompt,
    config: {
      responseModalities: ["AUDIO"],
    }
  });
  
  return response;
}

// Examples
const happyMusic = await generateByMood("happy", "upbeat", ["piano", "strings"]);
const relaxingMusic = await generateByMood("relaxing", "slow", ["ambient pads"]);
```

## Interactive Music Generator Class

```javascript
class MusicGenerator {
  constructor(ai) {
    this.ai = ai;
    this.currentStyle = null;
  }
  
  async generateVariation(basePrompt, variationType) {
    const variations = {
      faster: `${basePrompt} but with increased tempo`,
      slower: `${basePrompt} but more relaxed and slower`,
      intense: `${basePrompt} with more energy and intensity`,
      minimal: `${basePrompt} but stripped down and minimal`,
      complex: `${basePrompt} with additional layers and complexity`
    };
    
    const prompt = variations[variationType] || basePrompt;
    
    const response = await this.ai.models.generateContent({
      model: "lyria-realtime",
      contents: prompt,
      config: {
        responseModalities: ["AUDIO"],
      }
    });
    
    return response;
  }
  
  async generateSequence(prompts) {
    const results = [];
    
    for (let i = 0; i < prompts.length; i++) {
      console.log(`Generating part ${i + 1}: ${prompts[i]}`);
      
      const response = await this.ai.models.generateContent({
        model: "lyria-realtime",
        contents: prompts[i],
        config: {
          responseModalities: ["AUDIO"],
        }
      });
      
      results.push(response);
    }
    
    return results;
  }
}

// Usage
const generator = new MusicGenerator(ai);

// Create variations
const base = "Peaceful piano melody";
const variations = await Promise.all([
  generator.generateVariation(base, 'faster'),
  generator.generateVariation(base, 'intense'),
  generator.generateVariation(base, 'minimal')
]);
```

## Prompt Engineering

```javascript
function createDetailedPrompt(genre, mood, tempo, key = null, timeSignature = null, elements = []) {
  const promptParts = [];
  
  // Basic style
  promptParts.push(`${mood} ${genre} instrumental music`);
  
  // Technical details
  if (tempo) {
    promptParts.push(`at ${tempo} BPM`);
  }
  
  if (key) {
    promptParts.push(`in ${key}`);
  }
  
  if (timeSignature) {
    promptParts.push(`in ${timeSignature} time`);
  }
  
  // Musical elements
  if (elements.length > 0) {
    promptParts.push(`featuring ${elements.join(', ')}`);
  }
  
  return promptParts.join(' ');
}

// Examples
const prompt1 = createDetailedPrompt(
  "electronic",
  "energetic",
  "128",
  null,
  null,
  ["synthesizers", "drum machine", "bass drops"]
);

const prompt2 = createDetailedPrompt(
  "classical",
  "melancholic",
  "60",
  "D minor",
  "3/4",
  ["strings", "piano", "subtle percussion"]
);
```

## Browser-Based Music Player

```javascript
// For browser environments
class WebMusicPlayer {
  constructor(ai) {
    this.ai = ai;
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    this.currentAudio = null;
  }
  
  async generateAndPlay(prompt) {
    const response = await this.ai.models.generateContent({
      model: "lyria-realtime",
      contents: prompt,
      config: {
        responseModalities: ["AUDIO"],
      }
    });
    
    // Extract and play audio
    response.candidates[0].content.parts.forEach((part) => {
      if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
        const audioData = part.inlineData.data;
        const byteArray = Uint8Array.from(atob(audioData), c => c.charCodeAt(0));
        const blob = new Blob([byteArray], { type: 'audio/mp3' });
        const url = URL.createObjectURL(blob);
        
        // Create and play audio element
        if (this.currentAudio) {
          this.currentAudio.pause();
        }
        
        this.currentAudio = new Audio(url);
        this.currentAudio.play();
        
        // Add controls to page
        this.currentAudio.controls = true;
        document.body.appendChild(this.currentAudio);
      }
    });
  }
  
  stop() {
    if (this.currentAudio) {
      this.currentAudio.pause();
      this.currentAudio = null;
    }
  }
}
```

## Batch Processing

```javascript
async function batchGenerateMusic(prompts, concurrency = 3) {
  const results = [];
  
  // Process in chunks
  for (let i = 0; i < prompts.length; i += concurrency) {
    const chunk = prompts.slice(i, i + concurrency);
    
    const promises = chunk.map(async (prompt, index) => {
      try {
        const response = await ai.models.generateContent({
          model: "lyria-realtime",
          contents: prompt,
          config: {
            responseModalities: ["AUDIO"],
          }
        });
        
        return {
          index: i + index,
          prompt,
          response,
          success: true
        };
      } catch (error) {
        return {
          index: i + index,
          prompt,
          error,
          success: false
        };
      }
    });
    
    const chunkResults = await Promise.all(promises);
    results.push(...chunkResults);
    
    // Rate limiting
    if (i + concurrency < prompts.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
  
  return results;
}

// Generate music for different scenes
const scenePrompts = [
  "Mysterious ambient music for a dark forest scene",
  "Uplifting orchestral music for a victory moment",
  "Tense electronic music for a chase sequence",
  "Romantic piano melody for an emotional scene"
];

const results = await batchGenerateMusic(scenePrompts);
```

## Music Library Manager

```javascript
import path from 'path';

class MusicLibrary {
  constructor(basePath = 'generated_music') {
    this.basePath = basePath;
    if (!fs.existsSync(basePath)) {
      fs.mkdirSync(basePath, { recursive: true });
    }
  }
  
  async saveMusic(response, style, tags = []) {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const filename = `${style}_${timestamp}.mp3`;
    const filepath = path.join(this.basePath, filename);
    
    // Extract and save audio
    let saved = false;
    response.candidates[0].content.parts.forEach((part) => {
      if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
        const audioBuffer = Buffer.from(part.inlineData.data, 'base64');
        fs.writeFileSync(filepath, audioBuffer);
        saved = true;
        
        // Save metadata
        const metadata = {
          filename,
          style,
          tags,
          created: timestamp,
          size: audioBuffer.length
        };
        
        const metadataPath = filepath.replace('.mp3', '_metadata.json');
        fs.writeFileSync(metadataPath, JSON.stringify(metadata, null, 2));
      }
    });
    
    return saved ? filepath : null;
  }
  
  getLibrary() {
    const files = fs.readdirSync(this.basePath);
    return files
      .filter(file => file.endsWith('_metadata.json'))
      .map(file => {
        const metadata = JSON.parse(
          fs.readFileSync(path.join(this.basePath, file), 'utf8')
        );
        return metadata;
      });
  }
}

// Usage
const library = new MusicLibrary();
const response = await generateMusicStyle('ambient');
const savedPath = await library.saveMusic(response, 'ambient', ['relaxing', 'meditation']);
```

## Error Handling

```javascript
async function safeGenerateMusic(prompt, retries = 3) {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      const response = await ai.models.generateContent({
        model: "lyria-realtime",
        contents: prompt,
        config: {
          responseModalities: ["AUDIO"],
        }
      });
      
      // Verify audio was generated
      let hasAudio = false;
      response.candidates[0].content.parts.forEach((part) => {
        if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
          hasAudio = true;
        }
      });
      
      if (hasAudio) {
        return response;
      } else {
        console.log(`No audio generated for prompt: ${prompt}`);
      }
      
    } catch (error) {
      if (error.message.includes("RESOURCE_EXHAUSTED")) {
        console.log("Rate limit hit, waiting...");
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
      } else {
        console.error(`Error generating music: ${error.message}`);
      }
    }
  }
  
  return null;
}
```

## Use Case Functions

### Background Music
```javascript
async function generateBackgroundMusic(durationHint, activity) {
  const prompts = {
    work: `Focused instrumental music for productivity, ${durationHint}`,
    meditation: `Calm meditation music with nature sounds, ${durationHint}`,
    exercise: `High-energy workout music with strong beat, ${durationHint}`,
    sleep: `Gentle sleep music with soft ambient tones, ${durationHint}`
  };
  
  const prompt = prompts[activity] || `Background music for ${activity}, ${durationHint}`;
  return safeGenerateMusic(prompt);
}
```

### Game Music
```javascript
async function generateGameMusic(sceneType) {
  const scenes = {
    menu: "Ambient menu music with subtle melody",
    battle: "Intense battle music with drums and orchestra",
    exploration: "Adventurous exploration music with wonder",
    boss: "Epic boss battle music with dramatic tension",
    victory: "Triumphant victory fanfare"
  };
  
  const prompt = scenes[sceneType] || `Game music for ${sceneType}`;
  return safeGenerateMusic(prompt);
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
- [Text Generation](text-generation.md)