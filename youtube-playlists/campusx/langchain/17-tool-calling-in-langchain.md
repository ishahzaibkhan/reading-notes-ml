# LangChain Tool Calling

## Introduction

This guide covers **Tool Calling**, a fundamental concept for building AI Agents with LangChain. Tool Calling enables LLMs to interact with external systems and perform actions beyond their parametric knowledge.

---

## LLM Capabilities & Limitations

### What LLMs Can Do
- **Reasoning**: Break down and understand complex questions
- **Output Generation**: Access parametric knowledge to generate answers
- **Analogy**: Like humans who can think and speak

### What LLMs Cannot Do
- Perform direct actions (e.g., modify databases, post on social media, make API calls)
- Access real-time information
- Execute commands or interact with external systems
- **Analogy**: Like humans who can think and speak but lack hands and feet to perform actions

---

## Tools: The Solution

**Tools** are explicitly created entities designed to carry out specific tasks that LLMs cannot perform on their own.

### Examples of Tools
- **DuckDuckGo Search**: Searches the web
- **Shell Tool**: Runs command-line commands
- **Custom Tools**: User-created tools for specific needs

### Nature of Tools
Tools are essentially special Python functions capable of interacting with LLMs when needed.

---

## 1. Tool Binding

### Definition
**Tool Binding** is the process of connecting or registering a tool with an LLM.

### Purpose of Binding
- LLM knows which tools are available
- LLM understands what each tool does (via its description)
- LLM knows the input format (schema) each tool expects

### Creating a Tool
```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Given two numbers A and B, this tool returns their product."""
    return a * b
```

### Invoking a Tool
Tools are runnables and can be invoked directly:
```python
multiply.invoke({"a": 3, "b": 4})  # Output: 12
```

### Accessing Tool Attributes
```python
multiply.name          # Output: 'multiply'
multiply.description   # Output: 'Given two numbers A and B, this tool returns their product.'
multiply.args          # Output: {'a': {'type': 'integer'}, 'b': {'type': 'integer'}}
```

### Binding Tools to an LLM
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
llm_with_tools = llm.bind_tools([multiply])
```

**Note**: Not all LLMs support tool binding. Check compatibility before implementation.

---

## 2. Tool Calling

### Definition
**Tool Calling** is the process where the LLM decides it needs to use a specific tool and generates a structured output containing:
- The tool's name
- The arguments to call it with

### How It Works

#### Scenario 1: Normal Question (No Tool Needed)
```python
llm_with_tools.invoke("Hi, how are you?")
```

**Output**:
```python
AIMessage(content='Hello! I am an AI assistant, and I am here to help you. How can I assist you today?')
```
- `content` field is populated
- `tool_calls` is empty

#### Scenario 2: Question Requiring a Tool
```python
ai_message = llm_with_tools.invoke("Can you multiply 3 with 10")
```

**Output**:
```python
AIMessage(content='', tool_calls=[{'name': 'multiply', 'args': {'a': 3, 'b': 10}, 'id': 'call_...'}])
```
- `content` field is empty
- `tool_calls` is populated

### Accessing Tool Call Details
```python
ai_message.tool_calls[0]
# Output: {'name': 'multiply', 'args': {'a': 3, 'b': 10}, 'id': 'call_...'}
```

### Critical Point: LLM Does NOT Execute the Tool
- The LLM only **suggests** the tool and its input arguments
- Actual execution is handled by LangChain or the programmer
- This provides control and prevents risky operations
- The LLM acts as an "advisor" rather than executor

---

## 3. Tool Execution

### Definition
**Tool Execution** is the step where the actual Python function (tool) is run using the input arguments suggested by the LLM.

### Process
1. Take the `tool_call` object from the LLM
2. Extract the tool name and arguments
3. Manually invoke the tool with extracted arguments

### Implementation
```python
# Retrieve the tool call
tool_call = ai_message.tool_calls[0]

# Execute the tool
tool_result = multiply.invoke(tool_call["args"])  # Output: 30
```

### Tool Message
The result is encapsulated in a **ToolMessage** that can be sent back to the LLM:
```python
from langchain_core.messages import ToolMessage

tool_message = ToolMessage(
    content=str(tool_result),
    tool_call_id=tool_call["id"]
)
```

---

## 4. Orchestrating the Complete Flow

The complete conversation flow maintains a `messages` list to track history:

### Step 1: Human Message
```python
from langchain_core.messages import HumanMessage

query = "Can you multiply 3 with 1000?"
messages = [HumanMessage(content=query)]
```

### Step 2: AI Message (Tool Call)
```python
ai_message = llm_with_tools.invoke(messages)
messages.append(ai_message)
```

### Step 3: Tool Execution & Tool Message
```python
from langchain_core.messages import ToolMessage

tool_call = ai_message.tool_calls[0]
tool_result = multiply.invoke(tool_call["args"])
messages.append(ToolMessage(content=str(tool_result), tool_call_id=tool_call["id"]))
```

### Step 4: Final LLM Response
```python
final_response = llm_with_tools.invoke(messages)
print(final_response.content)  # Output: "The product of 3 and 1000 is 3000."
```

**Message Flow**:
1. HumanMessage (user query)
2. AIMessage (tool call suggestion)
3. ToolMessage (tool execution result)
4. AIMessage (final coherent answer)

---

## 5. Real-World Example: Currency Conversion

### Problem
LLMs have outdated or no real-time currency conversion rates.

### Solution
Create tools that fetch real-time data from external APIs.

### API Used
**ExchangeRate-API**: `https://www.exchangerate-api.com/`
- Takes: `source_currency`, `target_currency`, `API_KEY`
- Returns: Current conversion rate

### Tool 1: Get Conversion Factor
```python
import requests

api_key = "YOUR_API_KEY"

@tool
def get_conversion_factor(base_currency: str, target_currency: str) -> float:
    """This function fetches the currency conversion factor between a given base currency and a target currency."""
    url = f"https://v6.exchangerate-api.com/v6/{api_key}/pair/{base_currency}/{target_currency}"
    response = requests.get(url)
    data = response.json()
    return data['conversion_rate']
```

### Tool 2: Convert Currency
```python
@tool
def convert(base_currency_value: int, conversion_rate: float) -> float:
    """Given a currency conversion rate, this function calculates the target currency value from a given base currency value."""
    return base_currency_value * conversion_rate
```

---

## 6. Problem: Simultaneous Tool Calls

### The Issue
When the LLM receives: *"What is the conversion factor between USD and INR and can you convert 10 USD to INR?"*

**What happens**:
1. LLM correctly identifies it needs `get_conversion_factor` first
2. For the `convert` tool, it needs `conversion_rate` as input
3. Since `get_conversion_factor` hasn't executed yet, the LLM falls back to its parametric knowledge
4. It provides an outdated or incorrect `conversion_rate` (e.g., 73.73 instead of real-time value)
5. This breaks the intended sequential logic

---

## 7. Solution: InjectedToolArgument

### Purpose
Prevent the LLM from "guessing" values for arguments that should come from preceding tool executions.

### Implementation
```python
from typing import Annotated
from langchain_core.runnables import InjectedToolArgument

@tool
def convert(
    base_currency_value: int,
    conversion_rate: Annotated[float, InjectedToolArgument]
) -> float:
    """Given a currency conversion rate, this function calculates the target currency value from a given base currency value."""
    return base_currency_value * conversion_rate
```

### How It Works
- `InjectedToolArgument` tells the LLM: *"Do not fill this argument; the developer will inject it"*
- When the LLM suggests tool calls, `conversion_rate` will be empty
- The programmer injects the value after executing `get_conversion_factor`

---

## 8. Multi-Tool Call Flow

### Updated Orchestration with Sequential Tools

**Message Flow**:
1. **HumanMessage**: Initial query
2. **AIMessage**: Contains two ToolCall objects:
   - `get_conversion_factor` (with arguments)
   - `convert` (with `conversion_rate` left blank)
3. **ToolMessage**: Result from `get_conversion_factor`
4. **ToolMessage**: Result from `convert` (after injecting the rate)
5. **AIMessage**: Final answer based on full context

### Execution Steps
1. Iterate through `ai_message.tool_calls`
2. Execute tools sequentially
3. Inject results from earlier tools into later tools
4. Feed complete conversation history back to LLM
5. Generate final coherent answer

---

## Key Takeaways

1. **Tool Creation**: Define tools using the `@tool` decorator on Python functions with clear docstrings and type hints
2. **Tool Binding**: Connects tools to LLMs, informing them of available capabilities
3. **Tool Calling**: LLM suggests which tool to use and with what arguments
4. **Tool Execution**: Programmer/framework actually runs the tool
5. **Control**: Programmer maintains control over tool execution for safety
6. **Sequential Logic**: Use `InjectedToolArgument` for tools that depend on results from other tools
7. **Message History**: Maintain conversation context for coherent responses

---

## Best Practices

- Always set `temperature=0` for consistent tool calling behavior
- Provide clear, descriptive docstrings for tools
- Use type hints for proper schema generation
- Maintain message history for context
- Use `InjectedToolArgument` for dependent tool chains
- Never fully trust LLM-generated tool arguments for critical operations
- Verify tool compatibility with your chosen LLM
