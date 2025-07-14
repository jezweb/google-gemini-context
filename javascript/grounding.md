# Grounding in JavaScript

*Ground responses with Google Search and external data sources*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/grounding
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Google Search integration, fact verification, citation support
-->

## Quick Reference
- **Tool**: `googleSearchRetrieval`
- **Use cases**: Current events, fact checking, research
- **Features**: Real-time search, automatic citations
- **Models**: All Gemini 2.5 models support grounding

## Basic Google Search Grounding

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Enable Google Search grounding
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "What are the latest developments in quantum computing in 2025?",
  tools: [{
    googleSearchRetrieval: {}
  }]
});

console.log(response.text);
// Response includes information from recent search results with citations
```

## Grounded Research Assistant

```javascript
class GroundedResearcher {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async researchTopic(topic, depth = "comprehensive") {
    const depthPrompts = {
      basic: `Provide a basic overview of: ${topic}`,
      comprehensive: `Provide a comprehensive analysis of: ${topic}. Include recent developments, key players, and current trends.`,
      technical: `Provide a technical deep-dive into: ${topic}. Include latest research, methodologies, and expert opinions.`,
      news: `What are the latest news and developments about: ${topic}?`
    };

    const prompt = depthPrompts[depth] || depthPrompts.comprehensive;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return {
      content: response.text,
      sources: this.extractSources(response)
    };
  }

  async factCheck(claim) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Fact-check this claim using current, reliable sources:
      
      Claim: ${claim}
      
      Provide:
      1. Verification status (True/False/Partially True/Unclear)
      2. Supporting evidence with sources
      3. Any contradictory information
      4. Confidence level in the assessment`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async compareSources(topic) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Research ${topic} from multiple perspectives and sources.
      
      Compare and contrast different viewpoints, identify areas of consensus
      and disagreement, and provide a balanced analysis.`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  extractSources(response) {
    const sources = [];
    if (response.citations) {
      for (const citation of response.citations) {
        sources.push({
          title: citation.title,
          url: citation.url,
          snippet: citation.snippet
        });
      }
    }
    return sources;
  }
}

// Usage
const researcher = new GroundedResearcher();

// Research current topic
const research = await researcher.researchTopic("AI safety regulations 2025", "comprehensive");
console.log("Research findings:");
console.log(research.content);
console.log(`\nSources: ${research.sources.length} citations`);

// Fact-check a claim
const factCheck = await researcher.factCheck("OpenAI released GPT-5 in January 2025");
console.log("\nFact check result:");
console.log(factCheck);

// Compare perspectives
const comparison = await researcher.compareSources("climate change policies");
console.log("\nMultiple source comparison:");
console.log(comparison);
```

## Current Events Assistant

```javascript
class CurrentEventsAssistant {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async getLatestNews(topic, region = "global") {
    const regionContext = region !== "global" ? ` in ${region}` : "";

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `What are the latest news and developments about ${topic}${regionContext}?
      
      Provide:
      1. Recent headlines and key events
      2. Timeline of important developments
      3. Analysis of trends and implications
      4. Key stakeholders and their positions`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async analyzeTrend(trend) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Analyze the current trend: ${trend}
      
      Include:
      1. Origin and timeline of the trend
      2. Current status and adoption
      3. Key drivers and influencers
      4. Potential future implications
      5. Regional variations if applicable`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async marketUpdate(marketSector) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Provide a current market update for ${marketSector}:
      
      Include:
      1. Recent price movements and trends
      2. Key market drivers
      3. Important news affecting the sector
      4. Analyst opinions and forecasts
      5. Risk factors to watch`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async eventContext(event) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Provide comprehensive context for this event: ${event}
      
      Include:
      1. Background and historical context
      2. Key players and stakeholders
      3. Timeline of events leading up to this
      4. Current status and recent developments
      5. Potential implications and next steps`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }
}

// Usage
const newsAssistant = new CurrentEventsAssistant();

// Get latest news
const news = await newsAssistant.getLatestNews("artificial intelligence regulation", "United States");
console.log("Latest AI regulation news:");
console.log(news);

// Analyze trend
const trendAnalysis = await newsAssistant.analyzeTrend("electric vehicle adoption");
console.log("\nEV trend analysis:");
console.log(trendAnalysis);

// Market update
const marketInfo = await newsAssistant.marketUpdate("cryptocurrency");
console.log("\nCrypto market update:");
console.log(marketInfo);
```

## Academic Research Assistant

```javascript
class AcademicResearcher {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async literatureReview(researchQuestion) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Conduct a literature review for this research question:
      ${researchQuestion}
      
      Provide:
      1. Overview of current research landscape
      2. Key theories and methodologies
      3. Recent significant studies and findings
      4. Gaps in current research
      5. Emerging trends and future directions
      6. Recommendations for further research`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async findExperts(field) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Identify current leading experts and researchers in ${field}:
      
      For each expert, provide:
      1. Name and current affiliation
      2. Key areas of expertise
      3. Recent notable publications or work
      4. Current projects or research focus
      5. How to find their work (websites, profiles)`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async researchMethodology(topic) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `What are the current best practices and methodologies for researching ${topic}?
      
      Include:
      1. Recommended research approaches
      2. Data collection methods
      3. Analysis techniques
      4. Tools and software commonly used
      5. Ethical considerations
      6. Recent methodological innovations`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async conferenceTracker(field) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Find upcoming conferences, workshops, and events in ${field}:
      
      For each event, provide:
      1. Event name and dates
      2. Location (physical/virtual)
      3. Key themes and tracks
      4. Notable speakers or organizers
      5. Submission deadlines if applicable
      6. Registration information`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }
}

// Usage
const academicResearcher = new AcademicResearcher();

// Conduct literature review
const literature = await academicResearcher.literatureReview("machine learning interpretability");
console.log("Literature review:");
console.log(literature);

// Find experts
const experts = await academicResearcher.findExperts("quantum computing");
console.log("\nQuantum computing experts:");
console.log(experts);

// Research methodology
const methodology = await academicResearcher.researchMethodology("climate change impact assessment");
console.log("\nResearch methodology:");
console.log(methodology);
```

## Business Intelligence Assistant

```javascript
class BusinessIntelligenceAssistant {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async marketAnalysis(companyOrIndustry) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Provide a comprehensive market analysis for ${companyOrIndustry}:
      
      Include:
      1. Current market size and growth trends
      2. Key competitors and market share
      3. Recent industry developments
      4. Regulatory environment and changes
      5. Technological disruptions
      6. Investment and funding activities
      7. Future outlook and opportunities`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async competitorIntelligence(company, competitors) {
    const competitorList = competitors.join(", ");

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Analyze ${company} against its competitors (${competitorList}):
      
      For each competitor, provide:
      1. Recent strategic moves and announcements
      2. Product launches and updates
      3. Financial performance highlights
      4. Market positioning changes
      5. Partnership and acquisition activities
      6. Strengths and weaknesses relative to ${company}`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async industryTrends(industry) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `What are the current and emerging trends in the ${industry} industry?
      
      Analyze:
      1. Technology trends and innovations
      2. Consumer behavior changes
      3. Regulatory and policy trends
      4. Investment and funding patterns
      5. Sustainability and ESG considerations
      6. Global vs regional variations
      7. Disruption threats and opportunities`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async startupLandscape(sector) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Map the current startup landscape in ${sector}:
      
      Include:
      1. Notable startups and their focus areas
      2. Recent funding rounds and valuations
      3. Key investors and accelerators
      4. Emerging business models
      5. Geographic distribution of activity
      6. Success stories and unicorns
      7. Challenges and failure patterns`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }
}

// Usage
const biAssistant = new BusinessIntelligenceAssistant();

// Market analysis
const market = await biAssistant.marketAnalysis("electric vehicle charging infrastructure");
console.log("Market analysis:");
console.log(market);

// Competitor intelligence
const competitors = await biAssistant.competitorIntelligence("Tesla", ["BYD", "Volkswagen", "General Motors"]);
console.log("\nCompetitor analysis:");
console.log(competitors);

// Industry trends
const trends = await biAssistant.industryTrends("fintech");
console.log("\nFintech trends:");
console.log(trends);
```

## Real-time Information Verification

```javascript
class InformationVerifier {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async verifyClaim(claim, context = "") {
    const contextText = context ? `Context: ${context}\n\n` : "";

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `${contextText}Verify this claim using reliable, current sources:
      
      Claim: ${claim}
      
      Assessment should include:
      1. Verification status (Verified/False/Partially True/Insufficient Evidence)
      2. Sources supporting or refuting the claim
      3. Any nuances or important context
      4. Confidence level in the assessment
      5. Date sensitivity (when was this true/false)`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async crossReferenceSources(topic) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Research ${topic} across multiple reliable sources and provide:
      
      1. Points of consensus across sources
      2. Areas of disagreement or contradiction
      3. Quality and reliability of different sources
      4. Most recent and authoritative information
      5. Any bias or perspective differences
      6. Recommended primary sources for further research`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }

  async temporalVerification(claim, timeFrame) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Analyze how this claim has evolved over ${timeFrame}:
      
      Claim: ${claim}
      
      Provide:
      1. Historical accuracy of the claim
      2. Key events that changed its validity
      3. Current status of the claim
      4. Timeline of relevant developments
      5. Factors that might affect future validity`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return response.text;
  }
}

// Usage
const verifier = new InformationVerifier();

// Verify a claim
const verification = await verifier.verifyClaim(
  "China is the world's largest producer of electric vehicles",
  "As of 2025"
);
console.log("Claim verification:");
console.log(verification);

// Cross-reference sources
const crossRef = await verifier.crossReferenceSources("effectiveness of COVID-19 vaccines");
console.log("\nCross-reference analysis:");
console.log(crossRef);

// Temporal verification
const temporal = await verifier.temporalVerification(
  "Remote work is the dominant work model",
  "the past 5 years"
);
console.log("\nTemporal verification:");
console.log(temporal);
```

## Web-based Grounded Research

```javascript
// Client-side grounded research
class WebGroundedResearcher {
  constructor(apiKey) {
    this.ai = new GoogleGenAI({ apiKey });
  }

  async researchWithProgress(topic, onProgress) {
    const response = await this.ai.models.generateContentStream({
      model: "gemini-2.5-flash",
      contents: `Provide comprehensive, current information about: ${topic}`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    let fullContent = "";
    for await (const chunk of response) {
      fullContent += chunk.text;
      onProgress({
        type: 'content',
        chunk: chunk.text,
        fullContent
      });
    }

    onProgress({
      type: 'complete',
      fullContent
    });

    return fullContent;
  }

  async factCheckWithSources(claim) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Fact-check this claim with sources: ${claim}`,
      tools: [{
        googleSearchRetrieval: {}
      }]
    });

    return {
      result: response.text,
      citations: this.extractCitations(response)
    };
  }

  extractCitations(response) {
    // Extract citations from response
    const citations = [];
    if (response.citations) {
      for (const citation of response.citations) {
        citations.push({
          title: citation.title,
          url: citation.url,
          snippet: citation.snippet
        });
      }
    }
    return citations;
  }
}

// Usage in browser
const researcher = new WebGroundedResearcher(API_KEY);

// Research with progress updates
await researcher.researchWithProgress(
  "latest renewable energy breakthroughs",
  (update) => {
    if (update.type === 'content') {
      document.getElementById('research-output').innerHTML += update.chunk;
    } else if (update.type === 'complete') {
      document.getElementById('status').textContent = 'Research complete';
    }
  }
);

// Fact-check with source display
const factCheck = await researcher.factCheckWithSources(
  "Solar energy is now cheaper than fossil fuels"
);

document.getElementById('fact-check-result').textContent = factCheck.result;
document.getElementById('sources').innerHTML = factCheck.citations
  .map(citation => `<a href="${citation.url}">${citation.title}</a>`)
  .join('<br>');
```

## Streaming Grounded Responses

```javascript
async function streamGroundedResearch(topic) {
  const ai = new GoogleGenAI({});

  const response = await ai.models.generateContentStream({
    model: "gemini-2.5-flash",
    contents: `Provide comprehensive, current information about: ${topic}`,
    tools: [{
      googleSearchRetrieval: {}
    }]
  });

  console.log("Streaming grounded research:");
  for await (const chunk of response) {
    process.stdout.write(chunk.text);
  }

  console.log("\n\nResearch complete.");
}

// Usage
await streamGroundedResearch("latest developments in renewable energy storage");
```

## Node.js Research Automation

```javascript
import * as fs from "node:fs";

class ResearchAutomation {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async researchAndSave(topics, outputDir) {
    for (const topic of topics) {
      console.log(`Researching: ${topic}`);

      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-pro",
        contents: `Provide comprehensive research on: ${topic}`,
        tools: [{
          googleSearchRetrieval: {}
        }]
      });

      // Save research to file
      const filename = `${outputDir}/${topic.replace(/[^a-z0-9]/gi, '_').toLowerCase()}.md`;
      const content = `# Research: ${topic}\n\n${response.text}\n\n---\n*Generated: ${new Date().toISOString()}*`;
      
      fs.writeFileSync(filename, content);
      console.log(`Saved research to: ${filename}`);
    }
  }

  async generateReports(queries, reportType = "summary") {
    const reports = {};

    for (const query of queries) {
      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents: `${reportType}: ${query}`,
        tools: [{
          googleSearchRetrieval: {}
        }]
      });

      reports[query] = response.text;
    }

    return reports;
  }
}

// Usage
const automation = new ResearchAutomation();

// Research multiple topics
const topics = [
  "AI regulation developments 2025",
  "Electric vehicle market trends",
  "Renewable energy policy changes"
];

await automation.researchAndSave(topics, "./research_output");

// Generate comparative reports
const queries = [
  "Compare renewable energy adoption across countries",
  "Analyze semiconductor supply chain resilience"
];

const reports = await automation.generateReports(queries, "Market analysis");
console.log("Generated reports:", Object.keys(reports));
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: "What are the latest tech industry layoffs?",
    tools: [{
      googleSearchRetrieval: {}
    }]
  });

  console.log(response.text);

} catch (error) {
  if (error.message.includes("grounding not available")) {
    console.log("Google Search grounding not available in this region");
    
    // Fallback to regular generation
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: "What are generally known patterns in tech industry layoffs?"
    });
    
    console.log("Fallback response:", response.text);
  } else if (error.message.includes("quota exceeded")) {
    console.log("Search quota exceeded - try again later");
  } else {
    throw error;
  }
}
```

## Best Practices

1. **Current Information**: Use grounding for time-sensitive queries
2. **Fact Verification**: Always ground factual claims
3. **Source Quality**: Grounding provides more reliable sources
4. **Regional Availability**: Check if grounding is available in your region
5. **Query Specificity**: Be specific to get better search results
6. **Citation Handling**: Parse and present source citations appropriately

## See Also
- [Text Generation](text-generation.md) - Regular generation without grounding
- [Function Calling](function-calling.md) - Custom tool integration
- [Structured Output](structured-output.md) - JSON responses with grounding