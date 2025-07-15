# Python URL Context

*Fetch and analyze web content directly in prompts*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/url-context
Verified: 2025-01-14
Models: Gemini 2.5 Pro/Flash/Flash-Lite, Gemini 2.0 Flash
Note: Experimental feature, 99.6% token reduction
-->

## Quick Reference
- **Tool**: URL Context
- **Feature**: Direct web page analysis
- **Benefit**: 99.6% token reduction
- **Limit**: Up to 20 URLs per request
- **Models**: Most Gemini models

## Basic URL Context

```python
from google import genai
from google.genai import types

client = genai.Client()

# Create URL context tool
url_context_tool = types.Tool(url_context=types.UrlContext())

# Analyze a single URL
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Summarize the main points from https://example.com/article",
    config=types.GenerateContentConfig(
        tools=[url_context_tool],
        response_modalities=["TEXT"],
    )
)

print(response.text)
```

## Multiple URLs

```python
# Compare content from multiple URLs
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="""Compare the pricing plans from:
    - https://service1.com/pricing
    - https://service2.com/pricing
    - https://service3.com/pricing
    
    Create a comparison table with features and costs.""",
    config=types.GenerateContentConfig(
        tools=[url_context_tool],
        response_modalities=["TEXT"],
    )
)

print(response.text)
```

## Combined with Google Search

```python
# Use both URL context and Google Search
url_tool = types.Tool(url_context=types.UrlContext())
search_tool = types.Tool(google_search=types.GoogleSearch())

response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""First search for recent Python web frameworks reviews,
    then analyze the top 3 results in detail.""",
    config=types.GenerateContentConfig(
        tools=[url_tool, search_tool],
        response_modalities=["TEXT"],
    )
)

print(response.text)
```

## Data Extraction

```python
def extract_data_from_urls(urls, extraction_prompt):
    """Extract structured data from multiple URLs"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    
    # Build prompt with URLs
    prompt = f"{extraction_prompt}\n\nURLs to analyze:\n"
    for url in urls:
        prompt += f"- {url}\n"
    
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt,
        config=types.GenerateContentConfig(
            tools=[url_context_tool],
            response_modalities=["TEXT"],
            response_mime_type="application/json",  # Get structured output
        )
    )
    
    return response.text

# Example: Extract product information
product_urls = [
    "https://store.com/product1",
    "https://store.com/product2",
    "https://store.com/product3"
]

extraction_prompt = """Extract the following information from each product page:
- Product name
- Price
- Key features (list)
- Availability status

Return as JSON array."""

product_data = extract_data_from_urls(product_urls, extraction_prompt)
```

## News Aggregation

```python
def aggregate_news(topic, news_urls):
    """Aggregate and summarize news from multiple sources"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    
    prompt = f"""Analyze these news articles about {topic}:
    {chr(10).join(f'- {url}' for url in news_urls)}
    
    Provide:
    1. Common themes across all articles
    2. Unique perspectives from each source
    3. Overall summary of the situation
    4. Key facts and figures mentioned"""
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=prompt,
        config=types.GenerateContentConfig(
            tools=[url_context_tool],
            response_modalities=["TEXT"],
        )
    )
    
    return response.text

# Example usage
tech_news = [
    "https://techcrunch.com/article1",
    "https://theverge.com/article2",
    "https://wired.com/article3"
]

summary = aggregate_news("AI developments", tech_news)
```

## Research Assistant

```python
class ResearchAssistant:
    def __init__(self, client):
        self.client = client
        self.url_tool = types.Tool(url_context=types.UrlContext())
        
    def analyze_topic(self, topic, urls):
        """Deep analysis of a topic from multiple sources"""
        
        prompt = f"""Research {topic} using these sources:
        {chr(10).join(f'{i+1}. {url}' for i, url in enumerate(urls))}
        
        Provide:
        - Executive summary
        - Key findings from each source
        - Common agreements
        - Contradictions or debates
        - Gaps in information
        - Recommendations for further research"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            config=types.GenerateContentConfig(
                tools=[self.url_tool],
                response_modalities=["TEXT"],
            )
        )
        
        return response.text
    
    def fact_check(self, claim, sources):
        """Fact-check a claim against multiple sources"""
        
        prompt = f"""Fact-check this claim: "{claim}"
        
        Using these sources:
        {chr(10).join(f'- {url}' for url in sources)}
        
        Determine:
        1. Is the claim supported?
        2. What evidence exists?
        3. Are there contradictions?
        4. Overall verdict with confidence level"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=prompt,
            config=types.GenerateContentConfig(
                tools=[self.url_tool],
                response_modalities=["TEXT"],
            )
        )
        
        return response.text

# Usage
assistant = ResearchAssistant(client)

climate_urls = [
    "https://climate.nasa.gov/evidence/",
    "https://www.ipcc.ch/report/ar6/wg1/",
    "https://www.noaa.gov/climate"
]

research = assistant.analyze_topic("climate change impacts", climate_urls)
```

## Documentation Parser

```python
def parse_documentation(doc_urls, query):
    """Parse technical documentation to answer queries"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    
    prompt = f"""Using these documentation pages:
    {chr(10).join(f'- {url}' for url in doc_urls)}
    
    Answer this question: {query}
    
    Include:
    - Direct answer
    - Code examples if relevant
    - Best practices mentioned
    - Related topics to explore"""
    
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt,
        config=types.GenerateContentConfig(
            tools=[url_context_tool],
            response_modalities=["TEXT"],
        )
    )
    
    return response.text

# Example: Parse API documentation
api_docs = [
    "https://api.example.com/docs/authentication",
    "https://api.example.com/docs/endpoints",
    "https://api.example.com/docs/errors"
]

answer = parse_documentation(api_docs, "How do I handle rate limiting?")
```

## Competitive Analysis

```python
def competitive_analysis(company_urls):
    """Analyze competitor websites"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    
    prompt = f"""Analyze these company websites:
    {chr(10).join(f'- {url}' for url in company_urls)}
    
    Compare:
    1. Product offerings
    2. Pricing strategies
    3. Target audience
    4. Unique value propositions
    5. Website user experience
    
    Create a competitive analysis matrix."""
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=prompt,
        config=types.GenerateContentConfig(
            tools=[url_context_tool],
            response_modalities=["TEXT"],
            response_mime_type="application/json",
        )
    )
    
    return response.text
```

## Batch Processing

```python
def batch_analyze_urls(url_groups, analysis_type):
    """Process multiple groups of URLs"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    results = []
    
    for group_name, urls in url_groups.items():
        prompt = f"Analyze these {group_name} pages for {analysis_type}:\n"
        prompt += "\n".join(f"- {url}" for url in urls[:20])  # Max 20 URLs
        
        try:
            response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt,
                config=types.GenerateContentConfig(
                    tools=[url_context_tool],
                    response_modalities=["TEXT"],
                )
            )
            
            results.append({
                'group': group_name,
                'analysis': response.text,
                'url_count': len(urls)
            })
            
        except Exception as e:
            results.append({
                'group': group_name,
                'error': str(e)
            })
    
    return results

# Example usage
url_groups = {
    'tech_blogs': [
        'https://techblog1.com/latest',
        'https://techblog2.com/trending'
    ],
    'news_sites': [
        'https://news1.com/tech',
        'https://news2.com/innovation'
    ]
}

analyses = batch_analyze_urls(url_groups, "latest AI trends")
```

## Error Handling

```python
def safe_url_analysis(urls, prompt, fallback_response="Unable to analyze URLs"):
    """Safely analyze URLs with error handling"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    
    try:
        # Validate URLs (basic check)
        valid_urls = []
        for url in urls:
            if url.startswith(('http://', 'https://')):
                valid_urls.append(url)
            else:
                print(f"Skipping invalid URL: {url}")
        
        if not valid_urls:
            return fallback_response
        
        # Limit to 20 URLs
        if len(valid_urls) > 20:
            print(f"Limiting to first 20 URLs (provided: {len(valid_urls)})")
            valid_urls = valid_urls[:20]
        
        response = client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"{prompt}\n\nURLs:\n" + "\n".join(f"- {url}" for url in valid_urls),
            config=types.GenerateContentConfig(
                tools=[url_context_tool],
                response_modalities=["TEXT"],
            )
        )
        
        return response.text
        
    except Exception as e:
        print(f"Error analyzing URLs: {e}")
        return fallback_response
```

## Best Practices

### URL Validation
```python
import re
from urllib.parse import urlparse

def validate_urls(urls):
    """Validate and clean URLs before analysis"""
    valid_urls = []
    
    for url in urls:
        try:
            # Parse URL
            parsed = urlparse(url)
            
            # Check if valid
            if parsed.scheme in ['http', 'https'] and parsed.netloc:
                valid_urls.append(url)
            else:
                print(f"Invalid URL format: {url}")
                
        except Exception as e:
            print(f"Error parsing URL {url}: {e}")
    
    return valid_urls
```

### Chunking for Large Sets
```python
def analyze_large_url_set(urls, prompt, chunk_size=20):
    """Process large sets of URLs in chunks"""
    
    url_context_tool = types.Tool(url_context=types.UrlContext())
    all_results = []
    
    # Process in chunks
    for i in range(0, len(urls), chunk_size):
        chunk = urls[i:i + chunk_size]
        chunk_prompt = f"{prompt}\n\nBatch {i//chunk_size + 1} URLs:\n"
        chunk_prompt += "\n".join(f"- {url}" for url in chunk)
        
        response = client.models.generate_content(
            model="gemini-2.5-flash",
            contents=chunk_prompt,
            config=types.GenerateContentConfig(
                tools=[url_context_tool],
                response_modalities=["TEXT"],
            )
        )
        
        all_results.append(response.text)
    
    return all_results
```

## Limitations
- Max 20 URLs per request
- Best with standard web pages
- Experimental feature (may change)
- Daily quotas apply
- Token reduction varies by content

## See Also
- [Grounding](grounding.md) - Google Search integration
- [Document Understanding](document-understanding.md)
- [Text Generation](text-generation.md)