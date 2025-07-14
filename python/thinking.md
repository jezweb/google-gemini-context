# Thinking in Python

*Enable reasoning mode for complex problem solving*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/thinking-mode
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-pro (thinking mode)
Key Features: Step-by-step reasoning, problem decomposition
-->

## Quick Reference
- **Model**: `gemini-2.5-pro` only
- **Config**: `thinking_config` parameter
- **Use cases**: Complex reasoning, math, analysis
- **Output**: Visible thinking process + final answer

## Enable Thinking Mode

```python
from google import genai

client = genai.Client()

# Enable thinking mode
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="Solve this complex math problem: If a train travels 120 km in 1.5 hours, and another train travels 200 km in 2 hours, which is faster and by how much?",
    config={
        "thinking_config": {
            "include_thinking_process": True
        }
    }
)

print("Thinking process:")
print(response.thinking)
print("\nFinal answer:")
print(response.text)
```

## Complex Problem Solving

```python
class ThinkingAssistant:
    def __init__(self):
        self.client = genai.Client()
    
    def solve_problem(self, problem, show_thinking=True):
        """Solve complex problems with reasoning"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=problem,
            config={
                "thinking_config": {
                    "include_thinking_process": show_thinking
                }
            }
        )
        
        return {
            "thinking": response.thinking if show_thinking else None,
            "answer": response.text
        }
    
    def analyze_scenario(self, scenario):
        """Analyze complex scenarios step-by-step"""
        prompt = f"""Analyze this scenario comprehensively:
        {scenario}
        
        Consider all angles, potential outcomes, and provide recommendations."""
        
        return self.solve_problem(prompt)
    
    def debug_logic(self, problem_statement):
        """Debug logical problems with visible reasoning"""
        prompt = f"""Debug this logical problem step by step:
        {problem_statement}
        
        Show your reasoning process and identify any flaws."""
        
        return self.solve_problem(prompt)

# Usage
assistant = ThinkingAssistant()

# Solve complex problem
result = assistant.solve_problem(
    "A company has 3 departments. Sales has 12 people earning average $50k. Marketing has 8 people earning average $60k. Engineering has 15 people earning average $80k. If the company wants to give a 10% raise to all employees but needs to stay within a budget increase of $180k, is this possible?"
)

print("Thinking process:")
print(result["thinking"])
print("\nFinal answer:")
print(result["answer"])
```

## Mathematical Reasoning

```python
def solve_math_with_reasoning(problem):
    """Solve mathematical problems with step-by-step reasoning"""
    client = genai.Client()
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=f"Solve this mathematical problem step by step: {problem}",
        config={
            "thinking_config": {
                "include_thinking_process": True
            }
        }
    )
    
    return response

# Example: Complex calculus problem
result = solve_math_with_reasoning(
    "Find the area between the curves y = x² and y = 2x - x² from x = 0 to x = 1"
)

print("Mathematical reasoning:")
print(result.thinking)
print("\nSolution:")
print(result.text)
```

## Strategic Analysis

```python
class StrategyAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_business_case(self, case_study):
        """Analyze business cases with structured thinking"""
        prompt = f"""Analyze this business case:
        {case_study}
        
        Provide:
        1. Situation analysis
        2. Key challenges
        3. Available options
        4. Recommended strategy
        5. Implementation plan
        6. Risk assessment"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            config={
                "thinking_config": {
                    "include_thinking_process": True
                }
            }
        )
        
        return {
            "reasoning": response.thinking,
            "analysis": response.text
        }
    
    def compare_alternatives(self, options, criteria):
        """Compare alternatives with visible reasoning"""
        prompt = f"""Compare these alternatives:
        Options: {options}
        Criteria: {criteria}
        
        Provide detailed comparison with pros/cons and recommendation."""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            config={
                "thinking_config": {
                    "include_thinking_process": True
                }
            }
        )
        
        return {
            "thought_process": response.thinking,
            "comparison": response.text
        }

# Usage
analyzer = StrategyAnalyzer()

result = analyzer.compare_alternatives(
    options="Build in-house vs Buy existing solution vs Partner with vendor",
    criteria="Cost, time to market, control, scalability, risk"
)

print("Decision reasoning:")
print(result["thought_process"])
print("\nFinal comparison:")
print(result["comparison"])
```

## Logic Puzzle Solving

```python
def solve_logic_puzzle(puzzle):
    """Solve logic puzzles with visible reasoning"""
    client = genai.Client()
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=f"""Solve this logic puzzle step by step:
        {puzzle}
        
        Show your deductive reasoning process clearly.""",
        config={
            "thinking_config": {
                "include_thinking_process": True
            }
        }
    )
    
    return response

# Example: Classic logic puzzle
puzzle = """
Three friends - Alice, Bob, and Carol - each have a different pet (cat, dog, bird) and live in different colored houses (red, blue, green).

Clues:
1. Alice doesn't live in the red house
2. The person with the cat lives in the blue house
3. Bob doesn't have the bird
4. Carol doesn't live in the green house
5. The person in the red house has the dog

Who has which pet and lives in which house?
"""

result = solve_logic_puzzle(puzzle)
print("Logical reasoning:")
print(result.thinking)
print("\nSolution:")
print(result.text)
```

## Ethical Reasoning

```python
class EthicalReasoner:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_ethical_dilemma(self, dilemma):
        """Analyze ethical dilemmas with comprehensive reasoning"""
        prompt = f"""Analyze this ethical dilemma:
        {dilemma}
        
        Consider:
        - Stakeholders involved
        - Potential consequences
        - Ethical frameworks (utilitarian, deontological, virtue ethics)
        - Cultural considerations
        - Provide balanced recommendation"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            config={
                "thinking_config": {
                    "include_thinking_process": True
                }
            }
        )
        
        return {
            "reasoning_process": response.thinking,
            "ethical_analysis": response.text
        }

# Usage
reasoner = EthicalReasoner()

result = reasoner.analyze_ethical_dilemma(
    "A self-driving car's AI must choose between hitting one person to save five others. How should it be programmed to decide?"
)

print("Ethical reasoning:")
print(result["reasoning_process"])
print("\nAnalysis:")
print(result["ethical_analysis"])
```

## Scientific Hypothesis Testing

```python
def analyze_hypothesis(hypothesis, data):
    """Analyze scientific hypotheses with reasoning"""
    client = genai.Client()
    
    prompt = f"""Analyze this scientific hypothesis:
    Hypothesis: {hypothesis}
    Data: {data}
    
    Evaluate:
    1. Hypothesis validity
    2. Data quality and relevance
    3. Statistical significance
    4. Alternative explanations
    5. Conclusions and next steps"""
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=prompt,
        config={
            "thinking_config": {
                "include_thinking_process": True
            }
        }
    )
    
    return response

# Example
result = analyze_hypothesis(
    hypothesis="Increased screen time leads to decreased sleep quality in teenagers",
    data="Survey of 500 teens: avg 6.2 hours screen time, 6.8 hours sleep, correlation coefficient -0.67"
)

print("Scientific reasoning:")
print(result.thinking)
print("\nHypothesis evaluation:")
print(result.text)
```

## Streaming Thinking Mode

```python
def stream_thinking_process(problem):
    """Stream thinking process in real-time"""
    client = genai.Client()
    
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=problem,
        config={
            "thinking_config": {
                "include_thinking_process": True
            },
            "stream": True
        }
    )
    
    print("Thinking process (streaming):")
    thinking_text = ""
    final_answer = ""
    
    for chunk in response:
        if hasattr(chunk, 'thinking') and chunk.thinking:
            thinking_text += chunk.thinking
            print(chunk.thinking, end="", flush=True)
        elif chunk.text:
            final_answer += chunk.text
    
    print("\n\nFinal answer:")
    print(final_answer)
    
    return {
        "thinking": thinking_text,
        "answer": final_answer
    }

# Usage
stream_thinking_process(
    "Design an optimal algorithm for sorting a million integers with limited memory"
)
```

## Multi-Step Reasoning

```python
class MultiStepReasoner:
    def __init__(self):
        self.client = genai.Client()
    
    def break_down_problem(self, complex_problem):
        """Break down complex problems into steps"""
        prompt = f"""Break down this complex problem into manageable steps:
        {complex_problem}
        
        For each step:
        1. Identify what needs to be solved
        2. What information is needed
        3. How to approach it
        4. Expected outcome"""
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=prompt,
            config={
                "thinking_config": {
                    "include_thinking_process": True
                }
            }
        )
        
        return response
    
    def solve_step_by_step(self, problem_steps):
        """Solve each step with reasoning"""
        results = []
        
        for i, step in enumerate(problem_steps, 1):
            response = self.client.models.generate_content(
                model="gemini-2.5-pro",
                contents=f"Step {i}: {step}",
                config={
                    "thinking_config": {
                        "include_thinking_process": True
                    }
                }
            )
            
            results.append({
                "step": i,
                "problem": step,
                "thinking": response.thinking,
                "solution": response.text
            })
        
        return results
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-pro",
        contents=problem,
        config={
            "thinking_config": {
                "include_thinking_process": True
            }
        }
    )
    
    print("Thinking:", response.thinking)
    print("Answer:", response.text)
    
except Exception as error:
    if "thinking mode not supported" in str(error):
        print("Thinking mode only available on gemini-2.5-pro")
        # Fallback to regular generation
        response = client.models.generate_content(
            model="gemini-2.5-pro",
            contents=problem
        )
        print("Answer:", response.text)
    else:
        raise
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