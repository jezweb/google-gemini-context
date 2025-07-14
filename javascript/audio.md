# Audio in JavaScript

*Audio understanding and generation with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/audio
SDK: @google/genai (JavaScript)
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

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});

// Read audio file as base64
const audioFile = fs.readFileSync("speech.mp3", { encoding: "base64" });

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audioFile,
      },
    },
    { text: "Transcribe this audio and summarize the main points" },
  ],
});

console.log(response.text);
```

## Audio Transcription

```javascript
// Pure transcription
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/wav",
        data: audioFile,
      },
    },
    { text: "Transcribe this audio verbatim" },
  ],
});

console.log("Transcript:", response.text);
```

## Audio Analysis

```javascript
// Analyze audio content
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audioFile,
      },
    },
    { 
      text: `Analyze this audio and provide:
      1. Speaker identification
      2. Emotion/tone analysis
      3. Key topics discussed
      4. Audio quality assessment`
    },
  ],
});
```

## Multiple Audio Files

```javascript
const audio1 = fs.readFileSync("interview1.mp3", { encoding: "base64" });
const audio2 = fs.readFileSync("interview2.mp3", { encoding: "base64" });

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    { text: "Compare these two interviews" },
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audio1,
      },
    },
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audio2,
      },
    },
  ],
});
```

## Audio with Images

```javascript
const audioFile = fs.readFileSync("presentation.mp3", { encoding: "base64" });
const slideImage = fs.readFileSync("slide.jpg", { encoding: "base64" });

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audioFile,
      },
    },
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: slideImage,
      },
    },
    { text: "How does the audio narration relate to this slide?" },
  ],
});
```

## Streaming Audio Analysis

```javascript
// Stream response for long audio
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: longAudioFile,
      },
    },
    { text: "Provide a detailed transcript with timestamps" },
  ],
});

for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Audio Processing Utilities

```javascript
class AudioProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async transcribe(audioPath, options = {}) {
    const audioData = fs.readFileSync(audioPath, { encoding: "base64" });
    const mimeType = this.getMimeType(audioPath);
    
    const response = await this.ai.models.generateContent({
      model: options.model || "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType,
            data: audioData,
          },
        },
        { text: options.prompt || "Transcribe this audio" },
      ],
    });
    
    return response.text;
  }

  getMimeType(filePath) {
    const ext = filePath.split('.').pop().toLowerCase();
    const mimeTypes = {
      'mp3': 'audio/mp3',
      'wav': 'audio/wav',
      'flac': 'audio/flac',
      'aac': 'audio/aac',
      'ogg': 'audio/ogg',
      'opus': 'audio/opus',
    };
    return mimeTypes[ext] || 'audio/mp3';
  }

  async analyzePodcast(audioPath) {
    const audioData = fs.readFileSync(audioPath, { encoding: "base64" });
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "audio/mp3",
            data: audioData,
          },
        },
        { 
          text: `Analyze this podcast episode:
          - Title and topic
          - Guest speakers
          - Key discussion points
          - Notable quotes
          - Episode summary`
        },
      ],
    });
    
    return response.text;
  }
}
```

## Common Use Cases

### Meeting Transcription
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: meetingAudio,
      },
    },
    { 
      text: `Transcribe this meeting and provide:
      - Attendee list
      - Action items
      - Key decisions
      - Follow-up tasks`
    },
  ],
});
```

### Language Detection
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "audio/mp3",
        data: audioFile,
      },
    },
    { text: "What language is being spoken in this audio?" },
  ],
});
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: [audioContent, { text: prompt }],
  });
  return response.text;
} catch (error) {
  if (error.message.includes("File size exceeds")) {
    console.error("Audio file too large (max 20MB)");
  } else if (error.message.includes("Unsupported audio format")) {
    console.error("Use MP3, WAV, FLAC, AAC, OGG, or OPUS");
  } else {
    throw error;
  }
}
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