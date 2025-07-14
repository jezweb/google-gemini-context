# Embeddings in Python

*Generate text embeddings for semantic search and similarity*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/embeddings
SDK: google-genai (Python)
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

```python
from google import genai

client = genai.Client()

# Generate embedding for single text
response = client.models.embed_content(
    model="text-embedding-004",
    content="The quick brown fox jumps over the lazy dog"
)

print(response.embedding)
# List of 768 float values
```

## Batch Embeddings

```python
# Generate embeddings for multiple texts
texts = [
    "Machine learning is a subset of AI",
    "Deep learning uses neural networks",
    "Natural language processing handles text"
]

embeddings = []
for text in texts:
    response = client.models.embed_content(
        model="text-embedding-004",
        content=text
    )
    embeddings.append(response.embedding)

print(f"Generated {len(embeddings)} embeddings")
```

## Semantic Search Implementation

```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

class SemanticSearch:
    def __init__(self):
        self.client = genai.Client()
        self.documents = []
        self.embeddings = []
    
    def add_documents(self, docs):
        """Add documents to search index"""
        for doc in docs:
            # Generate embedding
            response = self.client.models.embed_content(
                model="text-embedding-004",
                content=doc["text"]
            )
            
            self.documents.append(doc)
            self.embeddings.append(response.embedding)
    
    def search(self, query, top_k=5):
        """Search for most similar documents"""
        # Generate query embedding
        query_response = self.client.models.embed_content(
            model="text-embedding-004",
            content=query
        )
        query_embedding = np.array(query_response.embedding).reshape(1, -1)
        
        # Calculate similarities
        doc_embeddings = np.array(self.embeddings)
        similarities = cosine_similarity(query_embedding, doc_embeddings)[0]
        
        # Get top results
        top_indices = np.argsort(similarities)[::-1][:top_k]
        
        results = []
        for idx in top_indices:
            results.append({
                "document": self.documents[idx],
                "score": similarities[idx]
            })
        
        return results

# Usage
search = SemanticSearch()

search.add_documents([
    {"id": 1, "text": "Python is great for data science"},
    {"id": 2, "text": "JavaScript excels at web development"},
    {"id": 3, "text": "Machine learning requires good data"}
])

results = search.search("programming languages for AI")
for result in results:
    print(f"Score: {result['score']:.3f} - {result['document']['text']}")
```

## Document Clustering

```python
from sklearn.cluster import KMeans
import numpy as np

class DocumentClusterer:
    def __init__(self):
        self.client = genai.Client()
    
    def cluster_documents(self, documents, num_clusters=3):
        """Cluster documents using K-means"""
        # Generate embeddings
        embeddings = []
        for doc in documents:
            response = self.client.models.embed_content(
                model="text-embedding-004",
                content=doc
            )
            embeddings.append(response.embedding)
        
        # Perform clustering
        embeddings_array = np.array(embeddings)
        kmeans = KMeans(n_clusters=num_clusters, random_state=42)
        clusters = kmeans.fit_predict(embeddings_array)
        
        # Group documents by cluster
        grouped = {}
        for i, cluster_id in enumerate(clusters):
            if cluster_id not in grouped:
                grouped[cluster_id] = []
            grouped[cluster_id].append(documents[i])
        
        return grouped

# Usage
clusterer = DocumentClusterer()
docs = [
    "Python programming tutorial",
    "JavaScript web development",
    "Machine learning basics",
    "Data science with Python",
    "React frontend development",
    "Neural network fundamentals"
]

clusters = clusterer.cluster_documents(docs, num_clusters=3)
for cluster_id, docs in clusters.items():
    print(f"Cluster {cluster_id}:")
    for doc in docs:
        print(f"  - {doc}")
```

## Question Answering System

```python
class QASystem:
    def __init__(self):
        self.client = genai.Client()
        self.knowledge_base = []
    
    def add_knowledge(self, passages):
        """Add passages to knowledge base"""
        for passage in passages:
            response = self.client.models.embed_content(
                model="text-embedding-004",
                content=passage
            )
            
            self.knowledge_base.append({
                "text": passage,
                "embedding": response.embedding
            })
    
    def answer(self, question):
        """Answer question using knowledge base"""
        # Get question embedding
        question_response = self.client.models.embed_content(
            model="text-embedding-004",
            content=question
        )
        question_embedding = np.array(question_response.embedding).reshape(1, -1)
        
        # Find most relevant passages
        scores = []
        for item in self.knowledge_base:
            doc_embedding = np.array(item["embedding"]).reshape(1, -1)
            similarity = cosine_similarity(question_embedding, doc_embedding)[0][0]
            scores.append((item["text"], similarity))
        
        # Sort by relevance
        scores.sort(key=lambda x: x[1], reverse=True)
        
        # Use top passages as context
        context = "\n\n".join([item[0] for item in scores[:3]])
        
        # Generate answer
        prompt = f"""Context:\n{context}\n\nQuestion: {question}\n\nAnswer:"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=prompt
        )
        
        return {
            "answer": response.text,
            "sources": [item[0] for item in scores[:3]]
        }

# Usage
qa = QASystem()
qa.add_knowledge([
    "Python is a high-level programming language known for its simplicity.",
    "Machine learning is a subset of artificial intelligence.",
    "Neural networks are inspired by biological neural networks."
])

result = qa.answer("What is Python?")
print(result["answer"])
```

## Embedding Cache

```python
import pickle
import os

class EmbeddingCache:
    def __init__(self, cache_file="embeddings.pkl"):
        self.client = genai.Client()
        self.cache_file = cache_file
        self.cache = self.load_cache()
    
    def load_cache(self):
        """Load existing cache"""
        if os.path.exists(self.cache_file):
            with open(self.cache_file, "rb") as f:
                return pickle.load(f)
        return {}
    
    def save_cache(self):
        """Save cache to disk"""
        with open(self.cache_file, "wb") as f:
            pickle.dump(self.cache, f)
    
    def get_embedding(self, text):
        """Get embedding with caching"""
        if text in self.cache:
            return self.cache[text]
        
        # Generate new embedding
        response = self.client.models.embed_content(
            model="text-embedding-004",
            content=text
        )
        
        embedding = response.embedding
        self.cache[text] = embedding
        self.save_cache()
        
        return embedding
    
    def get_batch_embeddings(self, texts):
        """Get embeddings for multiple texts"""
        embeddings = []
        new_texts = []
        
        for text in texts:
            if text in self.cache:
                embeddings.append(self.cache[text])
            else:
                new_texts.append(text)
        
        # Generate embeddings for new texts
        for text in new_texts:
            response = self.client.models.embed_content(
                model="text-embedding-004",
                content=text
            )
            embedding = response.embedding
            self.cache[text] = embedding
            embeddings.append(embedding)
        
        self.save_cache()
        return embeddings
```

## Similarity Functions

```python
import numpy as np

def cosine_similarity(a, b):
    """Calculate cosine similarity between two vectors"""
    dot_product = np.dot(a, b)
    norm_a = np.linalg.norm(a)
    norm_b = np.linalg.norm(b)
    return dot_product / (norm_a * norm_b)

def euclidean_distance(a, b):
    """Calculate Euclidean distance"""
    return np.linalg.norm(np.array(a) - np.array(b))

def manhattan_distance(a, b):
    """Calculate Manhattan distance"""
    return np.sum(np.abs(np.array(a) - np.array(b)))

def find_most_similar(query_embedding, embeddings, texts, metric="cosine"):
    """Find most similar text using specified metric"""
    if metric == "cosine":
        similarities = [cosine_similarity(query_embedding, emb) for emb in embeddings]
        best_idx = np.argmax(similarities)
        return texts[best_idx], similarities[best_idx]
    elif metric == "euclidean":
        distances = [euclidean_distance(query_embedding, emb) for emb in embeddings]
        best_idx = np.argmin(distances)
        return texts[best_idx], distances[best_idx]
    else:
        raise ValueError("Unsupported metric")
```

## Error Handling

```python
try:
    response = client.models.embed_content(
        model="text-embedding-004",
        content=text
    )
    return response.embedding
except Exception as e:
    if "token limit" in str(e):
        print("Text too long (max 2048 tokens)")
        # Truncate and retry
        truncated = text[:8000]  # Approximate
        response = client.models.embed_content(
            model="text-embedding-004",
            content=truncated
        )
        return response.embedding
    else:
        raise
```

## Best Practices

1. **Batch Processing**: Process multiple texts together
2. **Caching**: Store embeddings to avoid regeneration
3. **Normalization**: Consider normalizing embeddings
4. **Token Limits**: Stay under 2048 tokens
5. **Quality**: Use clean, relevant text

## See Also
- [Text Generation](text-generation.md) - Generate text
- [Semantic Search](../examples/semantic-search.md) - Full implementation
- [Vector Databases](../examples/vector-databases.md) - Storage solutions