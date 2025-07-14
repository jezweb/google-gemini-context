# Batch in Python

*Process multiple requests efficiently with Batch API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/batch
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Bulk processing, cost savings, async execution
-->

## Quick Reference
- **Method**: `client.batches.create()`
- **Cost savings**: ~50% discount for batch requests
- **Processing time**: Minutes to hours depending on size
- **Use cases**: Large datasets, bulk analysis, batch jobs

## Basic Batch Processing

```python
from google import genai
import json

client = genai.Client()

# Create batch job
requests = [
    {
        "model": "gemini-2.5-flash",
        "contents": [{"text": "Summarize: The quick brown fox jumps over the lazy dog."}]
    },
    {
        "model": "gemini-2.5-flash", 
        "contents": [{"text": "Translate to Spanish: Hello, how are you?"}]
    },
    {
        "model": "gemini-2.5-flash",
        "contents": [{"text": "Count the words in: Machine learning is fascinating."}]
    }
]

batch = client.batches.create(requests=requests)
print(f"Batch created: {batch.name}")

# Check status
status = client.batches.get(name=batch.name)
print(f"Status: {status.state}")

# Wait for completion and get results
while status.state == "PROCESSING":
    time.sleep(30)
    status = client.batches.get(name=batch.name)

if status.state == "COMPLETED":
    results = client.batches.get_results(name=batch.name)
    for i, result in enumerate(results):
        print(f"Request {i+1}: {result.response.text}")
```

## Large-Scale Text Processing

```python
import time
from datetime import datetime

class BatchProcessor:
    def __init__(self):
        self.client = genai.Client()
    
    def process_documents(self, documents, task_template, batch_size=100):
        """Process large number of documents in batches"""
        all_results = []
        
        # Split into chunks
        for i in range(0, len(documents), batch_size):
            chunk = documents[i:i + batch_size]
            
            # Create batch requests
            requests = []
            for doc in chunk:
                request = {
                    "model": "gemini-2.5-flash",
                    "contents": [{"text": task_template.format(document=doc)}]
                }
                requests.append(request)
            
            # Submit batch
            batch = self.client.batches.create(requests=requests)
            print(f"Submitted batch {i//batch_size + 1}: {batch.name}")
            
            # Wait for completion
            results = self.wait_for_batch(batch.name)
            all_results.extend(results)
            
            print(f"Completed batch {i//batch_size + 1}")
        
        return all_results
    
    def wait_for_batch(self, batch_name, poll_interval=60):
        """Wait for batch completion and return results"""
        while True:
            status = self.client.batches.get(name=batch_name)
            
            if status.state == "COMPLETED":
                return self.client.batches.get_results(name=batch_name)
            elif status.state == "FAILED":
                raise Exception(f"Batch failed: {status.error}")
            
            print(f"Batch {batch_name} status: {status.state}")
            time.sleep(poll_interval)

# Usage
processor = BatchProcessor()

documents = [
    "Document 1 content...",
    "Document 2 content...",
    # ... hundreds or thousands of documents
]

task_template = "Summarize this document in 2 sentences: {document}"

results = processor.process_documents(documents, task_template)
for i, result in enumerate(results):
    print(f"Summary {i+1}: {result.response.text}")
```

## Data Analysis Batches

```python
class DataAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_dataset(self, data_points, analysis_types):
        """Analyze dataset with multiple analysis types"""
        requests = []
        
        for data_point in data_points:
            for analysis_type in analysis_types:
                prompt = self.get_analysis_prompt(data_point, analysis_type)
                
                request = {
                    "model": "gemini-2.5-flash",
                    "contents": [{"text": prompt}],
                    "metadata": {
                        "data_id": data_point["id"],
                        "analysis_type": analysis_type
                    }
                }
                requests.append(request)
        
        # Submit batch
        batch = self.client.batches.create(requests=requests)
        
        # Wait for results
        results = self.wait_for_completion(batch.name)
        
        # Organize results
        return self.organize_results(results)
    
    def get_analysis_prompt(self, data_point, analysis_type):
        """Generate analysis prompts"""
        prompts = {
            "sentiment": f"Analyze the sentiment of: {data_point['text']}",
            "themes": f"Identify key themes in: {data_point['text']}",
            "entities": f"Extract named entities from: {data_point['text']}",
            "summary": f"Summarize: {data_point['text']}"
        }
        return prompts.get(analysis_type, "Analyze: " + data_point['text'])
    
    def wait_for_completion(self, batch_name):
        """Wait for batch completion"""
        while True:
            status = self.client.batches.get(name=batch_name)
            
            if status.state == "COMPLETED":
                return self.client.batches.get_results(name=batch_name)
            elif status.state == "FAILED":
                raise Exception("Batch processing failed")
            
            time.sleep(30)
    
    def organize_results(self, results):
        """Organize results by data point and analysis type"""
        organized = {}
        
        for result in results:
            data_id = result.metadata["data_id"]
            analysis_type = result.metadata["analysis_type"]
            
            if data_id not in organized:
                organized[data_id] = {}
            
            organized[data_id][analysis_type] = result.response.text
        
        return organized

# Usage
analyzer = DataAnalyzer()

data_points = [
    {"id": "1", "text": "I love this new product! It's amazing."},
    {"id": "2", "text": "The service was disappointing and slow."},
    {"id": "3", "text": "Great quality but expensive pricing."}
]

analysis_types = ["sentiment", "themes", "entities"]

results = analyzer.analyze_dataset(data_points, analysis_types)
for data_id, analyses in results.items():
    print(f"Data point {data_id}:")
    for analysis_type, result in analyses.items():
        print(f"  {analysis_type}: {result}")
```

## Content Generation Batches

```python
class ContentGenerator:
    def __init__(self):
        self.client = genai.Client()
    
    def generate_content_variations(self, base_content, variations_config):
        """Generate multiple variations of content"""
        requests = []
        
        for variation in variations_config:
            prompt = f"""
            Original content: {base_content}
            
            Generate a variation with these requirements:
            - Tone: {variation.get('tone', 'neutral')}
            - Length: {variation.get('length', 'medium')}
            - Target audience: {variation.get('audience', 'general')}
            - Style: {variation.get('style', 'standard')}
            """
            
            request = {
                "model": "gemini-2.5-flash",
                "contents": [{"text": prompt}],
                "config": {
                    "temperature": variation.get('temperature', 0.7),
                    "max_output_tokens": variation.get('max_tokens', 1000)
                },
                "metadata": {
                    "variation_id": variation['id']
                }
            }
            requests.append(request)
        
        # Submit batch
        batch = self.client.batches.create(requests=requests)
        
        # Wait and get results
        while True:
            status = self.client.batches.get(name=batch.name)
            if status.state == "COMPLETED":
                break
            elif status.state == "FAILED":
                raise Exception("Content generation failed")
            time.sleep(30)
        
        results = self.client.batches.get_results(name=batch.name)
        
        # Return organized results
        variations = {}
        for result in results:
            variation_id = result.metadata["variation_id"]
            variations[variation_id] = result.response.text
        
        return variations

# Usage
generator = ContentGenerator()

base_content = "Our new AI assistant helps businesses automate customer service."

variations_config = [
    {"id": "formal", "tone": "formal", "audience": "executives"},
    {"id": "casual", "tone": "casual", "audience": "general public"},
    {"id": "technical", "tone": "technical", "audience": "developers"},
    {"id": "marketing", "tone": "enthusiastic", "audience": "potential customers"}
]

variations = generator.generate_content_variations(base_content, variations_config)

for variation_id, content in variations.items():
    print(f"{variation_id.upper()} VERSION:")
    print(content)
    print("-" * 50)
```

## Translation Batches

```python
def batch_translate(texts, target_languages):
    """Translate texts to multiple languages in batch"""
    client = genai.Client()
    requests = []
    
    for text in texts:
        for lang in target_languages:
            request = {
                "model": "gemini-2.5-flash",
                "contents": [{"text": f"Translate to {lang}: {text}"}],
                "metadata": {
                    "original_text": text,
                    "target_language": lang
                }
            }
            requests.append(request)
    
    # Submit batch
    batch = client.batches.create(requests=requests)
    
    # Wait for completion
    while True:
        status = client.batches.get(name=batch.name)
        if status.state == "COMPLETED":
            break
        time.sleep(30)
    
    # Organize results
    results = client.batches.get_results(name=batch.name)
    translations = {}
    
    for result in results:
        original = result.metadata["original_text"]
        lang = result.metadata["target_language"]
        
        if original not in translations:
            translations[original] = {}
        
        translations[original][lang] = result.response.text
    
    return translations

# Usage
texts = [
    "Hello, how are you?",
    "The weather is beautiful today.",
    "Thank you for your help."
]

languages = ["Spanish", "French", "German", "Italian"]

translations = batch_translate(texts, languages)

for original_text, lang_translations in translations.items():
    print(f"Original: {original_text}")
    for lang, translation in lang_translations.items():
        print(f"  {lang}: {translation}")
    print()
```

## Batch Status Monitoring

```python
class BatchMonitor:
    def __init__(self):
        self.client = genai.Client()
    
    def monitor_batch(self, batch_name, callback=None):
        """Monitor batch progress with optional callback"""
        start_time = datetime.now()
        
        while True:
            status = self.client.batches.get(name=batch_name)
            
            progress_info = {
                "batch_name": batch_name,
                "state": status.state,
                "completed_requests": getattr(status, 'completed_requests', 0),
                "total_requests": getattr(status, 'total_requests', 0),
                "elapsed_time": datetime.now() - start_time
            }
            
            if callback:
                callback(progress_info)
            
            if status.state in ["COMPLETED", "FAILED", "CANCELLED"]:
                return status
            
            time.sleep(60)
    
    def list_active_batches(self):
        """List all active batch jobs"""
        batches = self.client.batches.list()
        
        active_batches = []
        for batch in batches:
            if batch.state in ["PROCESSING", "QUEUED"]:
                active_batches.append({
                    "name": batch.name,
                    "state": batch.state,
                    "created": batch.create_time,
                    "requests": getattr(batch, 'total_requests', 'unknown')
                })
        
        return active_batches

def progress_callback(info):
    """Example progress callback"""
    if info['total_requests'] > 0:
        progress = (info['completed_requests'] / info['total_requests']) * 100
        print(f"Batch {info['batch_name']}: {progress:.1f}% complete ({info['state']})")
    else:
        print(f"Batch {info['batch_name']}: {info['state']}")

# Usage
monitor = BatchMonitor()

# Monitor a batch with progress updates
final_status = monitor.monitor_batch("batch_12345", callback=progress_callback)
print(f"Final status: {final_status.state}")

# List all active batches
active = monitor.list_active_batches()
for batch in active:
    print(f"Active batch: {batch['name']} - {batch['state']}")
```

## Cost Optimization

```python
class CostOptimizedBatch:
    def __init__(self):
        self.client = genai.Client()
    
    def estimate_cost(self, requests):
        """Estimate batch processing cost"""
        total_tokens = 0
        
        for request in requests:
            # Rough token estimation
            text = request.get("contents", [{}])[0].get("text", "")
            estimated_tokens = len(text.split()) * 1.3  # Rough estimate
            total_tokens += estimated_tokens
        
        # Batch API typically offers ~50% discount
        regular_cost = total_tokens * 0.000001  # Example rate
        batch_cost = regular_cost * 0.5
        
        return {
            "estimated_tokens": total_tokens,
            "regular_cost": regular_cost,
            "batch_cost": batch_cost,
            "savings": regular_cost - batch_cost
        }
    
    def optimize_batch_size(self, all_requests, max_batch_size=1000):
        """Split requests into optimal batch sizes"""
        batches = []
        
        for i in range(0, len(all_requests), max_batch_size):
            batch_requests = all_requests[i:i + max_batch_size]
            batches.append(batch_requests)
        
        return batches
    
    def submit_optimized_batches(self, all_requests):
        """Submit requests in optimized batches"""
        optimized_batches = self.optimize_batch_size(all_requests)
        batch_jobs = []
        
        for i, batch_requests in enumerate(optimized_batches):
            cost_estimate = self.estimate_cost(batch_requests)
            
            print(f"Batch {i+1}: {len(batch_requests)} requests")
            print(f"  Estimated cost: ${cost_estimate['batch_cost']:.4f}")
            print(f"  Estimated savings: ${cost_estimate['savings']:.4f}")
            
            batch = self.client.batches.create(requests=batch_requests)
            batch_jobs.append(batch.name)
        
        return batch_jobs

# Usage
optimizer = CostOptimizedBatch()

# Large set of requests
requests = [
    {"contents": [{"text": f"Analyze sentiment: Sample text {i}"}]}
    for i in range(2500)  # 2500 requests
]

# Get cost estimate
cost_info = optimizer.estimate_cost(requests)
print(f"Total estimated savings: ${cost_info['savings']:.2f}")

# Submit optimized batches
batch_jobs = optimizer.submit_optimized_batches(requests)
print(f"Submitted {len(batch_jobs)} batch jobs")
```

## Error Handling

```python
try:
    batch = client.batches.create(requests=requests)
    
    # Wait for completion
    while True:
        status = client.batches.get(name=batch.name)
        
        if status.state == "COMPLETED":
            results = client.batches.get_results(name=batch.name)
            break
        elif status.state == "FAILED":
            raise Exception(f"Batch failed: {status.error}")
        elif status.state == "CANCELLED":
            raise Exception("Batch was cancelled")
        
        time.sleep(30)

except Exception as error:
    if "quota exceeded" in str(error):
        print("Batch quota exceeded - try smaller batches")
    elif "invalid request" in str(error):
        print("Check request format and model availability")
    else:
        raise
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