# Function Calling in Python

*Enable Gemini to use tools and external functions*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/function-calling
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Changes: New client pattern, tool definitions in request
-->

## Quick Reference
- **Define tools**: Describe functions for model to call
- **Automatic mode**: Model decides when to call
- **Manual mode**: You control function execution
- **Response handling**: Check for function calls in response

## Basic Function Definition

```python
from google import genai

client = genai.Client()

# Define function
def get_weather(location: str, unit: str = "celsius") -> dict:
    """Get current weather for a location"""
    # Simulate API call
    return {
        "location": location,
        "temperature": 22,
        "unit": unit,
        "conditions": "Partly cloudy"
    }

# Define tool
tools = [{
    "function_declarations": [{
        "name": "get_weather",
        "description": "Get current weather for a location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name or coordinates"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit"
                }
            },
            "required": ["location"]
        }
    }]
}]
```

## Manual Function Calling

```python
# Send message that might trigger function
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What's the weather in Tokyo?",
    tools=tools
)

# Check for function calls
function_call = None
for candidate in response.candidates:
    for part in candidate.content.parts:
        if hasattr(part, 'function_call'):
            function_call = part.function_call
            break

if function_call:
    # Execute function
    if function_call.name == "get_weather":
        api_response = get_weather(**function_call.args)
    
    # Send function response back
    response2 = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[
            {"role": "user", "parts": [{"text": "What's the weather in Tokyo?"}]},
            {"role": "model", "parts": [{"function_call": function_call}]},
            {"role": "function", "parts": [{
                "function_response": {
                    "name": function_call.name,
                    "response": api_response
                }
            }]}
        ],
        tools=tools
    )
    
    print(response2.text)
```

## Multiple Functions

```python
# Define multiple functions
def get_time(timezone: str) -> dict:
    """Get current time in timezone"""
    from datetime import datetime
    import pytz
    
    tz = pytz.timezone(timezone)
    time = datetime.now(tz).strftime("%Y-%m-%d %H:%M:%S")
    return {"time": time, "timezone": timezone}

def calculate(expression: str) -> dict:
    """Calculate math expressions"""
    try:
        # Note: eval is unsafe in production!
        result = eval(expression)
        return {"result": result, "expression": expression}
    except:
        return {"error": "Invalid expression"}

# Define tools
tools = [{
    "function_declarations": [
        {
            "name": "get_weather",
            "description": "Get weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        },
        {
            "name": "get_time",
            "description": "Get current time in timezone",
            "parameters": {
                "type": "object",
                "properties": {
                    "timezone": {"type": "string"}
                },
                "required": ["timezone"]
            }
        },
        {
            "name": "calculate",
            "description": "Calculate math expressions",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string"}
                },
                "required": ["expression"]
            }
        }
    ]
}]

# Map function names to implementations
functions = {
    "get_weather": get_weather,
    "get_time": get_time,
    "calculate": calculate
}
```

## Function Calling Handler

```python
from typing import Dict, List, Any, Callable

class FunctionHandler:
    def __init__(self, functions: Dict[str, Callable], tools: List[Dict]):
        self.client = genai.Client()
        self.functions = functions
        self.tools = tools
        self.history = []
    
    def send_message(self, message: str) -> str:
        # Add user message to history
        self.history.append({"role": "user", "parts": [{"text": message}]})
        
        # Generate response
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=self.history,
            tools=self.tools
        )
        
        # Handle function calls
        function_call = self._extract_function_call(response)
        while function_call:
            # Add model's function call to history
            self.history.append({"role": "model", "parts": [{"function_call": function_call}]})
            
            # Execute function
            function_response = self.execute_function(function_call)
            
            # Add function response to history
            self.history.append({"role": "function", "parts": [function_response]})
            
            # Get next response
            response = self.client.models.generate_content(
                model="gemini-2.5-flash",
                contents=self.history,
                tools=self.tools
            )
            
            function_call = self._extract_function_call(response)
        
        # Add final response to history
        final_text = response.text
        self.history.append({"role": "model", "parts": [{"text": final_text}]})
        
        return final_text
    
    def _extract_function_call(self, response):
        """Extract function call from response"""
        for candidate in response.candidates:
            for part in candidate.content.parts:
                if hasattr(part, 'function_call'):
                    return part.function_call
        return None
    
    def execute_function(self, function_call) -> Dict:
        """Execute a function call"""
        try:
            func = self.functions.get(function_call.name)
            if not func:
                raise ValueError(f"Function {function_call.name} not found")
            
            result = func(**function_call.args)
            
            return {
                "function_response": {
                    "name": function_call.name,
                    "response": result
                }
            }
        except Exception as e:
            return {
                "function_response": {
                    "name": function_call.name,
                    "response": {"error": str(e)}
                }
            }

# Usage
handler = FunctionHandler(functions, tools)
response = handler.send_message(
    "What's the weather in Tokyo and what time is it there?"
)
print(response)
```

## Real-World Example: Database Query

```python
import sqlite3
from typing import List, Dict, Any

class DatabaseFunctions:
    def __init__(self, db_path: str):
        self.db_path = db_path
    
    def query_database(self, query: str, table: str) -> dict:
        """Execute SQL query on database"""
        try:
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            # Validate table exists
            cursor.execute(
                "SELECT name FROM sqlite_master WHERE type='table' AND name=?",
                (table,)
            )
            if not cursor.fetchone():
                return {"error": f"Table {table} not found"}
            
            # Execute query
            cursor.execute(query)
            columns = [desc[0] for desc in cursor.description]
            rows = cursor.fetchall()
            
            conn.close()
            
            return {
                "columns": columns,
                "rows": rows,
                "count": len(rows)
            }
        except Exception as e:
            return {"error": str(e)}
    
    def get_schema(self, table: str) -> dict:
        """Get table schema"""
        try:
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            
            cursor.execute(f"PRAGMA table_info({table})")
            columns = cursor.fetchall()
            
            conn.close()
            
            return {
                "columns": [
                    {"name": col[1], "type": col[2]}
                    for col in columns
                ]
            }
        except Exception as e:
            return {"error": str(e)}

# Define tools for database
db_tools = [{
    "function_declarations": [
        {
            "name": "query_database",
            "description": "Execute SQL query on database",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "SQL query"},
                    "table": {"type": "string", "description": "Table name"}
                },
                "required": ["query", "table"]
            }
        },
        {
            "name": "get_schema",
            "description": "Get table schema",
            "parameters": {
                "type": "object",
                "properties": {
                    "table": {"type": "string"}
                },
                "required": ["table"]
            }
        }
    ]
}]
```

## Streaming with Functions

```python
# Note: Function calling with streaming requires special handling
async def stream_with_functions(message: str):
    # Stream initial response
    response_stream = client.models.generate_content_stream(
        model="gemini-2.5-flash",
        contents=message,
        tools=tools
    )
    
    # Collect full response
    full_response = ""
    function_call = None
    
    async for chunk in response_stream:
        if chunk.text:
            full_response += chunk.text
            print(chunk.text, end='', flush=True)
        
        # Check for function calls
        for candidate in chunk.candidates:
            for part in candidate.content.parts:
                if hasattr(part, 'function_call'):
                    function_call = part.function_call
    
    # Handle function call if present
    if function_call:
        # Execute and continue conversation
        pass
```

## Error Handling

```python
def safe_function_execution(function_call, functions):
    """Safely execute function with error handling"""
    try:
        func = functions.get(function_call.name)
        if not func:
            return {
                "function_response": {
                    "name": function_call.name,
                    "response": {"error": "Function not found"}
                }
            }
        
        # Validate parameters
        result = func(**function_call.args)
        
        return {
            "function_response": {
                "name": function_call.name,
                "response": result
            }
        }
    except TypeError as e:
        return {
            "function_response": {
                "name": function_call.name,
                "response": {"error": f"Invalid parameters: {str(e)}"}
            }
        }
    except Exception as e:
        return {
            "function_response": {
                "name": function_call.name,
                "response": {"error": f"Execution failed: {str(e)}"}
            }
        }
```

## Best Practices

1. **Clear descriptions**: Help model understand when to use functions
2. **Type safety**: Validate parameters before execution
3. **Error handling**: Always catch and report errors gracefully
4. **Security**: Never execute untrusted code or queries
5. **Logging**: Track function calls for debugging

## See Also
- [Chat](chat.md) - Multi-turn with functions
- [Structured Output](structured-output.md) - JSON responses
- [Text Generation](text-generation.md) - Basic generation