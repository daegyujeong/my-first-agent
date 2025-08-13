# How the AI Chat Agent Works

This document explains the architecture and flow of the AI chat agent with function calling capabilities.

## Overview

The AI chat agent is built using OpenAI's function calling feature, which allows the AI to:
- Maintain conversation context
- Decide when to use tools/functions
- Process function results and provide natural responses

## Architecture Components

### 1. Conversation History (`messages`)
```python
messages = []
```
- Stores the entire conversation as a list of message objects
- Each message contains:
  - `role`: "user", "assistant", or "tool"
  - `content`: The actual message text
  - `tool_calls`: (optional) Information about function calls

### 2. Function Definitions
```python
def get_weather(city):
    return f"The weather of {city} is sunny"

FUNCTION_MAP = {
    "get_weather": get_weather,
}
```
- Python functions that perform actual tasks
- `FUNCTION_MAP` links function names to their implementations

### 3. Tool Descriptions (`TOOLS`)
```python
TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the weather of a country",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "The city to get the weather of"
                    }
                },
                "required": ["city"],
            }
        }
    }
]
```
- Describes available functions to the AI in OpenAI's schema format
- Helps AI understand when and how to use each function

## Process Flow

### Step 1: User Input
```python
message = input("You: ")
messages.append({"role": "user", "content": message})
```
User enters a message, which is added to the conversation history.

### Step 2: AI Processing
```python
def call_ai():
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        tools=TOOLS,
    )
    process_ai_response(response.choices[0].message)
```
The entire conversation history and available tools are sent to OpenAI.

### Step 3: Response Handling

The AI can respond in two ways:

#### Option A: Direct Text Response
If the AI doesn't need to use a tool:
```python
messages.append({"role": "assistant", "content": message.content})
print(f"AI: {message.content}")
```

#### Option B: Function Call
If the AI decides to use a tool:

1. **Record the tool call**:
   ```python
   messages.append({
       "role": "assistant",
       "content": message.content or "",
       "tool_calls": [...]
   })
   ```

2. **Execute the function**:
   ```python
   function_to_run = FUNCTION_MAP.get(function_name)
   result = function_to_run(**arguments)
   ```

3. **Add result to conversation**:
   ```python
   messages.append({
       "role": "tool",
       "tool_call_id": tool_call.id,
       "name": function_name,
       "content": result,
   })
   ```

4. **Call AI again** to process the result and generate final response

## Example Conversation Flow

```
User: "What's the weather in Spain?"
↓
AI: [Decides to call get_weather("Spain")]
↓
System: Executes get_weather("Spain") → "The weather of Spain is sunny"
↓
AI: "The weather in Spain is sunny today!"
```

## Key Concepts

1. **Conversation Memory**: The `messages` list maintains full context, allowing the AI to reference previous interactions.

2. **Tool Calling**: The AI doesn't execute functions directly - it requests function execution, and your code handles it.

3. **Recursive Processing**: After a function call, the AI is called again with the function result to generate the final response.

4. **Flexibility**: You can add any Python function as a tool by:
   - Writing the function
   - Adding it to `FUNCTION_MAP`
   - Describing it in `TOOLS`

## Benefits

- **Natural Interaction**: Users can ask questions naturally without knowing the underlying functions
- **Extensibility**: Easy to add new capabilities by adding functions
- **Context Awareness**: The AI understands the full conversation context
- **Intelligent Routing**: AI decides when to use tools vs. respond directly

## Limitations

- Functions must be pre-defined
- AI can only call functions you've explicitly made available
- Function execution happens locally in your Python environment