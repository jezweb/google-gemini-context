# Embeddings in JavaScript

*Generate text embeddings for semantic search and similarity*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/embeddings
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: text-embedding-004
Key Features: 768-dimensional embeddings, semantic search
-->

## Quick Reference
- **Model**: `text-embedding-004`
- **Dimensions**: 768
- **Max input**: 2048 tokens
- **Use cases**: Semantic search, clustering, classification

## Basic Embedding Generation

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Generate embedding for single text
const response = await ai.models.embedContent({
  model: "text-embedding-004",
  content: "The quick brown fox jumps over the lazy dog"
});

console.log(response.embedding.values);
// Float array of 768 dimensions
```

## Batch Embeddings

```javascript
// Generate embeddings for multiple texts
const texts = [
  "Machine learning is a subset of AI",
  "Deep learning uses neural networks",
  "Natural language processing handles text"
];

const embeddings = await Promise.all(
  texts.map(text => 
    ai.models.embedContent({
      model: "text-embedding-004",
      content: text
    })
  )
);

// Extract embedding vectors
const vectors = embeddings.map(e => e.embedding.values);
```

## Semantic Search Implementation

```javascript
class SemanticSearch {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.documents = [];
    this.embeddings = [];
  }

  async addDocuments(docs) {
    // Generate embeddings for documents
    const newEmbeddings = await Promise.all(
      docs.map(doc => 
        this.ai.models.embedContent({
          model: "text-embedding-004",
          content: doc.text
        })
      )
    );

    // Store documents and embeddings
    docs.forEach((doc, i) => {
      this.documents.push(doc);
      this.embeddings.push(newEmbeddings[i].embedding.values);
    });
  }

  async search(query, topK = 5) {
    // Generate embedding for query
    const queryEmbedding = await this.ai.models.embedContent({
      model: "text-embedding-004",
      content: query
    });

    // Calculate similarities
    const similarities = this.embeddings.map((emb, i) => ({
      index: i,
      similarity: this.cosineSimilarity(
        queryEmbedding.embedding.values,
        emb
      )
    }));

    // Sort by similarity and return top K
    similarities.sort((a, b) => b.similarity - a.similarity);
    
    return similarities.slice(0, topK).map(s => ({
      document: this.documents[s.index],
      score: s.similarity
    }));
  }

  cosineSimilarity(a, b) {
    let dotProduct = 0;
    let normA = 0;
    let normB = 0;
    
    for (let i = 0; i < a.length; i++) {
      dotProduct += a[i] * b[i];
      normA += a[i] * a[i];
      normB += b[i] * b[i];
    }
    
    return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
  }
}

// Usage
const search = new SemanticSearch();

await search.addDocuments([
  { id: 1, text: "Python is great for data science" },
  { id: 2, text: "JavaScript excels at web development" },
  { id: 3, text: "Machine learning requires good data" }
]);

const results = await search.search("programming languages for AI");
console.log(results);
```

## Document Clustering

```javascript
class DocumentClusterer {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async clusterDocuments(documents, numClusters = 3) {
    // Generate embeddings
    const embeddings = await Promise.all(
      documents.map(doc =>
        this.ai.models.embedContent({
          model: "text-embedding-004",
          content: doc
        })
      )
    );

    const vectors = embeddings.map(e => e.embedding.values);
    
    // Simple k-means clustering
    const clusters = this.kMeans(vectors, numClusters);
    
    // Group documents by cluster
    const grouped = {};
    clusters.forEach((cluster, i) => {
      if (!grouped[cluster]) grouped[cluster] = [];
      grouped[cluster].push(documents[i]);
    });
    
    return grouped;
  }

  kMeans(vectors, k, maxIters = 100) {
    // Initialize random centroids
    const centroids = vectors.slice(0, k);
    let assignments = new Array(vectors.length);
    
    for (let iter = 0; iter < maxIters; iter++) {
      // Assign points to nearest centroid
      let changed = false;
      
      for (let i = 0; i < vectors.length; i++) {
        let minDist = Infinity;
        let nearest = 0;
        
        for (let j = 0; j < k; j++) {
          const dist = this.euclideanDistance(vectors[i], centroids[j]);
          if (dist < minDist) {
            minDist = dist;
            nearest = j;
          }
        }
        
        if (assignments[i] !== nearest) {
          changed = true;
          assignments[i] = nearest;
        }
      }
      
      if (!changed) break;
      
      // Update centroids
      for (let j = 0; j < k; j++) {
        const cluster = vectors.filter((_, i) => assignments[i] === j);
        if (cluster.length > 0) {
          centroids[j] = this.average(cluster);
        }
      }
    }
    
    return assignments;
  }

  euclideanDistance(a, b) {
    return Math.sqrt(
      a.reduce((sum, val, i) => sum + Math.pow(val - b[i], 2), 0)
    );
  }

  average(vectors) {
    const sum = new Array(vectors[0].length).fill(0);
    vectors.forEach(v => {
      v.forEach((val, i) => sum[i] += val);
    });
    return sum.map(s => s / vectors.length);
  }
}
```

## Question Answering System

```javascript
class QASystem {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.knowledgeBase = [];
  }

  async addKnowledge(passages) {
    for (const passage of passages) {
      const embedding = await this.ai.models.embedContent({
        model: "text-embedding-004",
        content: passage
      });
      
      this.knowledgeBase.push({
        text: passage,
        embedding: embedding.embedding.values
      });
    }
  }

  async answer(question) {
    // Find most relevant passages
    const questionEmb = await this.ai.models.embedContent({
      model: "text-embedding-004",
      content: question
    });

    // Rank passages by relevance
    const ranked = this.knowledgeBase.map(item => ({
      text: item.text,
      score: this.cosineSimilarity(
        questionEmb.embedding.values,
        item.embedding
      )
    })).sort((a, b) => b.score - a.score);

    // Use top passages as context
    const context = ranked.slice(0, 3).map(r => r.text).join('\n\n');
    
    // Generate answer using context
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Context:\n${context}\n\nQuestion: ${question}\n\nAnswer:`
    });
    
    return {
      answer: response.text,
      sources: ranked.slice(0, 3)
    };
  }

  cosineSimilarity(a, b) {
    // Same implementation as above
  }
}
```

## Caching Embeddings

```javascript
class EmbeddingCache {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.cache = new Map();
  }

  async getEmbedding(text) {
    // Check cache first
    if (this.cache.has(text)) {
      return this.cache.get(text);
    }

    // Generate new embedding
    const response = await this.ai.models.embedContent({
      model: "text-embedding-004",
      content: text
    });

    const embedding = response.embedding.values;
    this.cache.set(text, embedding);
    
    return embedding;
  }

  async getBatchEmbeddings(texts) {
    const results = [];
    const toGenerate = [];
    const indices = [];

    // Check cache
    texts.forEach((text, i) => {
      if (this.cache.has(text)) {
        results[i] = this.cache.get(text);
      } else {
        toGenerate.push(text);
        indices.push(i);
      }
    });

    // Generate missing embeddings
    if (toGenerate.length > 0) {
      const newEmbeddings = await Promise.all(
        toGenerate.map(text =>
          this.ai.models.embedContent({
            model: "text-embedding-004",
            content: text
          })
        )
      );

      // Cache and store results
      toGenerate.forEach((text, j) => {
        const embedding = newEmbeddings[j].embedding.values;
        this.cache.set(text, embedding);
        results[indices[j]] = embedding;
      });
    }

    return results;
  }

  saveCache(filename) {
    const data = Array.from(this.cache.entries());
    fs.writeFileSync(filename, JSON.stringify(data));
  }

  loadCache(filename) {
    const data = JSON.parse(fs.readFileSync(filename, 'utf8'));
    this.cache = new Map(data);
  }
}
```

## Error Handling

```javascript
try {
  const response = await ai.models.embedContent({
    model: "text-embedding-004",
    content: text
  });
  return response.embedding.values;
} catch (error) {
  if (error.message.includes("token limit")) {
    console.error("Text too long (max 2048 tokens)");
    // Truncate and retry
    const truncated = text.substring(0, 8000); // Approximate
    return await ai.models.embedContent({
      model: "text-embedding-004",
      content: truncated
    });
  }
  throw error;
}
```

## Best Practices

1. **Batch Processing**: Process multiple texts together
2. **Caching**: Store embeddings to avoid regeneration
3. **Normalization**: Consider normalizing embeddings
4. **Token Limits**: Stay under 2048 tokens
5. **Use Cases**: Match embedding model to your task

## See Also
- [Text Generation](text-generation.md) - Generate text
- [Semantic Search Guide](../examples/javascript/semantic-search.md)
- [Vector Databases](../examples/javascript/vector-db.md)