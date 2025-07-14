# Vision in JavaScript

*Image understanding and generation with Gemini*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/vision
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New SDK syntax, updated image handling
-->

## Quick Reference
- **Image Input**: Base64 data or file upload
- **Max Size**: 20MB per image
- **Formats**: JPEG, PNG, WebP, HEIC, HEIF
- **Multiple Images**: Supported in single request

## Basic Image Understanding

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});

// Read image as base64
const base64ImageFile = fs.readFileSync("image.jpg", {
  encoding: "base64",
});

// Create contents array with image and text
const contents = [
  {
    inlineData: {
      mimeType: "image/jpeg",
      data: base64ImageFile,
    },
  },
  { text: "Describe this image in detail" },
];

// Generate response
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: contents,
});

console.log(response.text);
```

## Multiple Images

```javascript
// Read multiple images
const image1 = fs.readFileSync("image1.jpg", { encoding: "base64" });
const image2 = fs.readFileSync("image2.jpg", { encoding: "base64" });

const contents = [
  { text: "Compare these two images" },
  {
    inlineData: {
      mimeType: "image/jpeg",
      data: image1,
    },
  },
  {
    inlineData: {
      mimeType: "image/jpeg",
      data: image2,
    },
  },
];

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: contents,
});

console.log(response.text);
```

## Image from URL

```javascript
async function analyzeImageFromUrl(imageUrl, prompt) {
  // Fetch image from URL
  const response = await fetch(imageUrl);
  const imageArrayBuffer = await response.arrayBuffer();
  
  // Convert to base64
  const base64ImageData = Buffer.from(imageArrayBuffer).toString('base64');
  const mimeType = response.headers.get('content-type') || 'image/jpeg';
  
  // Create contents
  const contents = [
    {
      inlineData: {
        mimeType: mimeType,
        data: base64ImageData,
      },
    },
    { text: prompt }
  ];
  
  const result = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: contents,
  });
  
  return result.text;
}
```

## Image Generation

```javascript
// Note: Image generation may use a different endpoint/model
// This is a placeholder - check latest docs for image generation API

// For now, Gemini focuses on image understanding
// Image generation capabilities may be available through
// separate Imagen models or future updates
```

## Image + Text Chat

```javascript
// First message with image
const imageData = fs.readFileSync("chart.png", { encoding: "base64" });

const response1 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/png",
        data: imageData,
      },
    },
    { text: "What does this chart show?" }
  ],
});
console.log(response1.text);

// Follow-up with conversation history
const response2 = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    // Include previous conversation
    {
      inlineData: {
        mimeType: "image/png",
        data: imageData,
      },
    },
    { text: "What does this chart show?" },
    { role: "model", parts: [{ text: response1.text }] },
    { role: "user", parts: [{ text: "What's the highest value?" }] }
  ],
});
console.log(response2.text);
```

## Streaming with Images

```javascript
const imageData = fs.readFileSync("complex-diagram.png", { encoding: "base64" });

const response = await ai.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/png",
        data: imageData,
      },
    },
    { text: "Explain each component in this diagram" }
  ],
});

for await (const chunk of response) {
  process.stdout.write(chunk.text);
}
```

## Image Processing Utilities

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

class ImageProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async processImage(imagePath, mimeType = null) {
    // Auto-detect MIME type
    if (!mimeType) {
      const ext = imagePath.split('.').pop().toLowerCase();
      mimeType = {
        'jpg': 'image/jpeg',
        'jpeg': 'image/jpeg',
        'png': 'image/png',
        'webp': 'image/webp',
        'heic': 'image/heic',
        'heif': 'image/heif'
      }[ext] || 'image/jpeg';
    }

    const imageData = fs.readFileSync(imagePath, { encoding: "base64" });
    return {
      inlineData: {
        mimeType: mimeType,
        data: imageData,
      },
    };
  }

  async analyzeImage(imagePath, prompt) {
    const imagePart = await this.processImage(imagePath);
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [imagePart, { text: prompt }],
    });
    return response.text;
  }

  async compareImages(paths, prompt = "Compare these images") {
    const imageParts = await Promise.all(
      paths.map(path => this.processImage(path))
    );
    
    const contents = [{ text: prompt }, ...imageParts];
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: contents,
    });
    return response.text;
  }

  async extractText(imagePath) {
    const imagePart = await this.processImage(imagePath);
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        imagePart,
        { text: "Extract all text from this image. Format as plain text." }
      ],
    });
    return response.text;
  }
}
```

## Image Size Optimization

```javascript
import sharp from 'sharp'; // npm install sharp

async function optimizeImage(inputPath, maxWidth = 1024) {
  const metadata = await sharp(inputPath).metadata();
  
  if (metadata.width > maxWidth) {
    const buffer = await sharp(inputPath)
      .resize(maxWidth, null, { withoutEnlargement: true })
      .jpeg({ quality: 85 })
      .toBuffer();
    
    return {
      inlineData: {
        data: buffer.toString('base64'),
        mimeType: 'image/jpeg'
      }
    };
  }
  
  // Return original if no resize needed
  const imageData = fs.readFileSync(inputPath, { encoding: "base64" });
  return {
    inlineData: {
      data: imageData,
      mimeType: `image/${metadata.format}`
    }
  };
}
```

## Common Use Cases

### OCR/Text Extraction
```javascript
const imageData = fs.readFileSync("document.jpg", { encoding: "base64" });
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: imageData,
      },
    },
    { text: "Extract all text, maintaining the original formatting" }
  ],
});
console.log(response.text);
```

### Image Classification
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: imageData,
      },
    },
    { text: "Classify this image into categories: nature, urban, people, animals, food, or other" }
  ],
});
```

### Visual Q&A
```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: imageData,
      },
    },
    { text: "Is there a red car in this image? Answer yes or no." }
  ],
});
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: [imagePart, { text: prompt }],
  });
  return response.text;
} catch (error) {
  if (error.message.includes('File size exceeds maximum')) {
    console.error('Image too large, please reduce size');
  } else if (error.message.includes('Unsupported file type')) {
    console.error('Please use JPEG, PNG, WebP, HEIC, or HEIF');
  } else {
    throw error;
  }
}
```

## See Also
- [Audio](audio.md) - Audio processing
- [Video](video.md) - Video understanding
- [File Handling](file-handling.md) - File uploads