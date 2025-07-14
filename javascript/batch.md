# Batch in JavaScript

*Process multiple requests efficiently with Batch API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/batch
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Bulk processing, cost savings, async execution
-->

## Quick Reference
- **Method**: `ai.batches.create()`
- **Cost savings**: ~50% discount for batch requests
- **Processing time**: Minutes to hours depending on size
- **Use cases**: Large datasets, bulk analysis, batch jobs

## Basic Batch Processing

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Create batch job
const requests = [
  {
    model: "gemini-2.5-flash",
    contents: [{ text: "Summarize: The quick brown fox jumps over the lazy dog." }]
  },
  {
    model: "gemini-2.5-flash",
    contents: [{ text: "Translate to Spanish: Hello, how are you?" }]
  },
  {
    model: "gemini-2.5-flash",
    contents: [{ text: "Count the words in: Machine learning is fascinating." }]
  }
];

const batch = await ai.batches.create({ requests });
console.log(`Batch created: ${batch.name}`);

// Check status
let status = await ai.batches.get({ name: batch.name });
console.log(`Status: ${status.state}`);

// Wait for completion and get results
while (status.state === "PROCESSING") {
  await new Promise(resolve => setTimeout(resolve, 30000));
  status = await ai.batches.get({ name: batch.name });
}

if (status.state === "COMPLETED") {
  const results = await ai.batches.getResults({ name: batch.name });
  results.forEach((result, i) => {
    console.log(`Request ${i + 1}: ${result.response.text}`);
  });
}
```

## Large-Scale Text Processing

```javascript
class BatchProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async processDocuments(documents, taskTemplate, batchSize = 100) {
    const allResults = [];

    // Split into chunks
    for (let i = 0; i < documents.length; i += batchSize) {
      const chunk = documents.slice(i, i + batchSize);

      // Create batch requests
      const requests = chunk.map(doc => ({
        model: "gemini-2.5-flash",
        contents: [{ text: taskTemplate.replace("{document}", doc) }]
      }));

      // Submit batch
      const batch = await this.ai.batches.create({ requests });
      console.log(`Submitted batch ${Math.floor(i / batchSize) + 1}: ${batch.name}`);

      // Wait for completion
      const results = await this.waitForBatch(batch.name);
      allResults.push(...results);

      console.log(`Completed batch ${Math.floor(i / batchSize) + 1}`);
    }

    return allResults;
  }

  async waitForBatch(batchName, pollInterval = 60000) {
    while (true) {
      const status = await this.ai.batches.get({ name: batchName });

      if (status.state === "COMPLETED") {
        return await this.ai.batches.getResults({ name: batchName });
      } else if (status.state === "FAILED") {
        throw new Error(`Batch failed: ${status.error}`);
      }

      console.log(`Batch ${batchName} status: ${status.state}`);
      await new Promise(resolve => setTimeout(resolve, pollInterval));
    }
  }
}

// Usage
const processor = new BatchProcessor();

const documents = [
  "Document 1 content...",
  "Document 2 content...",
  // ... hundreds or thousands of documents
];

const taskTemplate = "Summarize this document in 2 sentences: {document}";

const results = await processor.processDocuments(documents, taskTemplate);
results.forEach((result, i) => {
  console.log(`Summary ${i + 1}: ${result.response.text}`);
});
```

## Data Analysis Batches

```javascript
class DataAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzeDataset(dataPoints, analysisTypes) {
    const requests = [];

    for (const dataPoint of dataPoints) {
      for (const analysisType of analysisTypes) {
        const prompt = this.getAnalysisPrompt(dataPoint, analysisType);

        const request = {
          model: "gemini-2.5-flash",
          contents: [{ text: prompt }],
          metadata: {
            data_id: dataPoint.id,
            analysis_type: analysisType
          }
        };
        requests.push(request);
      }
    }

    // Submit batch
    const batch = await this.ai.batches.create({ requests });

    // Wait for results
    const results = await this.waitForCompletion(batch.name);

    // Organize results
    return this.organizeResults(results);
  }

  getAnalysisPrompt(dataPoint, analysisType) {
    const prompts = {
      sentiment: `Analyze the sentiment of: ${dataPoint.text}`,
      themes: `Identify key themes in: ${dataPoint.text}`,
      entities: `Extract named entities from: ${dataPoint.text}`,
      summary: `Summarize: ${dataPoint.text}`
    };
    return prompts[analysisType] || `Analyze: ${dataPoint.text}`;
  }

  async waitForCompletion(batchName) {
    while (true) {
      const status = await this.ai.batches.get({ name: batchName });

      if (status.state === "COMPLETED") {
        return await this.ai.batches.getResults({ name: batchName });
      } else if (status.state === "FAILED") {
        throw new Error("Batch processing failed");
      }

      await new Promise(resolve => setTimeout(resolve, 30000));
    }
  }

  organizeResults(results) {
    const organized = {};

    for (const result of results) {
      const dataId = result.metadata.data_id;
      const analysisType = result.metadata.analysis_type;

      if (!organized[dataId]) {
        organized[dataId] = {};
      }

      organized[dataId][analysisType] = result.response.text;
    }

    return organized;
  }
}

// Usage
const analyzer = new DataAnalyzer();

const dataPoints = [
  { id: "1", text: "I love this new product! It's amazing." },
  { id: "2", text: "The service was disappointing and slow." },
  { id: "3", text: "Great quality but expensive pricing." }
];

const analysisTypes = ["sentiment", "themes", "entities"];

const results = await analyzer.analyzeDataset(dataPoints, analysisTypes);
for (const [dataId, analyses] of Object.entries(results)) {
  console.log(`Data point ${dataId}:`);
  for (const [analysisType, result] of Object.entries(analyses)) {
    console.log(`  ${analysisType}: ${result}`);
  }
}
```

## Content Generation Batches

```javascript
class ContentGenerator {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async generateContentVariations(baseContent, variationsConfig) {
    const requests = variationsConfig.map(variation => {
      const prompt = `
        Original content: ${baseContent}
        
        Generate a variation with these requirements:
        - Tone: ${variation.tone || 'neutral'}
        - Length: ${variation.length || 'medium'}
        - Target audience: ${variation.audience || 'general'}
        - Style: ${variation.style || 'standard'}
      `;

      return {
        model: "gemini-2.5-flash",
        contents: [{ text: prompt }],
        config: {
          temperature: variation.temperature || 0.7,
          maxOutputTokens: variation.maxTokens || 1000
        },
        metadata: {
          variation_id: variation.id
        }
      };
    });

    // Submit batch
    const batch = await this.ai.batches.create({ requests });

    // Wait and get results
    while (true) {
      const status = await this.ai.batches.get({ name: batch.name });
      if (status.state === "COMPLETED") {
        break;
      } else if (status.state === "FAILED") {
        throw new Error("Content generation failed");
      }
      await new Promise(resolve => setTimeout(resolve, 30000));
    }

    const results = await this.ai.batches.getResults({ name: batch.name });

    // Return organized results
    const variations = {};
    for (const result of results) {
      const variationId = result.metadata.variation_id;
      variations[variationId] = result.response.text;
    }

    return variations;
  }
}

// Usage
const generator = new ContentGenerator();

const baseContent = "Our new AI assistant helps businesses automate customer service.";

const variationsConfig = [
  { id: "formal", tone: "formal", audience: "executives" },
  { id: "casual", tone: "casual", audience: "general public" },
  { id: "technical", tone: "technical", audience: "developers" },
  { id: "marketing", tone: "enthusiastic", audience: "potential customers" }
];

const variations = await generator.generateContentVariations(baseContent, variationsConfig);

for (const [variationId, content] of Object.entries(variations)) {
  console.log(`${variationId.toUpperCase()} VERSION:`);
  console.log(content);
  console.log("-".repeat(50));
}
```

## Translation Batches

```javascript
async function batchTranslate(texts, targetLanguages) {
  const ai = new GoogleGenAI({});
  const requests = [];

  for (const text of texts) {
    for (const lang of targetLanguages) {
      const request = {
        model: "gemini-2.5-flash",
        contents: [{ text: `Translate to ${lang}: ${text}` }],
        metadata: {
          original_text: text,
          target_language: lang
        }
      };
      requests.push(request);
    }
  }

  // Submit batch
  const batch = await ai.batches.create({ requests });

  // Wait for completion
  while (true) {
    const status = await ai.batches.get({ name: batch.name });
    if (status.state === "COMPLETED") {
      break;
    }
    await new Promise(resolve => setTimeout(resolve, 30000));
  }

  // Organize results
  const results = await ai.batches.getResults({ name: batch.name });
  const translations = {};

  for (const result of results) {
    const original = result.metadata.original_text;
    const lang = result.metadata.target_language;

    if (!translations[original]) {
      translations[original] = {};
    }

    translations[original][lang] = result.response.text;
  }

  return translations;
}

// Usage
const texts = [
  "Hello, how are you?",
  "The weather is beautiful today.",
  "Thank you for your help."
];

const languages = ["Spanish", "French", "German", "Italian"];

const translations = await batchTranslate(texts, languages);

for (const [originalText, langTranslations] of Object.entries(translations)) {
  console.log(`Original: ${originalText}`);
  for (const [lang, translation] of Object.entries(langTranslations)) {
    console.log(`  ${lang}: ${translation}`);
  }
  console.log();
}
```

## Batch Status Monitoring

```javascript
class BatchMonitor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async monitorBatch(batchName, callback = null) {
    const startTime = new Date();

    while (true) {
      const status = await this.ai.batches.get({ name: batchName });

      const progressInfo = {
        batchName,
        state: status.state,
        completedRequests: status.completedRequests || 0,
        totalRequests: status.totalRequests || 0,
        elapsedTime: new Date() - startTime
      };

      if (callback) {
        callback(progressInfo);
      }

      if (["COMPLETED", "FAILED", "CANCELLED"].includes(status.state)) {
        return status;
      }

      await new Promise(resolve => setTimeout(resolve, 60000));
    }
  }

  async listActiveBatches() {
    const batches = await this.ai.batches.list();

    const activeBatches = [];
    for (const batch of batches) {
      if (["PROCESSING", "QUEUED"].includes(batch.state)) {
        activeBatches.push({
          name: batch.name,
          state: batch.state,
          created: batch.createTime,
          requests: batch.totalRequests || 'unknown'
        });
      }
    }

    return activeBatches;
  }
}

function progressCallback(info) {
  if (info.totalRequests > 0) {
    const progress = (info.completedRequests / info.totalRequests) * 100;
    console.log(`Batch ${info.batchName}: ${progress.toFixed(1)}% complete (${info.state})`);
  } else {
    console.log(`Batch ${info.batchName}: ${info.state}`);
  }
}

// Usage
const monitor = new BatchMonitor();

// Monitor a batch with progress updates
const finalStatus = await monitor.monitorBatch("batch_12345", progressCallback);
console.log(`Final status: ${finalStatus.state}`);

// List all active batches
const active = await monitor.listActiveBatches();
for (const batch of active) {
  console.log(`Active batch: ${batch.name} - ${batch.state}`);
}
```

## Cost Optimization

```javascript
class CostOptimizedBatch {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  estimateCost(requests) {
    let totalTokens = 0;

    for (const request of requests) {
      // Rough token estimation
      const text = request.contents?.[0]?.text || "";
      const estimatedTokens = text.split(" ").length * 1.3; // Rough estimate
      totalTokens += estimatedTokens;
    }

    // Batch API typically offers ~50% discount
    const regularCost = totalTokens * 0.000001; // Example rate
    const batchCost = regularCost * 0.5;

    return {
      estimatedTokens: totalTokens,
      regularCost,
      batchCost,
      savings: regularCost - batchCost
    };
  }

  optimizeBatchSize(allRequests, maxBatchSize = 1000) {
    const batches = [];

    for (let i = 0; i < allRequests.length; i += maxBatchSize) {
      const batchRequests = allRequests.slice(i, i + maxBatchSize);
      batches.push(batchRequests);
    }

    return batches;
  }

  async submitOptimizedBatches(allRequests) {
    const optimizedBatches = this.optimizeBatchSize(allRequests);
    const batchJobs = [];

    for (let i = 0; i < optimizedBatches.length; i++) {
      const batchRequests = optimizedBatches[i];
      const costEstimate = this.estimateCost(batchRequests);

      console.log(`Batch ${i + 1}: ${batchRequests.length} requests`);
      console.log(`  Estimated cost: $${costEstimate.batchCost.toFixed(4)}`);
      console.log(`  Estimated savings: $${costEstimate.savings.toFixed(4)}`);

      const batch = await this.ai.batches.create({ requests: batchRequests });
      batchJobs.push(batch.name);
    }

    return batchJobs;
  }
}

// Usage
const optimizer = new CostOptimizedBatch();

// Large set of requests
const requests = Array.from({ length: 2500 }, (_, i) => ({
  contents: [{ text: `Analyze sentiment: Sample text ${i}` }]
}));

// Get cost estimate
const costInfo = optimizer.estimateCost(requests);
console.log(`Total estimated savings: $${costInfo.savings.toFixed(2)}`);

// Submit optimized batches
const batchJobs = await optimizer.submitOptimizedBatches(requests);
console.log(`Submitted ${batchJobs.length} batch jobs`);
```

## Node.js Batch Processing

```javascript
import * as fs from "node:fs";
import * as path from "node:path";

class FileBatchProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async processFiles(directory, filePattern, task) {
    // Read files
    const files = fs.readdirSync(directory)
      .filter(file => file.match(filePattern))
      .map(file => ({
        name: file,
        content: fs.readFileSync(path.join(directory, file), "utf8")
      }));

    // Create batch requests
    const requests = files.map(file => ({
      model: "gemini-2.5-flash",
      contents: [{ text: `${task}: ${file.content}` }],
      metadata: { filename: file.name }
    }));

    // Process batch
    const batch = await this.ai.batches.create({ requests });

    // Wait for completion
    let status;
    do {
      await new Promise(resolve => setTimeout(resolve, 30000));
      status = await this.ai.batches.get({ name: batch.name });
      console.log(`Processing ${files.length} files: ${status.state}`);
    } while (status.state === "PROCESSING");

    // Get and save results
    if (status.state === "COMPLETED") {
      const results = await this.ai.batches.getResults({ name: batch.name });
      
      for (const result of results) {
        const filename = result.metadata.filename;
        const outputFile = path.join(directory, `processed_${filename}`);
        fs.writeFileSync(outputFile, result.response.text);
        console.log(`Saved result for ${filename}`);
      }
    }

    return status;
  }
}

// Usage
const processor = new FileBatchProcessor();

await processor.processFiles(
  "./documents",
  /\.txt$/,
  "Summarize this document"
);
```

## Error Handling

```javascript
try {
  const batch = await ai.batches.create({ requests });

  // Wait for completion
  while (true) {
    const status = await ai.batches.get({ name: batch.name });

    if (status.state === "COMPLETED") {
      const results = await ai.batches.getResults({ name: batch.name });
      break;
    } else if (status.state === "FAILED") {
      throw new Error(`Batch failed: ${status.error}`);
    } else if (status.state === "CANCELLED") {
      throw new Error("Batch was cancelled");
    }

    await new Promise(resolve => setTimeout(resolve, 30000));
  }

} catch (error) {
  if (error.message.includes("quota exceeded")) {
    console.log("Batch quota exceeded - try smaller batches");
  } else if (error.message.includes("invalid request")) {
    console.log("Check request format and model availability");
  } else {
    throw error;
  }
}
```

## Best Practices

1. **Batch Size**: Keep batches under 1000 requests for optimal processing
2. **Cost Savings**: Use for large-scale processing to get ~50% discount
3. **Monitoring**: Regularly check batch status and progress
4. **Error Handling**: Handle failed requests gracefully
5. **Organization**: Use metadata to track and organize results
6. **Optimization**: Group similar requests for better efficiency

## See Also
- [Text Generation](text-generation.md) - Individual requests
- [Cost Optimization](cost-optimization.md) - Reduce API costs
- [Rate Limits](rate-limits.md) - Understanding limits