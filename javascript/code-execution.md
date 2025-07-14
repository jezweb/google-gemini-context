# Code Execution in JavaScript

*Enable Gemini to write and execute code*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/code-execution
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-pro, gemini-2.5-flash
Key Features: Python execution, data analysis, problem solving
-->

## Quick Reference
- **Supported languages**: Python only
- **Available models**: Gemini 2.5 Pro/Flash
- **Use cases**: Data analysis, math, algorithms
- **Limitations**: No network, file system, or external packages

## Enable Code Execution

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Enable code execution in request
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Calculate the factorial of 20 and show your work",
  tools: [{
    codeExecution: {}
  }]
});

console.log(response.text);
// Output includes both explanation and executed code results
```

## Data Analysis Example

```javascript
// Analyze data with code execution
const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `Analyze this sales data and create a summary:
    Month, Sales
    Jan, 45000
    Feb, 52000
    Mar, 48000
    Apr, 61000
    May, 58000
    Jun, 67000
    
    Calculate the average, find the trend, and predict July sales.`,
  tools: [{
    codeExecution: {}
  }]
});

console.log(response.text);
// Gemini will write and execute Python code to analyze the data
```

## Mathematical Computations

```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: `Solve this system of equations:
    2x + 3y = 13
    x - y = -1
    
    Show all steps and verify the solution.`,
  tools: [{
    codeExecution: {}
  }]
});

// Response includes Python code solving the equations
```

## Algorithm Implementation

```javascript
const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `Implement and test a binary search algorithm.
    Create a sorted list of 20 random numbers and search for specific values.
    Show the algorithm's efficiency.`,
  tools: [{
    codeExecution: {}
  }]
});

// Gemini writes, executes, and tests the algorithm
```

## Code Execution Handler

```javascript
class CodeExecutor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async executeWithCode(prompt, model = "gemini-2.5-flash") {
    try {
      const response = await this.ai.models.generateContent({
        model,
        contents: prompt,
        tools: [{
          codeExecution: {}
        }]
      });

      // Parse response to extract code blocks
      const result = {
        explanation: response.text,
        codeBlocks: this.extractCodeBlocks(response.text),
        fullResponse: response
      };

      return result;
    } catch (error) {
      console.error("Code execution error:", error);
      throw error;
    }
  }

  extractCodeBlocks(text) {
    const codeBlocks = [];
    const regex = /```python\n([\s\S]*?)```/g;
    let match;

    while ((match = regex.exec(text)) !== null) {
      codeBlocks.push({
        language: 'python',
        code: match[1].trim()
      });
    }

    return codeBlocks;
  }

  async analyzeDataset(data, analysisType) {
    const prompts = {
      statistical: `Perform statistical analysis on this dataset: ${data}
        Calculate mean, median, mode, standard deviation, and create a distribution summary.`,
      
      trend: `Analyze trends in this time series data: ${data}
        Identify patterns, calculate growth rates, and make predictions.`,
      
      correlation: `Find correlations in this dataset: ${data}
        Calculate correlation coefficients and identify significant relationships.`
    };

    const prompt = prompts[analysisType] || prompts.statistical;
    return await this.executeWithCode(prompt);
  }
}
```

## Data Visualization Tasks

```javascript
// Request data visualization
const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `Create a visualization for this data:
    Categories: A, B, C, D, E
    Values: 23, 45, 12, 67, 34
    
    Generate code to create a bar chart and pie chart.
    Calculate percentages for each category.`,
  tools: [{
    codeExecution: {}
  }]
});

// Note: Actual chart rendering not available, but calculations provided
```

## Complex Problem Solving

```javascript
class ProblemSolver {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async solveMathProblem(problem) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Solve this problem step by step with code:
        ${problem}
        
        Show all calculations, verify the answer, and explain the approach.`,
      tools: [{
        codeExecution: {}
      }]
    });

    return response.text;
  }

  async optimizationProblem(constraints, objective) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: `Solve this optimization problem:
        Objective: ${objective}
        Constraints: ${constraints}
        
        Use appropriate algorithms and show the optimal solution.`,
      tools: [{
        codeExecution: {}
      }]
    });

    return response.text;
  }

  async generateTestCases(functionDescription) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: `Generate comprehensive test cases for: ${functionDescription}
        
        Include edge cases, normal cases, and error cases.
        Implement the function and run all tests.`,
      tools: [{
        codeExecution: {}
      }]
    });

    return response.text;
  }
}
```

## Scientific Computing

```javascript
// Physics calculations
const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `Calculate the trajectory of a projectile:
    Initial velocity: 50 m/s
    Launch angle: 45 degrees
    
    Find: maximum height, range, time of flight
    Plot the trajectory path.`,
  tools: [{
    codeExecution: {}
  }]
});

// Chemistry calculations
const response2 = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: `Balance this chemical equation and calculate molarity:
    H2SO4 + NaOH → Na2SO4 + H2O
    
    If we have 250ml of 0.5M H2SO4, how much 0.1M NaOH is needed?`,
  tools: [{
    codeExecution: {}
  }]
});
```

## Limitations and Workarounds

```javascript
// Code execution limitations
const limitations = {
  noNetwork: "Cannot make HTTP requests or access internet",
  noFileSystem: "Cannot read/write files",
  noExternalPackages: "Only standard Python library available",
  timeLimit: "Execution time is limited",
  memoryLimit: "Memory usage is restricted"
};

// Workaround example - provide data inline
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: `Analyze this CSV data (provided inline since file access isn't available):
    name,age,score
    Alice,25,85
    Bob,30,92
    Charlie,28,78
    
    Calculate statistics and identify patterns.`,
  tools: [{
    codeExecution: {}
  }]
});
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: prompt,
    tools: [{
      codeExecution: {}
    }]
  });
  
  return response.text;
} catch (error) {
  if (error.message.includes("not supported")) {
    console.error("Code execution not supported for this model");
  } else if (error.message.includes("timeout")) {
    console.error("Code execution timed out");
  } else {
    throw error;
  }
}
```

## Best Practices

1. **Clear Instructions**: Specify exactly what to calculate
2. **Data Format**: Provide data in easily parseable format
3. **Verification**: Ask for answer verification
4. **Step-by-Step**: Request detailed explanations
5. **Error Handling**: Handle execution failures gracefully

## See Also
- [Function Calling](function-calling.md) - External function execution
- [Structured Output](structured-output.md) - JSON generation
- [Text Generation](text-generation.md) - Regular generation