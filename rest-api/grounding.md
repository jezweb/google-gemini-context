# Grounding in REST API

*Ground responses with Google Search via REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/grounding
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Google Search integration, fact verification, citation support
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/{model}:generateContent`
- **Tool**: `google_search_retrieval`
- **Use cases**: Current events, fact checking, research
- **Features**: Real-time search, automatic citations

## Basic Google Search Grounding

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "What are the latest developments in quantum computing in 2025?"
      }]
    }],
    "tools": [{
      "google_search_retrieval": {}
    }]
  }'
```

## Fact-Checking Script

```bash
#!/bin/bash
# fact_checker.sh

CLAIM="$1"
CONTEXT="${2:-}"

# Build the prompt
if [ -n "$CONTEXT" ]; then
    PROMPT="Context: $CONTEXT\n\nFact-check this claim using reliable, current sources:\n\nClaim: $CLAIM\n\nProvide:\n1. Verification status (Verified/False/Partially True/Insufficient Evidence)\n2. Supporting evidence with sources\n3. Any contradictory information\n4. Confidence level in the assessment"
else
    PROMPT="Fact-check this claim using reliable, current sources:\n\nClaim: $CLAIM\n\nProvide:\n1. Verification status (Verified/False/Partially True/Insufficient Evidence)\n2. Supporting evidence with sources\n3. Any contradictory information\n4. Confidence level in the assessment"
fi

curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [{
                "text": "'"$PROMPT"'"
            }]
        }],
        "tools": [{
            "google_search_retrieval": {}
        }]
    }' | jq -r '.candidates[0].content.parts[0].text'
```

## Current Events Research

```bash
#!/bin/bash
# news_researcher.sh

TOPIC="$1"
REGION="${2:-global}"

if [ "$REGION" != "global" ]; then
    REGION_CONTEXT=" in $REGION"
else
    REGION_CONTEXT=""
fi

PROMPT="What are the latest news and developments about $TOPIC$REGION_CONTEXT?

Provide:
1. Recent headlines and key events
2. Timeline of important developments
3. Analysis of trends and implications
4. Key stakeholders and their positions"

curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [{
                "text": "'"$PROMPT"'"
            }]
        }],
        "tools": [{
            "google_search_retrieval": {}
        }]
    }' | jq -r '.candidates[0].content.parts[0].text'
```

## Market Analysis Tool

```bash
#!/bin/bash
# market_analyzer.sh

MARKET_SECTOR="$1"

PROMPT="Provide a current market update for $MARKET_SECTOR:

Include:
1. Recent price movements and trends
2. Key market drivers
3. Important news affecting the sector
4. Analyst opinions and forecasts
5. Risk factors to watch"

curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [{
                "text": "'"$PROMPT"'"
            }]
        }],
        "tools": [{
            "google_search_retrieval": {}
        }]
    }' | jq -r '.candidates[0].content.parts[0].text'
```

## Python Implementation

```python
import requests
import json

class GroundedResearchAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def research_topic(self, topic, depth="comprehensive"):
        depth_prompts = {
            "basic": f"Provide a basic overview of: {topic}",
            "comprehensive": f"Provide a comprehensive analysis of: {topic}. Include recent developments, key players, and current trends.",
            "technical": f"Provide a technical deep-dive into: {topic}. Include latest research, methodologies, and expert opinions.",
            "news": f"What are the latest news and developments about: {topic}?"
        }
        
        prompt = depth_prompts.get(depth, depth_prompts["comprehensive"])
        
        url = f"{self.base_url}/models/gemini-2.5-pro:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": prompt
                }]
            }],
            "tools": [{
                "google_search_retrieval": {}
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        if "error" in result:
            raise Exception(f"Research failed: {result['error']['message']}")
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def fact_check(self, claim, context=""):
        context_text = f"Context: {context}\n\n" if context else ""
        
        prompt = f"""{context_text}Fact-check this claim using reliable, current sources:
        
        Claim: {claim}
        
        Assessment should include:
        1. Verification status (Verified/False/Partially True/Insufficient Evidence)
        2. Sources supporting or refuting the claim
        3. Any nuances or important context
        4. Confidence level in the assessment
        5. Date sensitivity (when was this true/false)"""
        
        url = f"{self.base_url}/models/gemini-2.5-pro:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": prompt
                }]
            }],
            "tools": [{
                "google_search_retrieval": {}
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def market_analysis(self, company_or_industry):
        prompt = f"""Provide a comprehensive market analysis for {company_or_industry}:
        
        Include:
        1. Current market size and growth trends
        2. Key competitors and market share
        3. Recent industry developments
        4. Regulatory environment and changes
        5. Technological disruptions
        6. Investment and funding activities
        7. Future outlook and opportunities"""
        
        url = f"{self.base_url}/models/gemini-2.5-pro:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": prompt
                }]
            }],
            "tools": [{
                "google_search_retrieval": {}
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def competitor_intelligence(self, company, competitors):
        competitor_list = ", ".join(competitors)
        
        prompt = f"""Analyze {company} against its competitors ({competitor_list}):
        
        For each competitor, provide:
        1. Recent strategic moves and announcements
        2. Product launches and updates
        3. Financial performance highlights
        4. Market positioning changes
        5. Partnership and acquisition activities
        6. Strengths and weaknesses relative to {company}"""
        
        url = f"{self.base_url}/models/gemini-2.5-flash:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": prompt
                }]
            }],
            "tools": [{
                "google_search_retrieval": {}
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        return result["candidates"][0]["content"]["parts"][0]["text"]

# Usage
research_api = GroundedResearchAPI(os.environ["GEMINI_API_KEY"])

# Research topic
research = research_api.research_topic("AI safety regulations 2025", "comprehensive")
print("Research findings:")
print(research)

# Fact-check claim
fact_check = research_api.fact_check("OpenAI released GPT-5 in January 2025")
print("\nFact check result:")
print(fact_check)

# Market analysis
market = research_api.market_analysis("electric vehicle charging infrastructure")
print("\nMarket analysis:")
print(market)
```

## Academic Research Tool

```bash
#!/bin/bash
# academic_researcher.sh

RESEARCH_QUESTION="$1"
FIELD="${2:-general}"

case $FIELD in
    "literature")
        PROMPT="Conduct a literature review for this research question: $RESEARCH_QUESTION

Provide:
1. Overview of current research landscape
2. Key theories and methodologies
3. Recent significant studies and findings
4. Gaps in current research
5. Emerging trends and future directions
6. Recommendations for further research"
        ;;
    "experts")
        PROMPT="Identify current leading experts and researchers in $RESEARCH_QUESTION:

For each expert, provide:
1. Name and current affiliation
2. Key areas of expertise
3. Recent notable publications or work
4. Current projects or research focus
5. How to find their work (websites, profiles)"
        ;;
    "methodology")
        PROMPT="What are the current best practices and methodologies for researching $RESEARCH_QUESTION?

Include:
1. Recommended research approaches
2. Data collection methods
3. Analysis techniques
4. Tools and software commonly used
5. Ethical considerations
6. Recent methodological innovations"
        ;;
    *)
        PROMPT="Provide comprehensive academic research on: $RESEARCH_QUESTION"
        ;;
esac

curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [{
                "text": "'"$PROMPT"'"
            }]
        }],
        "tools": [{
            "google_search_retrieval": {}
        }]
    }' | jq -r '.candidates[0].content.parts[0].text'
```

## Trend Analysis

```json
{
  "contents": [{
    "parts": [{
      "text": "Analyze the current trend: electric vehicle adoption\n\nInclude:\n1. Origin and timeline of the trend\n2. Current status and adoption\n3. Key drivers and influencers\n4. Potential future implications\n5. Regional variations if applicable"
    }]
  }],
  "tools": [{
    "google_search_retrieval": {}
  }]
}
```

## Streaming Grounded Research

```bash
#!/bin/bash
# stream_research.sh

TOPIC="$1"

curl -N -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [{
                "text": "Provide comprehensive, current information about: '"$TOPIC"'"
            }]
        }],
        "tools": [{
            "google_search_retrieval": {}
        }]
    }' | while IFS= read -r line; do
        if [[ $line == data:* ]]; then
            json_data="${line#data: }"
            if [ "$json_data" != "[DONE]" ]; then
                echo "$json_data" | jq -r '.candidates[0].content.parts[0].text // empty' 2>/dev/null | tr -d '\n'
            fi
        fi
    done

echo ""
```

## Batch Grounded Research

```bash
#!/bin/bash
# batch_grounded_research.sh

TOPICS_FILE="$1"
OUTPUT_DIR="${2:-./research_output}"

# Create output directory
mkdir -p "$OUTPUT_DIR"

# Create batch requests
create_batch_requests() {
    local requests="["
    local first=true
    
    while IFS= read -r topic; do
        if [ -n "$topic" ]; then
            if [ "$first" = true ]; then
                first=false
            else
                requests+=","
            fi
            
            requests+='{
                "model": "models/gemini-2.5-flash",
                "contents": [{
                    "parts": [{
                        "text": "Provide comprehensive, current research on: '"$topic"'"
                    }]
                }],
                "tools": [{
                    "google_search_retrieval": {}
                }],
                "metadata": {
                    "topic": "'"$topic"'"
                }
            }'
        fi
    done < "$TOPICS_FILE"
    
    requests+="]"
    echo "$requests"
}

# Submit batch
echo "Creating batch for grounded research..."
BATCH_REQUESTS=$(create_batch_requests)

BATCH_RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/batches" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "requests": '"$BATCH_REQUESTS"'
    }')

BATCH_ID=$(echo "$BATCH_RESPONSE" | jq -r '.name' | sed 's/batches\///')

if [ "$BATCH_ID" = "null" ]; then
    echo "Failed to create batch:"
    echo "$BATCH_RESPONSE" | jq '.error'
    exit 1
fi

echo "Batch created: $BATCH_ID"

# Monitor batch
while true; do
    STATUS=$(curl -s -X GET \
        "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID" \
        -H "x-goog-api-key: $GEMINI_API_KEY")
    
    STATE=$(echo "$STATUS" | jq -r '.state')
    COMPLETED=$(echo "$STATUS" | jq -r '.completedRequests // 0')
    TOTAL=$(echo "$STATUS" | jq -r '.totalRequests // 0')
    
    echo "Batch status: $STATE ($COMPLETED/$TOTAL)"
    
    if [ "$STATE" = "COMPLETED" ]; then
        break
    elif [ "$STATE" = "FAILED" ]; then
        echo "Batch failed!"
        exit 1
    fi
    
    sleep 30
done

# Get results and save to files
echo "Saving research results..."
curl -s -X GET \
    "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID/results" \
    -H "x-goog-api-key: $GEMINI_API_KEY" | \
jq -r '.results[] | [.metadata.topic, .response.candidates[0].content.parts[0].text] | @tsv' | \
while IFS=$'\t' read -r topic content; do
    filename=$(echo "$topic" | tr ' ' '_' | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9_]//g')
    echo "# Research: $topic" > "$OUTPUT_DIR/${filename}.md"
    echo "" >> "$OUTPUT_DIR/${filename}.md"
    echo "$content" >> "$OUTPUT_DIR/${filename}.md"
    echo "" >> "$OUTPUT_DIR/${filename}.md"
    echo "---" >> "$OUTPUT_DIR/${filename}.md"
    echo "*Generated: $(date)*" >> "$OUTPUT_DIR/${filename}.md"
    echo "Saved: $OUTPUT_DIR/${filename}.md"
done
```

## JavaScript Implementation

```javascript
class GroundedResearchAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async researchTopic(topic, depth = "comprehensive") {
        const depthPrompts = {
            basic: `Provide a basic overview of: ${topic}`,
            comprehensive: `Provide a comprehensive analysis of: ${topic}. Include recent developments, key players, and current trends.`,
            technical: `Provide a technical deep-dive into: ${topic}. Include latest research, methodologies, and expert opinions.`,
            news: `What are the latest news and developments about: ${topic}?`
        };
        
        const prompt = depthPrompts[depth] || depthPrompts.comprehensive;
        
        const response = await fetch(
            `${this.baseUrl}/models/gemini-2.5-pro:generateContent`,
            {
                method: 'POST',
                headers: {
                    'x-goog-api-key': this.apiKey,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }],
                    tools: [{
                        google_search_retrieval: {}
                    }]
                })
            }
        );
        
        const result = await response.json();
        
        if (result.error) {
            throw new Error(`Research failed: ${result.error.message}`);
        }
        
        return result.candidates[0].content.parts[0].text;
    }
    
    async factCheck(claim, context = "") {
        const contextText = context ? `Context: ${context}\n\n` : "";
        
        const prompt = `${contextText}Fact-check this claim using reliable, current sources:
        
        Claim: ${claim}
        
        Assessment should include:
        1. Verification status (Verified/False/Partially True/Insufficient Evidence)
        2. Sources supporting or refuting the claim
        3. Any nuances or important context
        4. Confidence level in the assessment
        5. Date sensitivity (when was this true/false)`;
        
        const response = await fetch(
            `${this.baseUrl}/models/gemini-2.5-pro:generateContent`,
            {
                method: 'POST',
                headers: {
                    'x-goog-api-key': this.apiKey,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }],
                    tools: [{
                        google_search_retrieval: {}
                    }]
                })
            }
        );
        
        const result = await response.json();
        return result.candidates[0].content.parts[0].text;
    }
    
    async streamResearch(topic) {
        const response = await fetch(
            `${this.baseUrl}/models/gemini-2.5-flash:streamGenerateContent`,
            {
                method: 'POST',
                headers: {
                    'x-goog-api-key': this.apiKey,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: `Provide comprehensive, current information about: ${topic}`
                        }]
                    }],
                    tools: [{
                        google_search_retrieval: {}
                    }]
                })
            }
        );
        
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            
            const chunk = decoder.decode(value);
            const lines = chunk.split('\n');
            
            for (const line of lines) {
                if (line.startsWith('data: ')) {
                    const data = line.slice(6);
                    if (data === '[DONE]') return;
                    
                    try {
                        const parsed = JSON.parse(data);
                        const text = parsed.candidates[0].content.parts[0].text;
                        if (text) {
                            console.log(text);
                        }
                    } catch (e) {
                        // Skip invalid JSON
                    }
                }
            }
        }
    }
}

// Usage
const researchAPI = new GroundedResearchAPI(process.env.GEMINI_API_KEY);

const research = await researchAPI.researchTopic("AI safety regulations 2025", "comprehensive");
console.log("Research findings:", research);
```

## PowerShell Implementation

```powershell
function Invoke-GroundedResearch {
    param(
        [string]$Topic,
        [string]$ApiKey,
        [string]$Depth = "comprehensive"
    )
    
    $DepthPrompts = @{
        "basic" = "Provide a basic overview of: $Topic"
        "comprehensive" = "Provide a comprehensive analysis of: $Topic. Include recent developments, key players, and current trends."
        "technical" = "Provide a technical deep-dive into: $Topic. Include latest research, methodologies, and expert opinions."
        "news" = "What are the latest news and developments about: $Topic?"
    }
    
    $Prompt = $DepthPrompts[$Depth]
    if (-not $Prompt) { $Prompt = $DepthPrompts["comprehensive"] }
    
    $Headers = @{
        "x-goog-api-key" = $ApiKey
        "Content-Type" = "application/json"
    }
    
    $Body = @{
        contents = @(
            @{
                parts = @(
                    @{
                        text = $Prompt
                    }
                )
            }
        )
        tools = @(
            @{
                google_search_retrieval = @{}
            }
        )
    } | ConvertTo-Json -Depth 4
    
    $Response = Invoke-RestMethod -Uri "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" -Method POST -Headers $Headers -Body $Body
    
    return $Response.candidates[0].content.parts[0].text
}

# Usage
$Research = Invoke-GroundedResearch -Topic "electric vehicle market trends" -ApiKey $env:GEMINI_API_KEY -Depth "comprehensive"
Write-Output $Research
```

## Error Handling

```bash
# Check for grounding errors
RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$REQUEST_DATA")

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"grounding not available"* ]]; then
        echo "Google Search grounding not available in this region"
        # Fallback to regular generation
        curl -s -X POST \
            "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
            -H "x-goog-api-key: $GEMINI_API_KEY" \
            -H "Content-Type: application/json" \
            -d '{
                "contents": [{
                    "parts": [{
                        "text": "What are generally known patterns in tech industry trends?"
                    }]
                }]
            }' | jq -r '.candidates[0].content.parts[0].text'
    elif [[ "$ERROR_MSG" == *"quota exceeded"* ]]; then
        echo "Search quota exceeded - try again later"
    else
        echo "Error: $ERROR_MSG"
    fi
    exit 1
fi

# Extract result
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
```

## Best Practices

1. **Current Information**: Use grounding for time-sensitive queries
2. **Fact Verification**: Always ground factual claims
3. **Source Quality**: Grounding provides more reliable sources
4. **Regional Availability**: Check if grounding is available in your region
5. **Query Specificity**: Be specific to get better search results
6. **Citation Handling**: Parse and present source citations appropriately

## See Also
- [Authentication](authentication.md) - API key setup
- [Tools](tools.md) - Other available tools
- [Streaming](streaming.md) - Stream grounded responses