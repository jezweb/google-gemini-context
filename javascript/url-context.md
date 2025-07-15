# JavaScript URL Context

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

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

// Create URL context tool
const urlContextTool = {
  urlContext: {}
};

// Analyze a single URL
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Summarize the main points from https://example.com/article",
  config: {
    tools: [urlContextTool],
    responseModalities: ["TEXT"],
  }
});

console.log(response.text);
```

## Multiple URLs

```javascript
// Compare content from multiple URLs
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: `Compare the pricing plans from:
    - https://service1.com/pricing
    - https://service2.com/pricing
    - https://service3.com/pricing
    
    Create a comparison table with features and costs.`,
  config: {
    tools: [urlContextTool],
    responseModalities: ["TEXT"],
  }
});

console.log(response.text);
```

## Combined with Google Search

```javascript
// Use both URL context and Google Search
const urlTool = { urlContext: {} };
const searchTool = { googleSearch: {} };

const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `First search for recent JavaScript framework comparisons,
    then analyze the top 3 results in detail.`,
  config: {
    tools: [urlTool, searchTool],
    responseModalities: ["TEXT"],
  }
});

console.log(response.text);
```

## Data Extraction

```javascript
async function extractDataFromUrls(urls, extractionPrompt) {
  const urlContextTool = { urlContext: {} };
  
  // Build prompt with URLs
  let prompt = `${extractionPrompt}\n\nURLs to analyze:\n`;
  urls.forEach(url => {
    prompt += `- ${url}\n`;
  });
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt,
    config: {
      tools: [urlContextTool],
      responseModalities: ["TEXT"],
      responseMimeType: "application/json",  // Get structured output
    }
  });
  
  return JSON.parse(response.text);
}

// Example: Extract product information
const productUrls = [
  "https://store.com/product1",
  "https://store.com/product2",
  "https://store.com/product3"
];

const extractionPrompt = `Extract the following information from each product page:
- Product name
- Price
- Key features (list)
- Availability status

Return as JSON array.`;

const productData = await extractDataFromUrls(productUrls, extractionPrompt);
```

## News Aggregation

```javascript
async function aggregateNews(topic, newsUrls) {
  const urlContextTool = { urlContext: {} };
  
  const prompt = `Analyze these news articles about ${topic}:
${newsUrls.map(url => `- ${url}`).join('\n')}
    
Provide:
1. Common themes across all articles
2. Unique perspectives from each source
3. Overall summary of the situation
4. Key facts and figures mentioned`;
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: prompt,
    config: {
      tools: [urlContextTool],
      responseModalities: ["TEXT"],
    }
  });
  
  return response.text;
}

// Example usage
const techNews = [
  "https://techcrunch.com/article1",
  "https://theverge.com/article2",
  "https://wired.com/article3"
];

const summary = await aggregateNews("AI developments", techNews);
```

## Research Assistant Class

```javascript
class ResearchAssistant {
  constructor(ai) {
    this.ai = ai;
    this.urlTool = { urlContext: {} };
  }
  
  async analyzeTopic(topic, urls) {
    const prompt = `Research ${topic} using these sources:
${urls.map((url, i) => `${i + 1}. ${url}`).join('\n')}
        
Provide:
- Executive summary
- Key findings from each source
- Common agreements
- Contradictions or debates
- Gaps in information
- Recommendations for further research`;
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      config: {
        tools: [this.urlTool],
        responseModalities: ["TEXT"],
      }
    });
    
    return response.text;
  }
  
  async factCheck(claim, sources) {
    const prompt = `Fact-check this claim: "${claim}"
        
Using these sources:
${sources.map(url => `- ${url}`).join('\n')}
        
Determine:
1. Is the claim supported?
2. What evidence exists?
3. Are there contradictions?
4. Overall verdict with confidence level`;
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: prompt,
      config: {
        tools: [this.urlTool],
        responseModalities: ["TEXT"],
      }
    });
    
    return response.text;
  }
  
  async compareContent(urls, criteria) {
    const prompt = `Compare these websites based on ${criteria}:
${urls.map(url => `- ${url}`).join('\n')}

Provide detailed comparison including strengths and weaknesses of each.`;
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: prompt,
      config: {
        tools: [this.urlTool],
        responseModalities: ["TEXT"],
      }
    });
    
    return response.text;
  }
}

// Usage
const assistant = new ResearchAssistant(ai);

const climateUrls = [
  "https://climate.nasa.gov/evidence/",
  "https://www.ipcc.ch/report/ar6/wg1/",
  "https://www.noaa.gov/climate"
];

const research = await assistant.analyzeTopic("climate change impacts", climateUrls);
```

## Documentation Parser

```javascript
async function parseDocumentation(docUrls, query) {
  const urlContextTool = { urlContext: {} };
  
  const prompt = `Using these documentation pages:
${docUrls.map(url => `- ${url}`).join('\n')}
    
Answer this question: ${query}
    
Include:
- Direct answer
- Code examples if relevant
- Best practices mentioned
- Related topics to explore`;
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt,
    config: {
      tools: [urlContextTool],
      responseModalities: ["TEXT"],
    }
  });
  
  return response.text;
}

// Example: Parse API documentation
const apiDocs = [
  "https://api.example.com/docs/authentication",
  "https://api.example.com/docs/endpoints",
  "https://api.example.com/docs/errors"
];

const answer = await parseDocumentation(apiDocs, "How do I handle rate limiting?");
```

## Competitive Analysis

```javascript
async function competitiveAnalysis(companyUrls) {
  const urlContextTool = { urlContext: {} };
  
  const prompt = `Analyze these company websites:
${companyUrls.map(url => `- ${url}`).join('\n')}
    
Compare:
1. Product offerings
2. Pricing strategies
3. Target audience
4. Unique value propositions
5. Website user experience
    
Create a competitive analysis matrix.`;
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: prompt,
    config: {
      tools: [urlContextTool],
      responseModalities: ["TEXT"],
      responseMimeType: "application/json",
    }
  });
  
  return JSON.parse(response.text);
}
```

## Batch Processing

```javascript
async function batchAnalyzeUrls(urlGroups, analysisType) {
  const urlContextTool = { urlContext: {} };
  const results = [];
  
  for (const [groupName, urls] of Object.entries(urlGroups)) {
    try {
      // Limit to 20 URLs per request
      const limitedUrls = urls.slice(0, 20);
      
      const prompt = `Analyze these ${groupName} pages for ${analysisType}:\n${
        limitedUrls.map(url => `- ${url}`).join('\n')
      }`;
      
      const response = await ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: prompt,
        config: {
          tools: [urlContextTool],
          responseModalities: ["TEXT"],
        }
      });
      
      results.push({
        group: groupName,
        analysis: response.text,
        urlCount: urls.length,
        analyzed: limitedUrls.length
      });
      
    } catch (error) {
      results.push({
        group: groupName,
        error: error.message
      });
    }
  }
  
  return results;
}

// Example usage
const urlGroups = {
  techBlogs: [
    'https://techblog1.com/latest',
    'https://techblog2.com/trending'
  ],
  newsSites: [
    'https://news1.com/tech',
    'https://news2.com/innovation'
  ]
};

const analyses = await batchAnalyzeUrls(urlGroups, "latest AI trends");
```

## Web Scraping Alternative

```javascript
class WebAnalyzer {
  constructor(ai) {
    this.ai = ai;
    this.urlTool = { urlContext: {} };
  }
  
  async extractStructuredData(url, schema) {
    const prompt = `Extract data from ${url} according to this schema:
${JSON.stringify(schema, null, 2)}

Return the extracted data as valid JSON matching the schema.`;
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: prompt,
      config: {
        tools: [this.urlTool],
        responseModalities: ["TEXT"],
        responseMimeType: "application/json",
      }
    });
    
    return JSON.parse(response.text);
  }
  
  async monitorChanges(urls, previousData) {
    const prompt = `Check these URLs for changes:
${urls.map(url => `- ${url}`).join('\n')}

Compare with previous data and report:
1. What has changed
2. New information added
3. Information removed
4. Significance of changes`;
    
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: prompt,
      config: {
        tools: [this.urlTool],
        responseModalities: ["TEXT"],
      }
    });
    
    return response.text;
  }
}

// Usage
const analyzer = new WebAnalyzer(ai);

const schema = {
  title: "string",
  price: "number",
  features: ["string"],
  inStock: "boolean"
};

const data = await analyzer.extractStructuredData("https://example.com/product", schema);
```

## Error Handling

```javascript
async function safeUrlAnalysis(urls, prompt, fallbackResponse = "Unable to analyze URLs") {
  const urlContextTool = { urlContext: {} };
  
  try {
    // Validate URLs
    const validUrls = urls.filter(url => {
      try {
        const urlObj = new URL(url);
        return urlObj.protocol === 'http:' || urlObj.protocol === 'https:';
      } catch {
        console.log(`Invalid URL: ${url}`);
        return false;
      }
    });
    
    if (validUrls.length === 0) {
      return fallbackResponse;
    }
    
    // Limit to 20 URLs
    if (validUrls.length > 20) {
      console.log(`Limiting to first 20 URLs (provided: ${validUrls.length})`);
      validUrls.splice(20);
    }
    
    const fullPrompt = `${prompt}\n\nURLs:\n${validUrls.map(url => `- ${url}`).join('\n')}`;
    
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: fullPrompt,
      config: {
        tools: [urlContextTool],
        responseModalities: ["TEXT"],
      }
    });
    
    return response.text;
    
  } catch (error) {
    console.error(`Error analyzing URLs: ${error.message}`);
    return fallbackResponse;
  }
}
```

## Best Practices

### URL Validation
```javascript
function validateUrls(urls) {
  const validUrls = [];
  const invalidUrls = [];
  
  urls.forEach(url => {
    try {
      const urlObj = new URL(url);
      if (urlObj.protocol === 'http:' || urlObj.protocol === 'https:') {
        validUrls.push(url);
      } else {
        invalidUrls.push({ url, reason: 'Invalid protocol' });
      }
    } catch (error) {
      invalidUrls.push({ url, reason: 'Invalid URL format' });
    }
  });
  
  return { validUrls, invalidUrls };
}
```

### Progress Tracking
```javascript
async function analyzeUrlsWithProgress(urls, analysisPrompt, onProgress) {
  const urlContextTool = { urlContext: {} };
  const chunkSize = 20;
  const results = [];
  
  for (let i = 0; i < urls.length; i += chunkSize) {
    const chunk = urls.slice(i, i + chunkSize);
    const progress = Math.min(100, ((i + chunk.length) / urls.length) * 100);
    
    onProgress?.({
      current: i + chunk.length,
      total: urls.length,
      percentage: progress
    });
    
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `${analysisPrompt}\n\nBatch ${Math.floor(i/chunkSize) + 1} URLs:\n${
        chunk.map(url => `- ${url}`).join('\n')
      }`,
      config: {
        tools: [urlContextTool],
        responseModalities: ["TEXT"],
      }
    });
    
    results.push({
      batch: Math.floor(i/chunkSize) + 1,
      response: response.text
    });
  }
  
  return results;
}
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