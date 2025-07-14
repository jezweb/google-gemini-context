# Caching in JavaScript

*Context caching for efficient token usage*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/caching
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Context caching, reduced costs, faster responses
-->

## Quick Reference
- **Method**: `ai.caches.create()`
- **TTL**: 5 minutes to 1 hour
- **Min tokens**: 32,768 tokens cached content
- **Use cases**: Long documents, repeated context

## Basic Context Caching

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Create cache with long document
const cache = await ai.caches.create({
  model: "gemini-2.5-flash",
  contents: [
    { text: "Long document content here..." },
    { text: "More content that exceeds 32K tokens..." }
  ],
  ttlSeconds: 1800 // 30 minutes
});

console.log(`Cache created: ${cache.name}`);

// Use cached content
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    { text: "Based on the document, answer: What is the main topic?" }
  ],
  cachedContent: cache.name
});

console.log(response.text);
```

## Document Analysis with Caching

```javascript
import * as fs from "node:fs";

class DocumentAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.cache = null;
  }

  async loadDocument(filePath, cacheDurationMinutes = 30) {
    const content = fs.readFileSync(filePath, "utf8");
    
    // Create cache for large documents
    if (content.length > 100000) { // Approximate token threshold
      this.cache = await this.ai.caches.create({
        model: "gemini-2.5-flash",
        contents: [{ text: content }],
        ttlSeconds: cacheDurationMinutes * 60
      });
      console.log(`Document cached: ${this.cache.name}`);
    } else {
      this.content = content;
    }
  }

  async askQuestion(question) {
    if (this.cache) {
      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: [{ text: question }],
        cachedContent: this.cache.name
      });
      return response.text;
    } else {
      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: [
          { text: this.content },
          { text: question }
        ]
      });
      return response.text;
    }
  }

  async cleanup() {
    if (this.cache) {
      await this.ai.caches.delete({ name: this.cache.name });
      console.log("Cache deleted");
    }
  }
}

// Usage
const analyzer = new DocumentAnalyzer();
await analyzer.loadDocument("large_document.txt");

// Ask multiple questions efficiently
const questions = [
  "What is the main theme?",
  "Who are the key characters?",
  "What is the conclusion?"
];

for (const question of questions) {
  const answer = await analyzer.askQuestion(question);
  console.log(`Q: ${question}`);
  console.log(`A: ${answer}\n`);
}

await analyzer.cleanup();
```

## Cache Management

```javascript
class CacheManager {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async createCache(content, model = "gemini-2.5-flash", ttlMinutes = 30) {
    const cache = await this.ai.caches.create({
      model,
      contents: content,
      ttlSeconds: ttlMinutes * 60
    });
    return cache;
  }

  async listCaches() {
    const caches = await this.ai.caches.list();
    
    caches.forEach(cache => {
      console.log(`Cache: ${cache.name}`);
      console.log(`  Model: ${cache.model}`);
      console.log(`  Created: ${cache.createTime}`);
      console.log(`  Expires: ${cache.expireTime}`);
      console.log(`  Tokens: ${cache.usageMetadata?.totalTokenCount}`);
      console.log();
    });
    
    return caches;
  }

  async getCache(cacheName) {
    return await this.ai.caches.get({ name: cacheName });
  }

  async updateCacheTTL(cacheName, newTtlMinutes) {
    const cache = await this.ai.caches.update({
      name: cacheName,
      ttlSeconds: newTtlMinutes * 60
    });
    return cache;
  }

  async deleteCache(cacheName) {
    await this.ai.caches.delete({ name: cacheName });
    console.log(`Cache ${cacheName} deleted`);
  }

  async cleanupExpired() {
    const caches = await this.ai.caches.list();
    const now = new Date();
    
    for (const cache of caches) {
      const expireTime = new Date(cache.expireTime);
      if (expireTime < now) {
        await this.deleteCache(cache.name);
      }
    }
  }
}
```

## Multi-Document Caching

```javascript
class MultiDocumentCache {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.caches = new Map();
  }

  async cacheDocuments(documents, ttlMinutes = 30) {
    for (const [docId, content] of Object.entries(documents)) {
      const cache = await this.ai.caches.create({
        model: "gemini-2.5-flash",
        contents: [{ text: content }],
        ttlSeconds: ttlMinutes * 60
      });
      
      this.caches.set(docId, cache);
      console.log(`Cached document: ${docId}`);
    }
  }

  async crossDocumentQuery(query, docIds = null) {
    if (!docIds) {
      docIds = Array.from(this.caches.keys());
    }

    const results = {};
    
    for (const docId of docIds) {
      if (this.caches.has(docId)) {
        const cache = this.caches.get(docId);
        const response = await this.ai.models.generateContent({
          model: "gemini-2.5-flash",
          contents: [{ text: query }],
          cachedContent: cache.name
        });
        results[docId] = response.text;
      }
    }

    return results;
  }

  async compareDocuments(doc1Id, doc2Id, comparisonQuery) {
    const results = {};
    
    for (const docId of [doc1Id, doc2Id]) {
      if (this.caches.has(docId)) {
        const cache = this.caches.get(docId);
        const response = await this.ai.models.generateContent({
          model: "gemini-2.5-flash",
          contents: [{ text: `In this document, ${comparisonQuery}` }],
          cachedContent: cache.name
        });
        results[docId] = response.text;
      }
    }

    return results;
  }
}
```

## Cost Optimization

```javascript
class CostOptimizedCache {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.cacheStats = new Map();
  }

  async createCacheWithStats(content, cacheId, ttlMinutes = 30) {
    const cache = await this.ai.caches.create({
      model: "gemini-2.5-flash",
      contents: content,
      ttlSeconds: ttlMinutes * 60
    });

    this.cacheStats.set(cacheId, {
      cache,
      queries: 0,
      tokensSaved: 0,
      created: new Date()
    });

    return cache;
  }

  async queryWithTracking(cacheId, query) {
    if (!this.cacheStats.has(cacheId)) {
      throw new Error(`Cache ${cacheId} not found`);
    }

    const cacheInfo = this.cacheStats.get(cacheId);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [{ text: query }],
      cachedContent: cacheInfo.cache.name
    });

    // Update stats
    cacheInfo.queries++;
    cacheInfo.tokensSaved += cacheInfo.cache.usageMetadata?.totalTokenCount || 0;

    return response.text;
  }

  getSavingsReport() {
    const report = {};
    
    for (const [cacheId, stats] of this.cacheStats) {
      const ageMinutes = (new Date() - stats.created) / (1000 * 60);
      
      report[cacheId] = {
        queries: stats.queries,
        totalTokensSaved: stats.tokensSaved,
        estimatedCostSaved: stats.tokensSaved * 0.000001, // Rough estimate
        ageMinutes: Math.round(ageMinutes)
      };
    }

    return report;
  }
}
```

## Streaming with Cached Content

```javascript
async function streamWithCache(cacheName, query) {
  const ai = new GoogleGenAI({});
  
  const response = await ai.models.generateContentStream({
    model: "gemini-2.5-flash",
    contents: [{ text: query }],
    cachedContent: cacheName
  });

  for await (const chunk of response) {
    process.stdout.write(chunk.text);
  }
}
```

## Automatic Cache Management

```javascript
class AutoCacheManager {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.activeCaches = new Map();
    this.cleanupInterval = null;
  }

  async getOrCreateCache(content, cacheId, ttlMinutes = 30) {
    // Check if cache already exists
    if (this.activeCaches.has(cacheId)) {
      const cachedInfo = this.activeCaches.get(cacheId);
      
      // Check if cache is still valid
      if (new Date() < cachedInfo.expireTime) {
        return cachedInfo.cache;
      } else {
        // Clean up expired cache
        this.activeCaches.delete(cacheId);
      }
    }

    // Create new cache
    const cache = await this.ai.caches.create({
      model: "gemini-2.5-flash",
      contents: content,
      ttlSeconds: ttlMinutes * 60
    });

    this.activeCaches.set(cacheId, {
      cache,
      expireTime: new Date(Date.now() + ttlMinutes * 60 * 1000)
    });

    return cache;
  }

  startAutoCleanup(intervalMinutes = 5) {
    this.cleanupInterval = setInterval(async () => {
      const now = new Date();
      
      for (const [cacheId, info] of this.activeCaches) {
        if (now > info.expireTime) {
          try {
            await this.ai.caches.delete({ name: info.cache.name });
            this.activeCaches.delete(cacheId);
            console.log(`Auto-deleted expired cache: ${cacheId}`);
          } catch (error) {
            console.error(`Error deleting cache ${cacheId}:`, error);
          }
        }
      }
    }, intervalMinutes * 60 * 1000);
  }

  stopAutoCleanup() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
      this.cleanupInterval = null;
    }
  }

  async cleanupAll() {
    for (const [cacheId, info] of this.activeCaches) {
      try {
        await this.ai.caches.delete({ name: info.cache.name });
      } catch (error) {
        console.error(`Error deleting cache ${cacheId}:`, error);
      }
    }
    this.activeCaches.clear();
  }
}
```

## Error Handling

```javascript
try {
  const cache = await ai.caches.create({
    model: "gemini-2.5-flash",
    contents: [{ text: content }],
    ttlSeconds: 1800
  });
} catch (error) {
  if (error.message.includes("minimum token count")) {
    console.error("Content too short for caching (need 32K+ tokens)");
  } else if (error.message.includes("maximum token count")) {
    console.error("Content too long for caching");
  } else if (error.message.includes("TTL")) {
    console.error("Invalid TTL (must be 5min-1hour)");
  } else {
    throw error;
  }
}
```

## Best Practices

1. **Token Minimum**: Only cache content >32K tokens
2. **TTL Management**: Set appropriate cache duration
3. **Cost Monitoring**: Track cache usage and savings
4. **Cleanup**: Delete caches when done
5. **Multi-Query**: Use cache for multiple questions
6. **Model Consistency**: Use same model for cache and queries

## See Also
- [Text Generation](text-generation.md) - Basic generation
- [Long Context](long-context.md) - Working with long inputs
- [Cost Optimization](cost-optimization.md) - Reduce API costs