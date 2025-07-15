# JavaScript Speech Generation (TTS)

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

```javascript
import { GoogleGenAI } from "@google/genai";
import fs from "fs";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

// Generate speech
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash-preview-tts",
  contents: "Welcome to the future of AI-powered speech synthesis!",
  config: {
    responseModalities: ["AUDIO"],
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: {
          voiceName: 'Kore',
        }
      }
    },
  }
});

// Save audio file
response.candidates[0].content.parts.forEach((part) => {
  if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
    const audioBuffer = Buffer.from(part.inlineData.data, 'base64');
    fs.writeFileSync('output.mp3', audioBuffer);
  }
});
```

## Voice Selection

```javascript
// Available voices with characteristics
const voices = {
  // Bright/Upbeat voices
  'Zephyr': 'Bright',
  'Puck': 'Upbeat',
  'Aura': 'Energetic',
  
  // Professional/Clear voices
  'Charon': 'Informative',
  'Kore': 'Engaging',
  'Helios': 'Confident',
  
  // Soft/Warm voices
  'Enceladus': 'Breathy',
  'Vega': 'Warm',
  'Luna': 'Gentle',
  
  // Deep/Authoritative voices
  'Orion': 'Deep',
  'Atlas': 'Authoritative',
  'Titan': 'Resonant'
};

// Generate with specific voice
async function generateWithVoice(text, voiceName) {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash-preview-tts",
    contents: text,
    config: {
      responseModalities: ["AUDIO"],
      speechConfig: {
        voiceConfig: {
          prebuiltVoiceConfig: {
            voiceName: voiceName,
          }
        }
      },
    }
  });
  
  return response;
}
```

## Natural Language Control

```javascript
// Control speech style through prompts
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash-preview-tts",
  contents: "Say cheerfully: Have a wonderful day!",
  config: {
    responseModalities: ["AUDIO"],
    speechConfig: {
      voiceConfig: {
        prebuiltVoiceConfig: {
          voiceName: 'Puck',  // Upbeat voice
        }
      }
    },
  }
});

// More style examples
const styles = [
  "Say sadly: I'll miss you",
  "Say excitedly: We won the championship!",
  "Say calmly: Everything will be alright",
  "Say mysteriously: The secret lies within",
  "Say urgently: We need to leave now!"
];

for (const style of styles) {
  await generateWithVoice(style, 'Kore');
}
```

## Multi-Speaker Dialog

```javascript
// Generate conversation with multiple voices
async function generateDialog(script) {
  const audioSegments = [];
  
  for (const line of script) {
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash-preview-tts",
      contents: line.text,
      config: {
        responseModalities: ["AUDIO"],
        speechConfig: {
          voiceConfig: {
            prebuiltVoiceConfig: {
              voiceName: line.speaker,
            }
          }
        },
      }
    });
    
    // Extract audio
    response.candidates[0].content.parts.forEach((part) => {
      if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
        audioSegments.push({
          speaker: line.speaker,
          audio: Buffer.from(part.inlineData.data, 'base64')
        });
      }
    });
  }
  
  return audioSegments;
}

// Example dialog
const script = [
  { speaker: 'Kore', text: 'Welcome to our podcast!' },
  { speaker: 'Orion', text: 'Thanks for having me.' },
  { speaker: 'Kore', text: "Let's dive into today's topic." }
];

const audioSegments = await generateDialog(script);
```

## Language Support

```javascript
// Multi-language generation (auto-detected)
const languages = {
  en: "Hello, how are you today?",
  es: "Hola, ¿cómo estás hoy?",
  fr: "Bonjour, comment allez-vous aujourd'hui?",
  de: "Hallo, wie geht es dir heute?",
  ja: "こんにちは、今日はどうですか？",
  zh: "你好，你今天怎么样？",
  hi: "नमस्ते, आज आप कैसे हैं?",
  ar: "مرحبا، كيف حالك اليوم؟"
};

async function generateMultilingual(texts, voiceName = 'Kore') {
  const results = {};
  
  for (const [lang, text] of Object.entries(texts)) {
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash-preview-tts",
      contents: text,  // Language auto-detected
      config: {
        responseModalities: ["AUDIO"],
        speechConfig: {
          voiceConfig: {
            prebuiltVoiceConfig: {
              voiceName: voiceName,
            }
          }
        },
      }
    });
    
    results[lang] = response;
  }
  
  return results;
}
```

## Advanced Speech Control

```javascript
// Control pace, tone, and emphasis
async function generateExpressiveSpeech(text, instructions) {
  // Combine instructions with text
  const prompt = `${instructions}: ${text}`;
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro-preview-tts",  // Pro for better control
    contents: prompt,
    config: {
      responseModalities: ["AUDIO"],
      speechConfig: {
        voiceConfig: {
          prebuiltVoiceConfig: {
            voiceName: 'Helios',
          }
        }
      },
    }
  });
  
  return response;
}

// Examples
const examples = [
  { text: "The quick brown fox", instructions: "Say slowly and clearly" },
  { text: "Breaking news!", instructions: "Say urgently with emphasis" },
  { text: "Once upon a time", instructions: "Say in a storytelling voice" },
  { text: "Thank you for your patience", instructions: "Say politely and warmly" }
];

for (const example of examples) {
  await generateExpressiveSpeech(example.text, example.instructions);
}
```

## Browser Usage

```javascript
// Generate TTS in browser
async function generateSpeechInBrowser(text, voiceName = 'Kore') {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash-preview-tts",
    contents: text,
    config: {
      responseModalities: ["AUDIO"],
      speechConfig: {
        voiceConfig: {
          prebuiltVoiceConfig: {
            voiceName: voiceName,
          }
        }
      },
    }
  });
  
  // Play audio in browser
  response.candidates[0].content.parts.forEach((part) => {
    if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
      const audioData = part.inlineData.data;
      const audioBlob = new Blob(
        [Uint8Array.from(atob(audioData), c => c.charCodeAt(0))],
        { type: 'audio/mp3' }
      );
      
      const audioUrl = URL.createObjectURL(audioBlob);
      const audio = new Audio(audioUrl);
      audio.play();
    }
  });
}
```

## Batch Processing

```javascript
async function processBatch(texts, voiceName = 'Kore', concurrency = 3) {
  const results = [];
  
  // Process in chunks for rate limiting
  for (let i = 0; i < texts.length; i += concurrency) {
    const chunk = texts.slice(i, i + concurrency);
    const promises = chunk.map(text => 
      generateWithVoice(text, voiceName)
        .then(response => ({ text, response, success: true }))
        .catch(error => ({ text, error, success: false }))
    );
    
    const chunkResults = await Promise.all(promises);
    results.push(...chunkResults);
    
    // Rate limiting delay
    if (i + concurrency < texts.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
  
  return results;
}

// Process multiple texts
const texts = [
  "Welcome to chapter one",
  "This is the introduction",
  "Let's begin our journey"
];

const results = await processBatch(texts);
```

## Save and Convert Audio

```javascript
import ffmpeg from 'fluent-ffmpeg';

async function saveAudio(response, filename, format = 'mp3') {
  for (const part of response.candidates[0].content.parts) {
    if (part.inlineData && part.inlineData.mimeType.includes('audio')) {
      const audioBuffer = Buffer.from(part.inlineData.data, 'base64');
      
      if (format === 'mp3') {
        fs.writeFileSync(`${filename}.mp3`, audioBuffer);
      } else {
        // Convert to other formats using ffmpeg
        await new Promise((resolve, reject) => {
          ffmpeg()
            .input(Buffer.from(audioBuffer))
            .output(`${filename}.${format}`)
            .on('end', resolve)
            .on('error', reject)
            .run();
        });
      }
      
      return `${filename}.${format}`;
    }
  }
}
```

## Error Handling

```javascript
async function safeGenerateSpeech(text, voiceName = 'Kore', retries = 3) {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      const response = await ai.models.generateContent({
        model: "gemini-2.5-flash-preview-tts",
        contents: text,
        config: {
          responseModalities: ["AUDIO"],
          speechConfig: {
            voiceConfig: {
              prebuiltVoiceConfig: {
                voiceName: voiceName,
              }
            }
          },
        }
      });
      
      return response;
      
    } catch (error) {
      if (error.message.includes("INVALID_ARGUMENT")) {
        console.error(`Invalid voice or text: ${error.message}`);
        break;
      } else if (error.message.includes("RESOURCE_EXHAUSTED")) {
        console.log(`Rate limit hit, waiting...`);
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
      } else {
        console.error(`Error: ${error.message}`);
      }
    }
  }
  
  return null;
}
```

## Voice Selection Helper

```javascript
function selectVoiceForContent(contentType) {
  const voiceMap = {
    'news': 'Charon',        // Informative
    'story': 'Luna',         // Gentle, narrative
    'tutorial': 'Kore',      // Engaging
    'announcement': 'Helios', // Confident
    'meditation': 'Enceladus' // Breathy, calm
  };
  
  return voiceMap[contentType] || 'Kore';
}

// Usage
const newsVoice = selectVoiceForContent('news');
await generateWithVoice("Breaking news update...", newsVoice);
```

## Text Preparation

```javascript
function prepareTextForTTS(text) {
  // Add pauses
  text = text.replace(/\./g, '. ');
  text = text.replace(/,/g, ', ');
  
  // Handle abbreviations
  text = text.replace(/Dr\./g, 'Doctor');
  text = text.replace(/Mr\./g, 'Mister');
  text = text.replace(/Ms\./g, 'Miss');
  
  // Handle special characters
  text = text.replace(/&/g, 'and');
  text = text.replace(/@/g, 'at');
  
  return text;
}
```

## Streaming Audio Player

```javascript
class TTSPlayer {
  constructor(ai) {
    this.ai = ai;
    this.queue = [];
    this.isPlaying = false;
  }
  
  async addText(text, voiceName = 'Kore') {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash-preview-tts",
      contents: text,
      config: {
        responseModalities: ["AUDIO"],
        speechConfig: {
          voiceConfig: {
            prebuiltVoiceConfig: {
              voiceName: voiceName,
            }
          }
        },
      }
    });
    
    this.queue.push(response);
    if (!this.isPlaying) {
      this.playNext();
    }
  }
  
  async playNext() {
    if (this.queue.length === 0) {
      this.isPlaying = false;
      return;
    }
    
    this.isPlaying = true;
    const response = this.queue.shift();
    
    // Play audio (browser example)
    // Implementation depends on environment
    
    // Continue with next
    setTimeout(() => this.playNext(), 100);
  }
}
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
- [Live API](live-api.md)