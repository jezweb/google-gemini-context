# Caching in Python

*Context caching for efficient token usage*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/caching
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Context caching, reduced costs, faster responses
-->

## Quick Reference
- **Method**: `client.caches.create()`
- **TTL**: 5 minutes to 1 hour
- **Min tokens**: 32,768 tokens cached content
- **Use cases**: Long documents, repeated context

## Basic Context Caching

```python
from google import genai
import datetime

client = genai.Client()

# Create cache with long document
cache = client.caches.create(
    model="gemini-2.5-flash",
    contents=[
        {"text": "Long document content here..."},
        {"text": "More content that exceeds 32K tokens..."}
    ],
    ttl=datetime.timedelta(minutes=30)
)

print(f"Cache created: {cache.name}")

# Use cached content
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"text": "Based on the document, answer: What is the main topic?"}
    ],
    cached_content=cache.name
)

print(response.text)
```

## Document Analysis with Caching

```python
import pathlib
import datetime

class DocumentAnalyzer:
    def __init__(self):
        self.client = genai.Client()
        self.cache = None
    
    def load_document(self, file_path, cache_duration=30):
        """Load document and create cache"""
        content = pathlib.Path(file_path).read_text()
        
        # Create cache for large documents
        if len(content) > 100000:  # Approximate token threshold
            self.cache = self.client.caches.create(
                model="gemini-2.5-flash",
                contents=[{"text": content}],
                ttl=datetime.timedelta(minutes=cache_duration)
            )
            print(f"Document cached: {self.cache.name}")
        else:
            self.content = content
    
    def ask_question(self, question):
        """Ask question about cached document"""
        if self.cache:
            response = self.client.models.generate_content(
                model="gemini-2.5-flash",
                contents=[{"text": question}],
                cached_content=self.cache.name
            )
        else:
            response = self.client.models.generate_content(
                model="gemini-2.5-flash",
                contents=[
                    {"text": self.content},
                    {"text": question}
                ]
            )
        
        return response.text
    
    def cleanup(self):
        """Delete cache"""
        if self.cache:
            self.client.caches.delete(name=self.cache.name)
            print("Cache deleted")

# Usage
analyzer = DocumentAnalyzer()
analyzer.load_document("large_document.txt")

# Ask multiple questions efficiently
questions = [
    "What is the main theme?",
    "Who are the key characters?",
    "What is the conclusion?"
]

for question in questions:
    answer = analyzer.ask_question(question)
    print(f"Q: {question}")
    print(f"A: {answer}\n")

analyzer.cleanup()
```

## Cache Management

```python
class CacheManager:
    def __init__(self):
        self.client = genai.Client()
    
    def create_cache(self, content, model="gemini-2.5-flash", ttl_minutes=30):
        """Create new cache"""
        cache = self.client.caches.create(
            model=model,
            contents=content,
            ttl=datetime.timedelta(minutes=ttl_minutes)
        )
        return cache
    
    def list_caches(self):
        """List all caches"""
        caches = self.client.caches.list()
        for cache in caches:
            print(f"Cache: {cache.name}")
            print(f"  Model: {cache.model}")
            print(f"  Created: {cache.create_time}")
            print(f"  Expires: {cache.expire_time}")
            print(f"  Tokens: {cache.usage_metadata.total_token_count}")
            print()
        return caches
    
    def get_cache(self, cache_name):
        """Get cache details"""
        return self.client.caches.get(name=cache_name)
    
    def update_cache_ttl(self, cache_name, new_ttl_minutes):
        """Update cache TTL"""
        cache = self.client.caches.update(
            name=cache_name,
            ttl=datetime.timedelta(minutes=new_ttl_minutes)
        )
        return cache
    
    def delete_cache(self, cache_name):
        """Delete cache"""
        self.client.caches.delete(name=cache_name)
        print(f"Cache {cache_name} deleted")
    
    def cleanup_expired(self):
        """Clean up expired caches"""
        caches = self.client.caches.list()
        now = datetime.datetime.now()
        
        for cache in caches:
            if cache.expire_time < now:
                self.delete_cache(cache.name)
```

## Multi-Document Caching

```python
class MultiDocumentCache:
    def __init__(self):
        self.client = genai.Client()
        self.caches = {}
    
    def cache_documents(self, documents, ttl_minutes=30):
        """Cache multiple documents"""
        for doc_id, content in documents.items():
            cache = self.client.caches.create(
                model="gemini-2.5-flash",
                contents=[{"text": content}],
                ttl=datetime.timedelta(minutes=ttl_minutes)
            )
            self.caches[doc_id] = cache
            print(f"Cached document: {doc_id}")
    
    def cross_document_query(self, query, doc_ids=None):
        """Query across multiple cached documents"""
        if doc_ids is None:
            doc_ids = list(self.caches.keys())
        
        results = {}
        for doc_id in doc_ids:
            if doc_id in self.caches:
                response = self.client.models.generate_content(
                    model="gemini-2.5-flash",
                    contents=[{"text": query}],
                    cached_content=self.caches[doc_id].name
                )
                results[doc_id] = response.text
        
        return results
    
    def compare_documents(self, doc1_id, doc2_id, comparison_query):
        """Compare two cached documents"""
        # This would require special handling as caches are separate
        # For now, query each document individually
        results = {}
        
        for doc_id in [doc1_id, doc2_id]:
            if doc_id in self.caches:
                response = self.client.models.generate_content(
                    model="gemini-2.5-flash",
                    contents=[{"text": f"In this document, {comparison_query}"}],
                    cached_content=self.caches[doc_id].name
                )
                results[doc_id] = response.text
        
        return results
```

## Cost Optimization

```python
class CostOptimizedCache:
    def __init__(self):
        self.client = genai.Client()
        self.cache_stats = {}
    
    def create_cache_with_stats(self, content, cache_id, ttl_minutes=30):
        """Create cache and track usage"""
        cache = self.client.caches.create(
            model="gemini-2.5-flash",
            contents=content,
            ttl=datetime.timedelta(minutes=ttl_minutes)
        )
        
        self.cache_stats[cache_id] = {
            "cache": cache,
            "queries": 0,
            "tokens_saved": 0,
            "created": datetime.datetime.now()
        }
        
        return cache
    
    def query_with_tracking(self, cache_id, query):
        """Query cache and track savings"""
        if cache_id not in self.cache_stats:
            raise ValueError(f"Cache {cache_id} not found")
        
        cache_info = self.cache_stats[cache_id]
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[{"text": query}],
            cached_content=cache_info["cache"].name
        )
        
        # Update stats
        cache_info["queries"] += 1
        cache_info["tokens_saved"] += cache_info["cache"].usage_metadata.total_token_count
        
        return response.text
    
    def get_savings_report(self):
        """Generate cost savings report"""
        report = {}
        
        for cache_id, stats in self.cache_stats.items():
            report[cache_id] = {
                "queries": stats["queries"],
                "total_tokens_saved": stats["tokens_saved"],
                "estimated_cost_saved": stats["tokens_saved"] * 0.000001,  # Rough estimate
                "age_minutes": (datetime.datetime.now() - stats["created"]).total_seconds() / 60
            }
        
        return report
```

## Streaming with Cached Content

```python
def stream_with_cache(cache_name, query):
    """Stream responses using cached content"""
    client = genai.Client()
    
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[{"text": query}],
        cached_content=cache_name,
        config={"stream": True}
    )
    
    for chunk in response:
        print(chunk.text, end="", flush=True)
```

## Error Handling

```python
try:
    cache = client.caches.create(
        model="gemini-2.5-flash",
        contents=[{"text": content}],
        ttl=datetime.timedelta(minutes=30)
    )
except Exception as e:
    if "minimum token count" in str(e):
        print("Content too short for caching (need 32K+ tokens)")
    elif "maximum token count" in str(e):
        print("Content too long for caching")
    elif "TTL" in str(e):
        print("Invalid TTL (must be 5min-1hour)")
    else:
        raise
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