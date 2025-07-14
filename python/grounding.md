# Grounding in Python

*Ground responses with Google Search and external data sources*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/grounding
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Google Search integration, fact verification, citation support
-->

## Quick Reference
- **Tool**: `google_search_retrieval`
- **Use cases**: Current events, fact checking, research
- **Features**: Real-time search, automatic citations
- **Models**: All Gemini 2.5 models support grounding

## Basic Google Search Grounding

```python
from google import genai

client = genai.Client()

# Enable Google Search grounding
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What are the latest developments in quantum computing in 2025?",
    tools=[{
        "google_search_retrieval": {}
    }]
)

print(response.text)
# Response includes information from recent search results with citations
```

## Grounded Research Assistant

```python
class GroundedResearcher:
    def __init__(self):
        self.client = genai.Client()
    
    def research_topic(self, topic, depth="comprehensive"):
        """Research a topic with Google Search grounding"""
        depth_prompts = {
            "basic": f"Provide a basic overview of: {topic}",
            "comprehensive": f"Provide a comprehensive analysis of: {topic}. Include recent developments, key players, and current trends.",
            "technical": f"Provide a technical deep-dive into: {topic}. Include latest research, methodologies, and expert opinions.",
            "news": f"What are the latest news and developments about: {topic}?"
        }
        
        prompt = depth_prompts.get(depth, depth_prompts["comprehensive"])
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return {
            "content": response.text,
            "sources": self._extract_sources(response)
        }
    
    def fact_check(self, claim):
        """Fact-check a claim using current information"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Fact-check this claim using current, reliable sources:
            
            Claim: {claim}
            
            Provide:
            1. Verification status (True/False/Partially True/Unclear)
            2. Supporting evidence with sources
            3. Any contradictory information
            4. Confidence level in the assessment""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def compare_sources(self, topic):
        """Compare information from multiple sources"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Research {topic} from multiple perspectives and sources.
            
            Compare and contrast different viewpoints, identify areas of consensus
            and disagreement, and provide a balanced analysis.""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def _extract_sources(self, response):
        """Extract citation sources from response"""
        # This would parse citations from the response
        # Implementation depends on response format
        sources = []
        if hasattr(response, 'citations'):
            for citation in response.citations:
                sources.append({
                    "title": citation.title,
                    "url": citation.url,
                    "snippet": citation.snippet
                })
        return sources

# Usage
researcher = GroundedResearcher()

# Research current topic
research = researcher.research_topic("AI safety regulations 2025", depth="comprehensive")
print("Research findings:")
print(research["content"])
print(f"\nSources: {len(research['sources'])} citations")

# Fact-check a claim
fact_check = researcher.fact_check("OpenAI released GPT-5 in January 2025")
print("\nFact check result:")
print(fact_check)

# Compare perspectives
comparison = researcher.compare_sources("climate change policies")
print("\nMultiple source comparison:")
print(comparison)
```

## Current Events Assistant

```python
class CurrentEventsAssistant:
    def __init__(self):
        self.client = genai.Client()
    
    def get_latest_news(self, topic, region="global"):
        """Get latest news on a topic"""
        region_context = f" in {region}" if region != "global" else ""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""What are the latest news and developments about {topic}{region_context}?
            
            Provide:
            1. Recent headlines and key events
            2. Timeline of important developments
            3. Analysis of trends and implications
            4. Key stakeholders and their positions""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def analyze_trend(self, trend):
        """Analyze a current trend with real-time data"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Analyze the current trend: {trend}
            
            Include:
            1. Origin and timeline of the trend
            2. Current status and adoption
            3. Key drivers and influencers
            4. Potential future implications
            5. Regional variations if applicable""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def market_update(self, market_sector):
        """Get current market information"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Provide a current market update for {market_sector}:
            
            Include:
            1. Recent price movements and trends
            2. Key market drivers
            3. Important news affecting the sector
            4. Analyst opinions and forecasts
            5. Risk factors to watch""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def event_context(self, event):
        """Provide context for a current event"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Provide comprehensive context for this event: {event}
            
            Include:
            1. Background and historical context
            2. Key players and stakeholders
            3. Timeline of events leading up to this
            4. Current status and recent developments
            5. Potential implications and next steps""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text

# Usage
news_assistant = CurrentEventsAssistant()

# Get latest news
news = news_assistant.get_latest_news("artificial intelligence regulation", "United States")
print("Latest AI regulation news:")
print(news)

# Analyze trend
trend_analysis = news_assistant.analyze_trend("electric vehicle adoption")
print("\nEV trend analysis:")
print(trend_analysis)

# Market update
market_info = news_assistant.market_update("cryptocurrency")
print("\nCrypto market update:")
print(market_info)
```

## Academic Research Assistant

```python
class AcademicResearcher:
    def __init__(self):
        self.client = genai.Client()
    
    def literature_review(self, research_question):
        """Conduct a literature review with current sources"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Conduct a literature review for this research question:
            {research_question}
            
            Provide:
            1. Overview of current research landscape
            2. Key theories and methodologies
            3. Recent significant studies and findings
            4. Gaps in current research
            5. Emerging trends and future directions
            6. Recommendations for further research""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def find_experts(self, field):
        """Find current experts and thought leaders in a field"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Identify current leading experts and researchers in {field}:
            
            For each expert, provide:
            1. Name and current affiliation
            2. Key areas of expertise
            3. Recent notable publications or work
            4. Current projects or research focus
            5. How to find their work (websites, profiles)""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def research_methodology(self, topic):
        """Get current best practices for research methodology"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""What are the current best practices and methodologies for researching {topic}?
            
            Include:
            1. Recommended research approaches
            2. Data collection methods
            3. Analysis techniques
            4. Tools and software commonly used
            5. Ethical considerations
            6. Recent methodological innovations""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def conference_tracker(self, field):
        """Track upcoming conferences and events"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Find upcoming conferences, workshops, and events in {field}:
            
            For each event, provide:
            1. Event name and dates
            2. Location (physical/virtual)
            3. Key themes and tracks
            4. Notable speakers or organizers
            5. Submission deadlines if applicable
            6. Registration information""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text

# Usage
academic_researcher = AcademicResearcher()

# Conduct literature review
literature = academic_researcher.literature_review("machine learning interpretability")
print("Literature review:")
print(literature)

# Find experts
experts = academic_researcher.find_experts("quantum computing")
print("\nQuantum computing experts:")
print(experts)

# Research methodology
methodology = academic_researcher.research_methodology("climate change impact assessment")
print("\nResearch methodology:")
print(methodology)
```

## Business Intelligence Assistant

```python
class BusinessIntelligenceAssistant:
    def __init__(self):
        self.client = genai.Client()
    
    def market_analysis(self, company_or_industry):
        """Analyze market conditions and competitive landscape"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Provide a comprehensive market analysis for {company_or_industry}:
            
            Include:
            1. Current market size and growth trends
            2. Key competitors and market share
            3. Recent industry developments
            4. Regulatory environment and changes
            5. Technological disruptions
            6. Investment and funding activities
            7. Future outlook and opportunities""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def competitor_intelligence(self, company, competitors):
        """Gather competitive intelligence"""
        competitor_list = ", ".join(competitors)
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Analyze {company} against its competitors ({competitor_list}):
            
            For each competitor, provide:
            1. Recent strategic moves and announcements
            2. Product launches and updates
            3. Financial performance highlights
            4. Market positioning changes
            5. Partnership and acquisition activities
            6. Strengths and weaknesses relative to {company}""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def industry_trends(self, industry):
        """Identify current and emerging industry trends"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""What are the current and emerging trends in the {industry} industry?
            
            Analyze:
            1. Technology trends and innovations
            2. Consumer behavior changes
            3. Regulatory and policy trends
            4. Investment and funding patterns
            5. Sustainability and ESG considerations
            6. Global vs regional variations
            7. Disruption threats and opportunities""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def startup_landscape(self, sector):
        """Map the startup landscape in a sector"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Map the current startup landscape in {sector}:
            
            Include:
            1. Notable startups and their focus areas
            2. Recent funding rounds and valuations
            3. Key investors and accelerators
            4. Emerging business models
            5. Geographic distribution of activity
            6. Success stories and unicorns
            7. Challenges and failure patterns""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text

# Usage
bi_assistant = BusinessIntelligenceAssistant()

# Market analysis
market = bi_assistant.market_analysis("electric vehicle charging infrastructure")
print("Market analysis:")
print(market)

# Competitor intelligence
competitors = bi_assistant.competitor_intelligence("Tesla", ["BYD", "Volkswagen", "General Motors"])
print("\nCompetitor analysis:")
print(competitors)

# Industry trends
trends = bi_assistant.industry_trends("fintech")
print("\nFintech trends:")
print(trends)
```

## Real-time Information Verification

```python
class InformationVerifier:
    def __init__(self):
        self.client = genai.Client()
    
    def verify_claim(self, claim, context=""):
        """Verify a specific claim against current sources"""
        context_text = f"Context: {context}\n\n" if context else ""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""{context_text}Verify this claim using reliable, current sources:
            
            Claim: {claim}
            
            Assessment should include:
            1. Verification status (Verified/False/Partially True/Insufficient Evidence)
            2. Sources supporting or refuting the claim
            3. Any nuances or important context
            4. Confidence level in the assessment
            5. Date sensitivity (when was this true/false)""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def cross_reference_sources(self, topic):
        """Cross-reference information across multiple sources"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Research {topic} across multiple reliable sources and provide:
            
            1. Points of consensus across sources
            2. Areas of disagreement or contradiction
            3. Quality and reliability of different sources
            4. Most recent and authoritative information
            5. Any bias or perspective differences
            6. Recommended primary sources for further research""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text
    
    def temporal_verification(self, claim, time_frame):
        """Verify how a claim's truth has changed over time"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Analyze how this claim has evolved over {time_frame}:
            
            Claim: {claim}
            
            Provide:
            1. Historical accuracy of the claim
            2. Key events that changed its validity
            3. Current status of the claim
            4. Timeline of relevant developments
            5. Factors that might affect future validity""",
            tools=[{
                "google_search_retrieval": {}
            }]
        )
        
        return response.text

# Usage
verifier = InformationVerifier()

# Verify a claim
verification = verifier.verify_claim(
    "China is the world's largest producer of electric vehicles",
    "As of 2025"
)
print("Claim verification:")
print(verification)

# Cross-reference sources
cross_ref = verifier.cross_reference_sources("effectiveness of COVID-19 vaccines")
print("\nCross-reference analysis:")
print(cross_ref)

# Temporal verification
temporal = verifier.temporal_verification(
    "Remote work is the dominant work model",
    "the past 5 years"
)
print("\nTemporal verification:")
print(temporal)
```

## Streaming Grounded Responses

```python
def stream_grounded_research(topic):
    """Stream grounded research responses"""
    client = genai.Client()
    
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=f"Provide comprehensive, current information about: {topic}",
        tools=[{
            "google_search_retrieval": {}
        }],
        config={"stream": True}
    )
    
    print("Streaming grounded research:")
    for chunk in response:
        print(chunk.text, end="", flush=True)
    
    print("\n\nResearch complete.")

# Usage
stream_grounded_research("latest developments in renewable energy storage")
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents="What are the latest tech industry layoffs?",
        tools=[{
            "google_search_retrieval": {}
        }]
    )
    
    print(response.text)
    
except Exception as error:
    if "grounding not available" in str(error):
        print("Google Search grounding not available in this region")
        # Fallback to regular generation
        response = client.models.generate_content(
            model="gemini-2.5-flash",
            contents="What are generally known patterns in tech industry layoffs?"
        )
        print("Fallback response:", response.text)
    elif "quota exceeded" in str(error):
        print("Search quota exceeded - try again later")
    else:
        raise
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