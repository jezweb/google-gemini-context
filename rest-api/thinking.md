# Thinking in REST API

*Enable reasoning mode with Gemini REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/thinking-mode
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-pro (thinking mode)
Key Features: Step-by-step reasoning, problem decomposition
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/gemini-2.5-pro:generateContent`
- **Config**: `thinkingConfig` parameter
- **Model**: `gemini-2.5-pro` only
- **Use cases**: Complex reasoning, math, analysis

## Enable Thinking Mode

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Solve this complex math problem: If a train travels 120 km in 1.5 hours, and another train travels 200 km in 2 hours, which is faster and by how much?"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }'
```

## Response Format

```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "text": "The first train is faster by 13.33 km/h."
      }],
      "role": "model"
    },
    "thinking": "Let me calculate the speed of each train...\n\nTrain 1:\nDistance = 120 km\nTime = 1.5 hours\nSpeed = 120 ÷ 1.5 = 80 km/h\n\nTrain 2:\nDistance = 200 km\nTime = 2 hours\nSpeed = 200 ÷ 2 = 100 km/h\n\nWait, that's not right. Let me recalculate...",
    "finishReason": "STOP"
  }]
}
```

## Complex Problem Solving

```json
{
  "contents": [{
    "parts": [{
      "text": "A company has 3 departments. Sales has 12 people earning average $50k. Marketing has 8 people earning average $60k. Engineering has 15 people earning average $80k. If the company wants to give a 10% raise to all employees but needs to stay within a budget increase of $180k, is this possible?"
    }]
  }],
  "generationConfig": {
    "thinkingConfig": {
      "includeThinkingProcess": true
    }
  }
}
```

## Mathematical Reasoning

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Find the area between the curves y = x² and y = 2x - x² from x = 0 to x = 1"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }'
```

## Complete Thinking Script

```bash
#!/bin/bash
# thinking_solver.sh

PROBLEM="$1"

RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PROBLEM"'"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }')

# Extract thinking process
echo "=== THINKING PROCESS ==="
echo "$RESPONSE" | jq -r '.candidates[0].thinking'

echo ""
echo "=== FINAL ANSWER ==="
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
```

## Python Implementation

```python
import requests
import json

class ThinkingAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def solve_with_thinking(self, problem):
        url = f"{self.base_url}/models/gemini-2.5-pro:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": problem
                }]
            }],
            "generationConfig": {
                "thinkingConfig": {
                    "includeThinkingProcess": True
                }
            }
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        if "error" in result:
            raise Exception(f"API Error: {result['error']['message']}")
        
        candidate = result["candidates"][0]
        
        return {
            "thinking": candidate.get("thinking", ""),
            "answer": candidate["content"]["parts"][0]["text"]
        }
    
    def analyze_business_case(self, case_study):
        prompt = f"""Analyze this business case:
        {case_study}
        
        Provide:
        1. Situation analysis
        2. Key challenges
        3. Available options
        4. Recommended strategy
        5. Implementation plan
        6. Risk assessment"""
        
        return self.solve_with_thinking(prompt)
    
    def solve_logic_puzzle(self, puzzle):
        prompt = f"""Solve this logic puzzle step by step:
        {puzzle}
        
        Show your deductive reasoning process clearly."""
        
        return self.solve_with_thinking(prompt)

# Usage
api = ThinkingAPI(os.environ["GEMINI_API_KEY"])

result = api.solve_with_thinking(
    "Design an optimal algorithm for sorting a million integers with limited memory"
)

print("Thinking process:")
print(result["thinking"])
print("\nFinal answer:")
print(result["answer"])
```

## Strategic Analysis

```json
{
  "contents": [{
    "parts": [{
      "text": "Compare these alternatives:\nOptions: Build in-house vs Buy existing solution vs Partner with vendor\nCriteria: Cost, time to market, control, scalability, risk\n\nProvide detailed comparison with pros/cons and recommendation."
    }]
  }],
  "generationConfig": {
    "thinkingConfig": {
      "includeThinkingProcess": true
    }
  }
}
```

## Logic Puzzle Solving

```bash
PUZZLE="Three friends - Alice, Bob, and Carol - each have a different pet (cat, dog, bird) and live in different colored houses (red, blue, green).

Clues:
1. Alice doesn't live in the red house
2. The person with the cat lives in the blue house
3. Bob doesn't have the bird
4. Carol doesn't live in the green house
5. The person in the red house has the dog

Who has which pet and lives in which house?"

curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PUZZLE"'"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }'
```

## Streaming Thinking Mode

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Analyze the ethical implications of AI in healthcare decision-making"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }' | while IFS= read -r line; do
    if [[ $line == data:* ]]; then
      json_data="${line#data: }"
      if [ "$json_data" != "[DONE]" ]; then
        # Extract thinking if present
        thinking=$(echo "$json_data" | jq -r '.candidates[0].thinking // empty' 2>/dev/null)
        if [ -n "$thinking" ]; then
          echo -n "$thinking"
        fi
        
        # Extract regular text
        text=$(echo "$json_data" | jq -r '.candidates[0].content.parts[0].text // empty' 2>/dev/null)
        if [ -n "$text" ]; then
          echo -n "$text"
        fi
      fi
    fi
  done
```

## JavaScript Implementation

```javascript
class ThinkingAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async solveWithThinking(problem) {
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
                            text: problem
                        }]
                    }],
                    generationConfig: {
                        thinkingConfig: {
                            includeThinkingProcess: true
                        }
                    }
                })
            }
        );
        
        const result = await response.json();
        
        if (result.error) {
            throw new Error(`API Error: ${result.error.message}`);
        }
        
        const candidate = result.candidates[0];
        
        return {
            thinking: candidate.thinking || "",
            answer: candidate.content.parts[0].text
        };
    }
    
    async streamThinking(problem) {
        const response = await fetch(
            `${this.baseUrl}/models/gemini-2.5-pro:streamGenerateContent`,
            {
                method: 'POST',
                headers: {
                    'x-goog-api-key': this.apiKey,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [{
                            text: problem
                        }]
                    }],
                    generationConfig: {
                        thinkingConfig: {
                            includeThinkingProcess: true
                        }
                    }
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
                        const candidate = parsed.candidates[0];
                        
                        if (candidate.thinking) {
                            console.log(candidate.thinking);
                        } else if (candidate.content?.parts[0]?.text) {
                            console.log(candidate.content.parts[0].text);
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
const api = new ThinkingAPI(process.env.GEMINI_API_KEY);

const result = await api.solveWithThinking(
    "Analyze the potential consequences of implementing universal basic income"
);

console.log("Thinking process:");
console.log(result.thinking);
console.log("\nFinal analysis:");
console.log(result.answer);
```

## Multi-Step Analysis

```bash
#!/bin/bash
# multi_step_thinking.sh

STEPS=(
  "Step 1: Identify the key components of the problem"
  "Step 2: Analyze each component separately"
  "Step 3: Look for relationships between components"
  "Step 4: Synthesize findings into recommendations"
)

PROBLEM="$1"

for step in "${STEPS[@]}"; do
  echo "=== $step ==="
  
  RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "For the problem: '"$PROBLEM"'\n\n'"$step"'"
        }]
      }],
      "generationConfig": {
        "thinkingConfig": {
          "includeThinkingProcess": true
        }
      }
    }')
  
  echo "Thinking:"
  echo "$RESPONSE" | jq -r '.candidates[0].thinking'
  echo ""
  echo "Analysis:"
  echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
  echo ""
  echo "---"
done
```

## PowerShell Implementation

```powershell
function Invoke-ThinkingMode {
    param(
        [string]$Problem,
        [string]$ApiKey,
        [switch]$ShowThinking
    )
    
    $Headers = @{
        "x-goog-api-key" = $ApiKey
        "Content-Type" = "application/json"
    }
    
    $Body = @{
        contents = @(
            @{
                parts = @(
                    @{
                        text = $Problem
                    }
                )
            }
        )
        generationConfig = @{
            thinkingConfig = @{
                includeThinkingProcess = $true
            }
        }
    } | ConvertTo-Json -Depth 4
    
    $Response = Invoke-RestMethod -Uri "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" -Method POST -Headers $Headers -Body $Body
    
    if ($ShowThinking) {
        Write-Host "=== THINKING PROCESS ===" -ForegroundColor Yellow
        Write-Host $Response.candidates[0].thinking
        Write-Host ""
        Write-Host "=== FINAL ANSWER ===" -ForegroundColor Green
    }
    
    return $Response.candidates[0].content.parts[0].text
}

# Usage
$Result = Invoke-ThinkingMode -Problem "Should we invest in renewable energy infrastructure?" -ApiKey $env:GEMINI_API_KEY -ShowThinking
Write-Output $Result
```

## Error Handling

```bash
# Check for thinking mode support
RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PROBLEM"'"
      }]
    }],
    "generationConfig": {
      "thinkingConfig": {
        "includeThinkingProcess": true
      }
    }
  }')

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"thinking mode not supported"* ]]; then
        echo "Thinking mode only available on gemini-2.5-pro"
        # Fallback to regular generation
        curl -s -X POST \
          "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
          -H "x-goog-api-key: $GEMINI_API_KEY" \
          -H "Content-Type: application/json" \
          -d '{
            "contents": [{
              "parts": [{
                "text": "'"$PROBLEM"'"
              }]
            }]
          }' | jq -r '.candidates[0].content.parts[0].text'
    else
        echo "Error: $ERROR_MSG"
        exit 1
    fi
else
    # Success - extract both thinking and answer
    echo "=== THINKING ==="
    echo "$RESPONSE" | jq -r '.candidates[0].thinking'
    echo ""
    echo "=== ANSWER ==="
    echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
fi
```

## Best Practices

1. **Model Selection**: Only use `gemini-2.5-pro` for thinking mode
2. **Complex Problems**: Best for multi-step reasoning tasks
3. **Transparency**: Use when you need to see reasoning process
4. **Problem Structure**: Break complex problems into clear parts
5. **Review Thinking**: Analyze reasoning for logical gaps
6. **Streaming**: Use streaming for long reasoning processes

## See Also
- [Text Generation](text-generation.md) - Regular generation
- [Streaming](streaming.md) - Stream thinking process
- [Error Handling](error-handling.md) - Handle failures