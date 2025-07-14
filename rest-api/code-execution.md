# Code Execution in REST API

*Enable Gemini to write and execute code via REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/code-execution
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-pro, gemini-2.5-flash
Key Features: Python execution, data analysis, problem solving
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/{model}:generateContent`
- **Tool**: `{"code_execution": {}}`
- **Languages**: Python only
- **Use cases**: Math, data analysis, algorithms

## Basic Code Execution

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Calculate the factorial of 20 and show your work"
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }'
```

## Data Analysis Example

```json
{
  "contents": [{
    "parts": [{
      "text": "Analyze this sales data and create a summary:\nMonth, Sales\nJan, 45000\nFeb, 52000\nMar, 48000\nApr, 61000\nMay, 58000\nJun, 67000\n\nCalculate the average, find the trend, and predict July sales."
    }]
  }],
  "tools": [{
    "code_execution": {}
  }]
}
```

## Mathematical Problem Solving

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Solve this system of equations:\n2x + 3y = 13\nx - y = -1\n\nShow all steps and verify the solution."
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }'
```

## Complete Analysis Script

```bash
#!/bin/bash
# code_execution.sh

PROBLEM="$1"

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PROBLEM"'"
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }' | jq -r '.candidates[0].content.parts[0].text'
```

## Python Implementation

```python
import requests
import json
import re

class CodeExecutionAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def execute_with_code(self, prompt, model="gemini-2.5-flash"):
        """Execute prompt with code execution enabled"""
        url = f"{self.base_url}/models/{model}:generateContent"
        
        data = {
            "contents": [{
                "parts": [{
                    "text": prompt
                }]
            }],
            "tools": [{
                "code_execution": {}
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        if "error" in result:
            raise Exception(f"API Error: {result['error']['message']}")
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def analyze_data(self, data, analysis_type="statistical"):
        """Analyze data with different focus"""
        prompts = {
            "statistical": f"Perform statistical analysis on this dataset: {data}\nCalculate mean, median, mode, standard deviation, and create a distribution summary.",
            "trend": f"Analyze trends in this time series data: {data}\nIdentify patterns, calculate growth rates, and make predictions.",
            "correlation": f"Find correlations in this dataset: {data}\nCalculate correlation coefficients and identify significant relationships."
        }
        
        prompt = prompts.get(analysis_type, prompts["statistical"])
        return self.execute_with_code(prompt)
    
    def solve_math_problem(self, problem):
        """Solve mathematical problems with code"""
        prompt = f"Solve this problem step by step with code:\n{problem}\n\nShow all calculations, verify the answer, and explain the approach."
        return self.execute_with_code(prompt)
    
    def extract_code_blocks(self, response_text):
        """Extract Python code blocks from response"""
        pattern = r"```python\n(.*?)```"
        matches = re.findall(pattern, response_text, re.DOTALL)
        
        code_blocks = []
        for match in matches:
            code_blocks.append({
                "language": "python",
                "code": match.strip()
            })
        
        return code_blocks

# Usage
api = CodeExecutionAPI(os.environ["GEMINI_API_KEY"])

# Analyze data
result = api.analyze_data(
    data="[10, 15, 12, 18, 22, 25, 28, 30, 35, 40]",
    analysis_type="statistical"
)
print(result)

# Solve math problem
result = api.solve_math_problem(
    "Find the roots of the quadratic equation 2x² - 5x + 3 = 0"
)
print(result)
```

## Scientific Computing

```json
{
  "contents": [{
    "parts": [{
      "text": "Calculate the trajectory of a projectile:\nInitial velocity: 50 m/s\nLaunch angle: 45 degrees\n\nFind: maximum height, range, time of flight\nPlot the trajectory path."
    }]
  }],
  "tools": [{
    "code_execution": {}
  }]
}
```

## Algorithm Implementation

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Implement and test a binary search algorithm. Create a sorted list of 20 random numbers and search for specific values. Show the algorithm'\''s efficiency."
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }'
```

## Data Visualization Request

```json
{
  "contents": [{
    "parts": [{
      "text": "Create a visualization for this data:\nCategories: A, B, C, D, E\nValues: 23, 45, 12, 67, 34\n\nGenerate code to create a bar chart and pie chart.\nCalculate percentages for each category."
    }]
  }],
  "tools": [{
    "code_execution": {}
  }]
}
```

## Streaming Code Execution

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:streamGenerateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Create a Monte Carlo simulation to estimate π"
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }'
```

## Complex Problem Solving

```bash
#!/bin/bash
# optimization_problem.sh

OBJECTIVE="$1"
CONSTRAINTS="$2"

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Solve this optimization problem:\nObjective: '"$OBJECTIVE"'\nConstraints: '"$CONSTRAINTS"'\n\nUse appropriate algorithms and show the optimal solution."
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }' | jq -r '.candidates[0].content.parts[0].text'
```

## JavaScript Implementation

```javascript
class CodeExecutionAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async executeWithCode(prompt, model = "gemini-2.5-flash") {
        const url = `${this.baseUrl}/models/${model}:generateContent`;
        
        const response = await fetch(url, {
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
                    code_execution: {}
                }]
            })
        });
        
        const result = await response.json();
        
        if (result.error) {
            throw new Error(`API Error: ${result.error.message}`);
        }
        
        return result.candidates[0].content.parts[0].text;
    }
    
    async analyzeData(data, analysisType = "statistical") {
        const prompts = {
            statistical: `Perform statistical analysis on this dataset: ${data}\nCalculate mean, median, mode, standard deviation, and create a distribution summary.`,
            trend: `Analyze trends in this time series data: ${data}\nIdentify patterns, calculate growth rates, and make predictions.`,
            correlation: `Find correlations in this dataset: ${data}\nCalculate correlation coefficients and identify significant relationships.`
        };
        
        const prompt = prompts[analysisType] || prompts.statistical;
        return await this.executeWithCode(prompt);
    }
    
    extractCodeBlocks(responseText) {
        const pattern = /```python\n(.*?)```/gs;
        const matches = [...responseText.matchAll(pattern)];
        
        return matches.map(match => ({
            language: "python",
            code: match[1].trim()
        }));
    }
}

// Usage
const api = new CodeExecutionAPI(process.env.GEMINI_API_KEY);

// Analyze data
const result = await api.analyzeData(
    "[10, 15, 12, 18, 22, 25, 28, 30, 35, 40]",
    "statistical"
);
console.log(result);
```

## Multi-Step Problem Solving

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Generate comprehensive test cases for: a function that sorts a list of integers\n\nInclude edge cases, normal cases, and error cases.\nImplement the function and run all tests."
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }'
```

## Response Format

```json
{
  "candidates": [{
    "content": {
      "parts": [{
        "text": "I'll help you calculate the factorial of 20 step by step.\n\n```python\nimport math\n\ndef factorial(n):\n    if n == 0 or n == 1:\n        return 1\n    result = 1\n    for i in range(2, n + 1):\n        result *= i\n    return result\n\n# Calculate factorial of 20\nresult = factorial(20)\nprint(f\"20! = {result}\")\n\n# Verify with built-in function\nverify = math.factorial(20)\nprint(f\"Verification: {verify}\")\nprint(f\"Match: {result == verify}\")\n```\n\nThe factorial of 20 is 2,432,902,008,176,640,000."
      }]
    }
  }]
}
```

## Error Handling

```bash
# Check for code execution errors
RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "'"$PROBLEM"'"
      }]
    }],
    "tools": [{
      "code_execution": {}
    }]
  }')

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"not supported"* ]]; then
        echo "Code execution not supported for this model"
    elif [[ "$ERROR_MSG" == *"timeout"* ]]; then
        echo "Code execution timed out"
    else
        echo "Error: $ERROR_MSG"
    fi
    exit 1
fi

# Extract result
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
```

## PowerShell Implementation

```powershell
function Invoke-CodeExecution {
    param(
        [string]$Prompt,
        [string]$ApiKey,
        [string]$Model = "gemini-2.5-flash"
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
                        text = $Prompt
                    }
                )
            }
        )
        tools = @(
            @{
                code_execution = @{}
            }
        )
    } | ConvertTo-Json -Depth 4
    
    $Response = Invoke-RestMethod -Uri "https://generativelanguage.googleapis.com/v1beta/models/$Model`:generateContent" -Method POST -Headers $Headers -Body $Body
    return $Response.candidates[0].content.parts[0].text
}

# Usage
$Result = Invoke-CodeExecution -Prompt "Calculate the area of a circle with radius 5" -ApiKey $env:GEMINI_API_KEY
Write-Output $Result
```

## Best Practices

1. **Clear Instructions**: Specify exactly what to calculate
2. **Data Format**: Provide data in easily parseable format
3. **Verification**: Ask for answer verification
4. **Step-by-Step**: Request detailed explanations
5. **Error Handling**: Check for execution failures
6. **Inline Data**: Provide data directly in requests

## See Also
- [Tools](tools.md) - Other available tools
- [Streaming](streaming.md) - Stream code execution
- [Error Handling](error-handling.md) - Handle failures