# Code Execution in Python

*Enable Gemini to write and execute code*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/code-execution
SDK: google-genai (Python)
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

```python
from google import genai

client = genai.Client()

# Enable code execution in request
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Calculate the factorial of 20 and show your work",
    tools=[{"code_execution": {}}]
)

print(response.text)
# Output includes both explanation and executed code results
```

## Data Analysis Example

```python
# Analyze data with code execution
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""Analyze this sales data and create a summary:
    Month, Sales
    Jan, 45000
    Feb, 52000
    Mar, 48000
    Apr, 61000
    May, 58000
    Jun, 67000
    
    Calculate the average, find the trend, and predict July sales.""",
    tools=[{"code_execution": {}}]
)

print(response.text)
# Gemini will write and execute Python code to analyze the data
```

## Mathematical Computations

```python
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="""Solve this system of equations:
    2x + 3y = 13
    x - y = -1
    
    Show all steps and verify the solution.""",
    tools=[{"code_execution": {}}]
)

print(response.text)
# Response includes Python code solving the equations
```

## Code Execution Handler

```python
import re
import json

class CodeExecutor:
    def __init__(self):
        self.client = genai.Client()
    
    def execute_with_code(self, prompt, model="gemini-2.5-flash"):
        """Execute prompt with code execution enabled"""
        try:
            response = self.client.models.generate_content(
                model=model,
                contents=prompt,
                tools=[{"code_execution": {}}]
            )
            
            # Parse response to extract code blocks
            result = {
                "explanation": response.text,
                "code_blocks": self.extract_code_blocks(response.text),
                "full_response": response
            }
            
            return result
        except Exception as error:
            print(f"Code execution error: {error}")
            raise error
    
    def extract_code_blocks(self, text):
        """Extract code blocks from response"""
        code_blocks = []
        # Match Python code blocks
        pattern = r"```python\n(.*?)```"
        matches = re.findall(pattern, text, re.DOTALL)
        
        for match in matches:
            code_blocks.append({
                "language": "python",
                "code": match.strip()
            })
        
        return code_blocks
    
    def analyze_dataset(self, data, analysis_type="statistical"):
        """Analyze dataset with different focus"""
        prompts = {
            "statistical": f"""Perform statistical analysis on this dataset: {data}
                Calculate mean, median, mode, standard deviation, and create a distribution summary.""",
            
            "trend": f"""Analyze trends in this time series data: {data}
                Identify patterns, calculate growth rates, and make predictions.""",
            
            "correlation": f"""Find correlations in this dataset: {data}
                Calculate correlation coefficients and identify significant relationships."""
        }
        
        prompt = prompts.get(analysis_type, prompts["statistical"])
        return self.execute_with_code(prompt)

# Usage
executor = CodeExecutor()

# Analyze data
result = executor.analyze_dataset(
    data="[10, 15, 12, 18, 22, 25, 28, 30, 35, 40]",
    analysis_type="statistical"
)

print(result["explanation"])
for block in result["code_blocks"]:
    print(f"Code: {block['code']}")
```

## Scientific Computing

```python
# Physics calculations
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""Calculate the trajectory of a projectile:
    Initial velocity: 50 m/s
    Launch angle: 45 degrees
    
    Find: maximum height, range, time of flight
    Plot the trajectory path.""",
    tools=[{"code_execution": {}}]
)

print(response.text)

# Chemistry calculations
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""Balance this chemical equation and calculate molarity:
    H2SO4 + NaOH → Na2SO4 + H2O
    
    If we have 250ml of 0.5M H2SO4, how much 0.1M NaOH is needed?""",
    tools=[{"code_execution": {}}]
)

print(response.text)
```

## Complex Problem Solving

```python
class ProblemSolver:
    def __init__(self):
        self.client = genai.Client()
    
    def solve_math_problem(self, problem):
        """Solve mathematical problems with code"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Solve this problem step by step with code:
            {problem}
            
            Show all calculations, verify the answer, and explain the approach.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text
    
    def optimization_problem(self, constraints, objective):
        """Solve optimization problems"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Solve this optimization problem:
            Objective: {objective}
            Constraints: {constraints}
            
            Use appropriate algorithms and show the optimal solution.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text
    
    def generate_test_cases(self, function_description):
        """Generate test cases for functions"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Generate comprehensive test cases for: {function_description}
            
            Include edge cases, normal cases, and error cases.
            Implement the function and run all tests.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text

# Usage
solver = ProblemSolver()

# Solve math problem
result = solver.solve_math_problem(
    "Find the roots of the quadratic equation 2x² - 5x + 3 = 0"
)
print(result)

# Optimization problem
result = solver.optimization_problem(
    constraints="x + y <= 10, x >= 0, y >= 0",
    objective="maximize 3x + 2y"
)
print(result)
```

## Data Visualization Tasks

```python
# Request data visualization
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""Create a visualization for this data:
    Categories: A, B, C, D, E
    Values: 23, 45, 12, 67, 34
    
    Generate code to create a bar chart and pie chart.
    Calculate percentages for each category.""",
    tools=[{"code_execution": {}}]
)

print(response.text)
# Note: Actual chart rendering not available, but calculations provided
```

## Algorithm Implementation

```python
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="""Implement and test a binary search algorithm.
    Create a sorted list of 20 random numbers and search for specific values.
    Show the algorithm's efficiency.""",
    tools=[{"code_execution": {}}]
)

print(response.text)
# Gemini writes, executes, and tests the algorithm
```

## Limitations and Workarounds

```python
# Code execution limitations
limitations = {
    "no_network": "Cannot make HTTP requests or access internet",
    "no_file_system": "Cannot read/write files",
    "no_external_packages": "Only standard Python library available",
    "time_limit": "Execution time is limited",
    "memory_limit": "Memory usage is restricted"
}

# Workaround example - provide data inline
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="""Analyze this CSV data (provided inline since file access isn't available):
    name,age,score
    Alice,25,85
    Bob,30,92
    Charlie,28,78
    
    Calculate statistics and identify patterns.""",
    tools=[{"code_execution": {}}]
)

print(response.text)
```

## Streaming Code Execution

```python
# Stream code execution responses
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Create a Monte Carlo simulation to estimate π",
    tools=[{"code_execution": {}}],
    config={"stream": True}
)

for chunk in response:
    print(chunk.text, end="", flush=True)
```

## Error Handling

```python
try:
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt,
        tools=[{"code_execution": {}}]
    )
    print(response.text)
except Exception as error:
    if "not supported" in str(error):
        print("Code execution not supported for this model")
    elif "timeout" in str(error):
        print("Code execution timed out")
    else:
        raise
```

## Advanced Code Analysis

```python
class CodeAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_algorithm_complexity(self, algorithm_description):
        """Analyze algorithm complexity"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Analyze the time and space complexity of: {algorithm_description}
            
            Implement the algorithm and demonstrate its complexity with test cases.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text
    
    def debug_code(self, code_snippet, error_description):
        """Debug code with execution"""
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=f"""Debug this code:
            {code_snippet}
            
            Error: {error_description}
            
            Fix the code and test the solution.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text
    
    def optimize_performance(self, code_snippet):
        """Optimize code performance"""
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=f"""Optimize this code for better performance:
            {code_snippet}
            
            Show before/after comparison with timing tests.""",
            tools=[{"code_execution": {}}]
        )
        
        return response.text
```

## Best Practices

1. **Clear Instructions**: Specify exactly what to calculate
2. **Data Format**: Provide data in easily parseable format
3. **Verification**: Ask for answer verification
4. **Step-by-Step**: Request detailed explanations
5. **Error Handling**: Handle execution failures gracefully
6. **Inline Data**: Provide data directly in prompts

## See Also
- [Function Calling](function-calling.md) - External function execution
- [Structured Output](structured-output.md) - JSON generation
- [Text Generation](text-generation.md) - Regular generation