# JavaScript Image Generation

*Generate images using Imagen models via Gemini API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/imagen
Verified: 2025-01-14
Models: Imagen 3, Imagen 4, Imagen 4 Ultra
Note: Paid tier only, includes SynthID watermark
-->

## Quick Reference
- **Models**: imagen-3.0-generate-002, imagen-4.0-generate-001, imagen-4.0-ultra-generate-001
- **Pricing**: Imagen 3: $0.03, Imagen 4: $0.04 per image
- **Max prompt**: 480 tokens
- **Output**: PNG images with invisible watermark

## Basic Image Generation

```javascript
import { GoogleGenAI } from "@google/genai";
import fs from "fs";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

// Generate with Imagen 4 (recommended)
const response = await ai.models.generateImages({
  model: "imagen-4.0-generate-001",
  prompt: "A serene Japanese garden with cherry blossoms",
  config: {
    numberOfImages: 1,
  }
});

// Save image
response.generatedImages.forEach((generatedImage, index) => {
  const buffer = Buffer.from(generatedImage.image.imageBytes, 'base64');
  fs.writeFileSync(`japanese_garden_${index}.png`, buffer);
});
```

## Multiple Images

```javascript
// Generate multiple variations
const response = await ai.models.generateImages({
  model: "imagen-4.0-generate-001",
  prompt: "A futuristic cityscape at sunset",
  config: {
    numberOfImages: 4,  // Max 4 for Imagen 4
  }
});

// Save all images
response.generatedImages.forEach((generatedImage, index) => {
  const buffer = Buffer.from(generatedImage.image.imageBytes, 'base64');
  fs.writeFileSync(`cityscape_${index + 1}.png`, buffer);
  console.log(`Saved cityscape_${index + 1}.png`);
});
```

## Aspect Ratios

```javascript
// Available aspect ratios
const response = await ai.models.generateImages({
  model: "imagen-4.0-generate-001",
  prompt: "A majestic mountain landscape",
  config: {
    numberOfImages: 1,
    aspectRatio: "16:9"  // Widescreen
  }
});

// Aspect ratio options:
// - "1:1" (default) - Square
// - "4:3" - Fullscreen
// - "3:4" - Portrait fullscreen
// - "16:9" - Widescreen
// - "9:16" - Portrait widescreen
```

## Imagen 4 Ultra (High Precision)

```javascript
// Use Ultra for precise prompt following
const response = await ai.models.generateImages({
  model: "imagen-4.0-ultra-generate-001",
  prompt: `A steampunk owl wearing brass goggles and a top hat,
           perched on a Victorian street lamp at twilight,
           intricate mechanical details visible`,
  config: {
    numberOfImages: 1,  // Ultra only supports 1 image
  }
});

const buffer = Buffer.from(response.generatedImages[0].image.imageBytes, 'base64');
fs.writeFileSync('steampunk_owl_ultra.png', buffer);
```

## Advanced Prompting

```javascript
// Detailed prompt structure
async function generateWithStyle(subject, context, style, modifiers) {
  const prompt = `${subject} in ${context}, ${style} style, ${modifiers.join(', ')}`;
  
  const response = await ai.models.generateImages({
    model: "imagen-4.0-generate-001",
    prompt,
    config: {
      numberOfImages: 2,
    }
  });
  
  return response;
}

// Example usage
const result = await generateWithStyle(
  "a wise wizard",
  "an ancient library",
  "oil painting",
  ["dramatic lighting", "rich colors", "detailed brushwork"]
);
```

## Native Gemini Image Generation

```javascript
// Gemini 2.5 Flash can also generate images natively
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Create an image of a robot chef cooking in a modern kitchen",
  config: {
    responseModalities: ["TEXT", "IMAGE"],
  }
});

// Extract and save generated images
response.candidates[0].content.parts.forEach((part, index) => {
  if (part.inlineData && part.inlineData.mimeType.startsWith('image/')) {
    const buffer = Buffer.from(part.inlineData.data, 'base64');
    fs.writeFileSync(`robot_chef_${index}.png`, buffer);
  }
});
```

## Batch Generation

```javascript
async function batchGenerate(prompts) {
  const promises = prompts.map(async (prompt, index) => {
    try {
      const response = await ai.models.generateImages({
        model: "imagen-4.0-generate-001",
        prompt,
        config: {
          numberOfImages: 1,
        }
      });
      
      return { index, response, prompt };
    } catch (error) {
      return { index, error, prompt };
    }
  });
  
  return await Promise.all(promises);
}

// Usage
const prompts = [
  "A peaceful zen garden",
  "A bustling market street",
  "A cozy reading nook"
];

const results = await batchGenerate(prompts);
results.forEach(result => {
  if (result.response) {
    const buffer = Buffer.from(
      result.response.generatedImages[0].image.imageBytes, 
      'base64'
    );
    fs.writeFileSync(`batch_${result.index}.png`, buffer);
  }
});
```

## Browser Usage

```javascript
// For browser environments
async function generateAndDisplay(prompt) {
  const response = await ai.models.generateImages({
    model: "imagen-4.0-generate-001",
    prompt,
    config: {
      numberOfImages: 1,
    }
  });
  
  // Convert to blob and display
  const imageBytes = response.generatedImages[0].image.imageBytes;
  const byteArray = Uint8Array.from(atob(imageBytes), c => c.charCodeAt(0));
  const blob = new Blob([byteArray], { type: 'image/png' });
  const url = URL.createObjectURL(blob);
  
  // Display in img element
  const img = document.createElement('img');
  img.src = url;
  document.body.appendChild(img);
}
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateImages({
    model: "imagen-4.0-generate-001",
    prompt: "Generate an image",
    config: {
      numberOfImages: 1,
    }
  });
} catch (error) {
  if (error.message.includes("INVALID_ARGUMENT")) {
    console.error("Invalid prompt or parameters");
  } else if (error.message.includes("RESOURCE_EXHAUSTED")) {
    console.error("Rate limit exceeded");
  } else if (error.message.includes("PERMISSION_DENIED")) {
    console.error("Paid tier required for image generation");
  } else {
    console.error("Generation failed:", error.message);
  }
}
```

## Best Practices

### Prompt Engineering
```javascript
// Good prompt structure
const goodPrompt = `
A majestic lion, golden mane flowing in the wind,
standing on a rocky outcrop at sunset,
photorealistic style, dramatic lighting,
high detail, professional wildlife photography
`;

// Avoid vague prompts
// const badPrompt = "nice picture";
```

### Model Selection
```javascript
function chooseModel(requirements) {
  if (requirements.ultraPrecision) {
    return "imagen-4.0-ultra-generate-001";
  } else if (requirements.textRendering) {
    return "imagen-4.0-generate-001";  // Better text
  } else {
    return "imagen-3.0-generate-002";  // Lower cost
  }
}
```

### Progress Tracking
```javascript
async function generateWithProgress(prompt, onProgress) {
  onProgress({ status: 'starting', progress: 0 });
  
  try {
    onProgress({ status: 'generating', progress: 50 });
    
    const response = await ai.models.generateImages({
      model: "imagen-4.0-generate-001",
      prompt,
      config: { numberOfImages: 1 }
    });
    
    onProgress({ status: 'complete', progress: 100 });
    return response;
  } catch (error) {
    onProgress({ status: 'error', progress: 0, error });
    throw error;
  }
}
```

## Limitations
- Paid tier only (no free tier)
- 480 token prompt limit
- No image editing/inpainting
- All images include SynthID watermark
- Ultra model limited to 1 image

## See Also
- [Vision](vision.md) - Image understanding
- [Text Generation](text-generation.md)
- [Client Setup](client-setup.md)