# Video in JavaScript

*Video understanding and analysis with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision#video
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Video understanding, frame analysis, multimodal
-->

## Quick Reference
- **Input formats**: MP4, AVI, MOV, FLV, WebM, MPEG, 3GPP
- **Max duration**: 1 hour
- **Max file size**: 2GB (via File API)
- **Frame sampling**: Automatic at 1 FPS

## Basic Video Understanding

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});

// For videos under 20MB, use inline data
const videoFile = fs.readFileSync("video.mp4", { encoding: "base64" });

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "video/mp4",
        data: videoFile,
      },
    },
    { text: "Describe what happens in this video" },
  ],
});

console.log(response.text);
```

## Upload Large Videos (File API)

```javascript
import { GoogleGenAI, FileState } from "@google/genai";
import { GoogleAIFileManager } from "@google/genai/server";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});
const fileManager = new GoogleAIFileManager(process.env.GEMINI_API_KEY);

// Upload video file
const uploadResult = await fileManager.uploadFile("large-video.mp4", {
  mimeType: "video/mp4",
  displayName: "Large Video",
});

// Wait for processing
let file = await fileManager.getFile(uploadResult.file.name);
while (file.state === FileState.PROCESSING) {
  process.stdout.write(".");
  await new Promise(resolve => setTimeout(resolve, 10_000));
  file = await fileManager.getFile(uploadResult.file.name);
}

if (file.state === FileState.FAILED) {
  throw new Error("Video processing failed");
}

// Use uploaded video
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: uploadResult.file.uri,
        mimeType: uploadResult.file.mimeType,
      },
    },
    { text: "Analyze this video in detail" },
  ],
});

console.log(response.text);
```

## Video Analysis Tasks

```javascript
// Scene detection
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { 
      text: `Analyze this video and provide:
      1. Scene-by-scene breakdown with timestamps
      2. Key objects and people identified
      3. Actions taking place
      4. Camera movements and techniques
      5. Overall narrative or purpose`
    },
  ],
});

// Extract specific information
const response2 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { text: "List all text that appears in this video (signs, captions, etc.)" },
  ],
});
```

## Video with Timestamp Queries

```javascript
// Ask about specific moments
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { text: "What happens between 0:30 and 0:45 in this video?" },
  ],
});

// Generate timeline
const response2 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { text: "Create a timeline of major events in this video with timestamps" },
  ],
});
```

## Video Processing Utilities

```javascript
class VideoProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.fileManager = new GoogleAIFileManager(process.env.GEMINI_API_KEY);
  }

  async uploadVideo(videoPath, displayName) {
    console.log(`Uploading ${videoPath}...`);
    const uploadResult = await this.fileManager.uploadFile(videoPath, {
      mimeType: this.getMimeType(videoPath),
      displayName: displayName || videoPath,
    });

    // Wait for processing
    let file = await this.fileManager.getFile(uploadResult.file.name);
    while (file.state === FileState.PROCESSING) {
      process.stdout.write(".");
      await new Promise(resolve => setTimeout(resolve, 5000));
      file = await this.fileManager.getFile(uploadResult.file.name);
    }

    if (file.state === FileState.FAILED) {
      throw new Error("Video processing failed");
    }

    console.log("\nUpload complete!");
    return file;
  }

  getMimeType(filePath) {
    const ext = filePath.split('.').pop().toLowerCase();
    const mimeTypes = {
      'mp4': 'video/mp4',
      'avi': 'video/avi',
      'mov': 'video/quicktime',
      'webm': 'video/webm',
      'flv': 'video/x-flv',
      'mpeg': 'video/mpeg',
      '3gpp': 'video/3gpp',
    };
    return mimeTypes[ext] || 'video/mp4';
  }

  async analyzeVideo(videoFile, analysisType = 'general') {
    const prompts = {
      general: "Provide a comprehensive analysis of this video",
      educational: "Analyze this video as educational content: topics covered, teaching methods, key concepts",
      security: "Analyze this security footage: identify people, activities, unusual events",
      sports: "Analyze this sports video: plays, techniques, key moments",
      tutorial: "Break down this tutorial video into steps with timestamps",
    };

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          fileData: {
            fileUri: videoFile.uri,
            mimeType: videoFile.mimeType,
          },
        },
        { text: prompts[analysisType] || prompts.general },
      ],
    });

    return response.text;
  }

  async extractFrames(videoFile, query) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          fileData: {
            fileUri: videoFile.uri,
            mimeType: videoFile.mimeType,
          },
        },
        { text: `Describe the frames that show: ${query}` },
      ],
    });

    return response.text;
  }

  async generateCaptions(videoFile) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          fileData: {
            fileUri: videoFile.uri,
            mimeType: videoFile.mimeType,
          },
        },
        { 
          text: "Generate captions for this video with timestamps in SRT format"
        },
      ],
    });

    return response.text;
  }
}
```

## Common Use Cases

### Content Moderation
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { 
      text: "Check this video for inappropriate content, violence, or policy violations"
    },
  ],
  safetySettings: [
    {
      category: "HARM_CATEGORY_DANGEROUS",
      threshold: "BLOCK_LOW_AND_ABOVE",
    },
  ],
});
```

### Video Summarization
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { 
      text: "Create a 3-paragraph summary of this video suitable for a video description"
    },
  ],
});
```

### Action Recognition
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { 
      text: "List all human actions/activities shown in this video with timestamps"
    },
  ],
});
```

## Streaming Video Analysis

```javascript
// Stream response for detailed analysis
const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: videoFile.uri,
        mimeType: "video/mp4",
      },
    },
    { text: "Provide a detailed minute-by-minute breakdown of this video" },
  ],
});

for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: [videoContent, { text: prompt }],
  });
  return response.text;
} catch (error) {
  if (error.message.includes("exceeds the maximum")) {
    console.error("Video too large or too long");
  } else if (error.message.includes("Unsupported video format")) {
    console.error("Use MP4, AVI, MOV, WebM, etc.");
  } else if (error.message.includes("PROCESSING")) {
    console.error("Video still processing, please wait");
  } else {
    throw error;
  }
}
```

## Best Practices

1. **File Size**: Use File API for videos >20MB
2. **Duration**: Keep under 1 hour for best results
3. **Quality**: Higher quality improves analysis
4. **Formats**: MP4 most reliable
5. **Patience**: Large videos take time to process
6. **Cleanup**: Delete uploaded files when done

## See Also
- [Vision](vision.md) - Image analysis
- [Audio](audio.md) - Audio processing
- [File Handling](file-handling.md) - File uploads