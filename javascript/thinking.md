# Thinking in JavaScript

*Enable reasoning mode for complex problem solving*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/thinking-mode
SDK: @google/genai (JavaScript)
Verified: 2025-01-14
Models: gemini-2.5-pro (thinking mode)
Key Features: Step-by-step reasoning, problem decomposition
-->

## Quick Reference
- **Model**: `gemini-2.5-pro` only
- **Config**: `thinkingConfig` parameter
- **Use cases**: Complex reasoning, math, analysis
- **Output**: Visible thinking process + final answer

## Enable Thinking Mode

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});

// Enable thinking mode
const response = await ai.models.generateContent({
  model: "gemini-2.5-pro",
  contents: "Solve this complex math problem: If a train travels 120 km in 1.5 hours, and another train travels 200 km in 2 hours, which is faster and by how much?",
  config: {
    thinkingConfig: {
      includeThinkingProcess: true
    }
  }
});

console.log("Thinking process:");
console.log(response.thinking);
console.log("\nFinal answer:");
console.log(response.text);
```

## Complex Problem Solving

```javascript
class ThinkingAssistant {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async solveProblem(problem, showThinking = true) {
    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: problem,
      config: {
        thinkingConfig: {
          includeThinkingProcess: showThinking
        }
      }
    });

    return {
      thinking: showThinking ? response.thinking : null,
      answer: response.text
    };
  }

  async analyzeScenario(scenario) {
    const prompt = `Analyze this scenario comprehensively:
    ${scenario}
    
    Consider all angles, potential outcomes, and provide recommendations.`;

    return await this.solveProblem(prompt);
  }

  async debugLogic(problemStatement) {
    const prompt = `Debug this logical problem step by step:
    ${problemStatement}
    
    Show your reasoning process and identify any flaws.`;

    return await this.solveProblem(prompt);
  }
}

// Usage
const assistant = new ThinkingAssistant();

const result = await assistant.solveProblem(
  "A company has 3 departments. Sales has 12 people earning average $50k. Marketing has 8 people earning average $60k. Engineering has 15 people earning average $80k. If the company wants to give a 10% raise to all employees but needs to stay within a budget increase of $180k, is this possible?"
);

console.log("Thinking process:");
console.log(result.thinking);
console.log("\nFinal answer:");
console.log(result.answer);
```

## Mathematical Reasoning

```javascript
async function solveMathWithReasoning(problem) {
  const ai = new GoogleGenAI({});
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: `Solve this mathematical problem step by step: ${problem}`,
    config: {
      thinkingConfig: {
        includeThinkingProcess: true
      }
    }
  });

  return response;
}

// Example: Complex calculus problem
const result = await solveMathWithReasoning(
  "Find the area between the curves y = x² and y = 2x - x² from x = 0 to x = 1"
);

console.log("Mathematical reasoning:");
console.log(result.thinking);
console.log("\nSolution:");
console.log(result.text);
```

## Strategic Analysis

```javascript
class StrategyAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzeBusinessCase(caseStudy) {
    const prompt = `Analyze this business case:
    ${caseStudy}
    
    Provide:
    1. Situation analysis
    2. Key challenges
    3. Available options
    4. Recommended strategy
    5. Implementation plan
    6. Risk assessment`;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    return {
      reasoning: response.thinking,
      analysis: response.text
    };
  }

  async compareAlternatives(options, criteria) {
    const prompt = `Compare these alternatives:
    Options: ${options}
    Criteria: ${criteria}
    
    Provide detailed comparison with pros/cons and recommendation.`;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    return {
      thoughtProcess: response.thinking,
      comparison: response.text
    };
  }
}

// Usage
const analyzer = new StrategyAnalyzer();

const result = await analyzer.compareAlternatives(
  "Build in-house vs Buy existing solution vs Partner with vendor",
  "Cost, time to market, control, scalability, risk"
);

console.log("Decision reasoning:");
console.log(result.thoughtProcess);
console.log("\nFinal comparison:");
console.log(result.comparison);
```

## Logic Puzzle Solving

```javascript
async function solveLogicPuzzle(puzzle) {
  const ai = new GoogleGenAI({});
  
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: `Solve this logic puzzle step by step:
    ${puzzle}
    
    Show your deductive reasoning process clearly.`,
    config: {
      thinkingConfig: {
        includeThinkingProcess: true
      }
    }
  });

  return response;
}

// Example: Classic logic puzzle
const puzzle = `
Three friends - Alice, Bob, and Carol - each have a different pet (cat, dog, bird) and live in different colored houses (red, blue, green).

Clues:
1. Alice doesn't live in the red house
2. The person with the cat lives in the blue house
3. Bob doesn't have the bird
4. Carol doesn't live in the green house
5. The person in the red house has the dog

Who has which pet and lives in which house?
`;

const result = await solveLogicPuzzle(puzzle);
console.log("Logical reasoning:");
console.log(result.thinking);
console.log("\nSolution:");
console.log(result.text);
```

## Ethical Reasoning

```javascript
class EthicalReasoner {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzeEthicalDilemma(dilemma) {
    const prompt = `Analyze this ethical dilemma:
    ${dilemma}
    
    Consider:
    - Stakeholders involved
    - Potential consequences
    - Ethical frameworks (utilitarian, deontological, virtue ethics)
    - Cultural considerations
    - Provide balanced recommendation`;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    return {
      reasoningProcess: response.thinking,
      ethicalAnalysis: response.text
    };
  }
}

// Usage
const reasoner = new EthicalReasoner();

const result = await reasoner.analyzeEthicalDilemma(
  "A self-driving car's AI must choose between hitting one person to save five others. How should it be programmed to decide?"
);

console.log("Ethical reasoning:");
console.log(result.reasoningProcess);
console.log("\nAnalysis:");
console.log(result.ethicalAnalysis);
```

## Scientific Hypothesis Testing

```javascript
async function analyzeHypothesis(hypothesis, data) {
  const ai = new GoogleGenAI({});
  
  const prompt = `Analyze this scientific hypothesis:
  Hypothesis: ${hypothesis}
  Data: ${data}
  
  Evaluate:
  1. Hypothesis validity
  2. Data quality and relevance
  3. Statistical significance
  4. Alternative explanations
  5. Conclusions and next steps`;

  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: prompt,
    config: {
      thinkingConfig: {
        includeThinkingProcess: true
      }
    }
  });

  return response;
}

// Example
const result = await analyzeHypothesis(
  "Increased screen time leads to decreased sleep quality in teenagers",
  "Survey of 500 teens: avg 6.2 hours screen time, 6.8 hours sleep, correlation coefficient -0.67"
);

console.log("Scientific reasoning:");
console.log(result.thinking);
console.log("\nHypothesis evaluation:");
console.log(result.text);
```

## Streaming Thinking Mode

```javascript
async function streamThinkingProcess(problem) {
  const ai = new GoogleGenAI({});
  
  const response = await ai.models.generateContentStream({
    model: "gemini-2.5-pro",
    contents: problem,
    config: {
      thinkingConfig: {
        includeThinkingProcess: true
      }
    }
  });

  console.log("Thinking process (streaming):");
  let thinkingText = "";
  let finalAnswer = "";

  for await (const chunk of response) {
    if (chunk.thinking) {
      thinkingText += chunk.thinking;
      process.stdout.write(chunk.thinking);
    } else if (chunk.text) {
      finalAnswer += chunk.text;
    }
  }

  console.log("\n\nFinal answer:");
  console.log(finalAnswer);

  return {
    thinking: thinkingText,
    answer: finalAnswer
  };
}

// Usage
await streamThinkingProcess(
  "Design an optimal algorithm for sorting a million integers with limited memory"
);
```

## Multi-Step Reasoning

```javascript
class MultiStepReasoner {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async breakDownProblem(complexProblem) {
    const prompt = `Break down this complex problem into manageable steps:
    ${complexProblem}
    
    For each step:
    1. Identify what needs to be solved
    2. What information is needed
    3. How to approach it
    4. Expected outcome`;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: prompt,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    return response;
  }

  async solveStepByStep(problemSteps) {
    const results = [];

    for (let i = 0; i < problemSteps.length; i++) {
      const step = problemSteps[i];
      
      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-pro",
        contents: `Step ${i + 1}: ${step}`,
        config: {
          thinkingConfig: {
            includeThinkingProcess: true
          }
        }
      });

      results.push({
        step: i + 1,
        problem: step,
        thinking: response.thinking,
        solution: response.text
      });
    }

    return results;
  }
}
```

## Web Integration

```javascript
// Client-side thinking mode for web apps
class WebThinkingInterface {
  constructor(apiKey) {
    this.ai = new GoogleGenAI({ apiKey });
  }

  async displayThinkingProcess(problem, thinkingContainer, answerContainer) {
    const response = await this.ai.models.generateContentStream({
      model: "gemini-2.5-pro",
      contents: problem,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    for await (const chunk of response) {
      if (chunk.thinking) {
        thinkingContainer.innerHTML += chunk.thinking;
        thinkingContainer.scrollTop = thinkingContainer.scrollHeight;
      } else if (chunk.text) {
        answerContainer.innerHTML += chunk.text;
      }
    }
  }

  async analyzeWithProgress(problem, progressCallback) {
    const response = await this.ai.models.generateContentStream({
      model: "gemini-2.5-pro",
      contents: problem,
      config: {
        thinkingConfig: {
          includeThinkingProcess: true
        }
      }
    });

    let thinkingProgress = 0;
    let thinking = "";
    let answer = "";

    for await (const chunk of response) {
      if (chunk.thinking) {
        thinking += chunk.thinking;
        thinkingProgress += chunk.thinking.length;
        
        progressCallback({
          type: 'thinking',
          content: chunk.thinking,
          progress: thinkingProgress
        });
      } else if (chunk.text) {
        answer += chunk.text;
        
        progressCallback({
          type: 'answer',
          content: chunk.text,
          complete: false
        });
      }
    }

    progressCallback({
      type: 'complete',
      thinking,
      answer
    });

    return { thinking, answer };
  }
}
```

## Error Handling

```javascript
try {
  const response = await ai.models.generateContent({
    model: "gemini-2.5-pro",
    contents: problem,
    config: {
      thinkingConfig: {
        includeThinkingProcess: true
      }
    }
  });

  console.log("Thinking:", response.thinking);
  console.log("Answer:", response.text);

} catch (error) {
  if (error.message.includes("thinking mode not supported")) {
    console.log("Thinking mode only available on gemini-2.5-pro");
    
    // Fallback to regular generation
    const response = await ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: problem
    });
    
    console.log("Answer:", response.text);
  } else {
    throw error;
  }
}
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
- [Structured Output](structured-output.md) - JSON reasoning
- [Code Execution](code-execution.md) - Computational reasoning