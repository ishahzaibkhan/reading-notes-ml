# Tools in LangChain

## Overview
Tools are essential components in LangChain that enable Language Models to perform specific tasks and interact with external systems. This guide covers what tools are, their types, and how to create them.

## Part 3: Building Agents

This is the first video in the agent-building segment, which includes:
1. **Tools** - Different types and custom tool creation
2. **Tool Calling** - Connecting tools with LLMs
3. **Agents** - Building complete agent systems using LangChain and LangGraph

---

## What is a Tool?

### Understanding LLM Capabilities

**Core Strengths:**
- **Reasoning Capability** - Can understand, break down questions, and think about solutions
- **Language Generation** - Can generate coherent, word-by-word responses

**Limitations:**
LLMs cannot directly perform:
- Booking tickets or making reservations
- Fetching live weather data
- Reliable complex mathematical calculations
- Calling external APIs (e.g., posting to Twitter)
- Running code
- Interacting with databases

### The Role of Tools

**Analogy:** LLMs are like "a human body without hands and legs" - they can think but cannot act.

**Tools provide:**
- The "hands and legs" for LLMs to perform actual tasks
- Specific functionality that LLMs can leverage to accomplish real-world actions

**Technical Definition:**
Tools are Python functions with specific task logic, packaged in a way that LLMs can understand and interact with.

**How It Works:**
1. LLM receives a task from the user
2. LLM autonomously decides when to call a tool
3. LLM provides inputs to the tool
4. Tool executes the task
5. Tool returns results to the LLM
6. LLM informs the user of completion

**Key Concept:** The combination of LLMs and tools forms an **AI Agent**

---

## AI Agents

**Definition:** An LLM-powered system that can autonomously think, decide, and take actions using external tools or APIs to achieve a goal.

**Agent Components:**
- **Reasoning & Decision Making** - Provided by the LLM
- **Action Performance** - Provided by tools

---

## Types of Tools in LangChain

### 1. Built-in Tools
Pre-built, production-ready tools provided by the LangChain team for common tasks.

**Characteristics:**
- Minimal to no setup required
- Just import and use
- Cover common use cases

**Popular Examples:**
- **DuckDuckGo Search** - Web search functionality, useful for real-time news
- **Wikipedia Query Run** - Search Wikipedia and get summarized results
- **Python REPL Tool** - Run raw Python code reliably (e.g., calculating factorials)
- **Shell Tool** - Execute shell/console commands on the host machine
- **RequestsGet Tool** - Make HTTP requests
- **Gmail Send Message Tool** - Send emails via Gmail
- **Slack Tool** - Slack integrations
- **SQL Database Query Tool** - Database interactions

### 2. Custom Tools
User-created tools for specific, unique use cases.

**Use Cases:**
- Integrating with company-specific APIs
- Encapsulating unique business logic
- Interacting with existing databases, products, or applications

---

## Built-in Tool Examples

### DuckDuckGo Search Tool
```python
from langchain_community.tools import DuckDuckGoSearchRun

search_tool = DuckDuckGoSearchRun()
result = search_tool.invoke("IPL news")
```

### Shell Tool
```python
from langchain_community.tools import ShellTool

shell_tool = ShellTool()
result = shell_tool.invoke("whoami")
```

**Requirements:**
- Requires `langchain-experimental` package

**Caution:** Can be risky in production as it executes commands directly on the system

---

## Tool Properties

Every tool in LangChain has three essential attributes:

1. **name** - Usually the function's name
2. **description** - Taken from the function's docstring; helps the LLM understand the tool's purpose
3. **arguments** - A dictionary describing required inputs, including their types (derived from type hinting)

### Tool Schema
When connecting a tool to an LLM, the LLM receives a **JSON schema** of the tool, not the function logic itself. This schema describes:
- Tool's capabilities
- Required arguments and their properties
- Data types
- Whether arguments are required
- Descriptions of each argument

---

## Creating Custom Tools

### Method 1: Using @tool Decorator

**Simplest and most straightforward method**

**Steps:**

1. **Create a Python function** with task logic
2. **Add a docstring** describing the function's purpose (highly recommended)
3. **Add type hinting** for inputs and outputs (highly recommended)
4. **Apply @tool decorator**
```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiplies two numbers.
    
    Args:
        a: The first integer.
        b: The second integer.
    """
    return a * b

# Invoke the tool
result = multiply.invoke({"a": 5, "b": 3})
```

**Tool Attributes:**
The decorated function automatically gets `name`, `description`, and `arguments` attributes derived from the function name, docstring, and type hints.

---

### Method 2: Using StructuredTool Class with Pydantic

**More strict and mature way for defining tool inputs**

**Concept:** Input follows a structured schema defined using Pydantic models

**Steps:**

1. **Import required classes**
2. **Create a Pydantic model** defining the input schema
3. **Create a function** with the task logic
4. **Create StructuredTool instance** using `from_function()`
```python
from langchain.tools import StructuredTool
from pydantic import BaseModel, Field

# Define input schema
class MultiplyInput(BaseModel):
    a: int = Field(description="First number")
    b: int = Field(description="Second number")

# Create function
def multiply_function(a: int, b: int) -> int:
    return a * b

# Create StructuredTool
multiply_tool = StructuredTool.from_function(
    func=multiply_function,
    name="multiply",
    description="Multiply two numbers together",
    args_schema=MultiplyInput
)
```

**Benefit:** Allows for more explicit and validated input structures

---

### Method 3: Inheriting from BaseTool Class

**Most customizable approach**

**Concept:** `BaseTool` is an abstract base class that all tools in LangChain inherit from. You can directly inherit from it for highly customized tools.

**Steps:**

1. **Import BaseTool and Type**
2. **Create a custom class** inheriting from BaseTool
3. **Define attributes:** name, description, args_schema
4. **Define _run method** containing the core logic
5. **Optionally define _arun method** for asynchronous operations
```python
from langchain.tools import BaseTool
from typing import Type
from pydantic import BaseModel, Field

class MultiplyInput(BaseModel):
    a: int = Field(description="First number")
    b: int = Field(description="Second number")

class MultiplyTool(BaseTool):
    name: str = "multiply"
    description: str = "Multiply two numbers together"
    args_schema: Type[BaseModel] = MultiplyInput
    
    def _run(self, a: int, b: int) -> int:
        """Core logic function - must be named _run"""
        return a * b
    
    async def _arun(self, a: int, b: int) -> int:
        """Asynchronous version of the tool"""
        return a * b
```

**Advantages:**
- Deep level of customization
- Ability to create asynchronous version (`_arun` method) for concurrency
- Better suited for production-level applications

**When to Use:**
- **@tool decorator** - Sufficient for most basic and experimental scenarios
- **BaseTool inheritance** - Production-level applications requiring advanced features

---

## Toolkits

**Concept:** A collection of multiple related tools grouped together

**Purpose:** Simplifies agent setup and management, especially for suite of functionalities related to a specific domain

**Example Use Cases:**
- Google Drive Toolkit (uploading, searching, reading files)
- CRM system tools
- Financial API tools
- Arithmetic operations toolkit

### Creating a Toolkit

A toolkit is typically represented as a Python list containing Tool objects:
```python
from langchain_core.tools import tool, BaseTool
from pydantic import BaseModel, Field
from typing import Type

# Define multiple tools using @tool decorator
@tool
def multiply(a: int, b: int) -> int:
    """Multiplies two numbers.
    
    Args:
        a: The first integer.
        b: The second integer.
    """
    return a * b

@tool
def divide(a: int, b: int) -> float:
    """Divides two numbers.
    
    Args:
        a: The numerator integer.
        b: The denominator integer. (Cannot be zero)
    """
    if b == 0:
        raise ValueError("Cannot divide by zero.")
    return a / b

# Define a tool using BaseTool class
class PowerCalculatorInput(BaseModel):
    base: int = Field(description="The base number")
    exponent: int = Field(description="The exponent")

class PowerCalculatorTool(BaseTool):
    name: str = "power_calculator"
    description: str = "Calculates the power of a base number raised to an exponent."
    args_schema: Type[BaseModel] = PowerCalculatorInput
    
    def _run(self, base: int, exponent: int) -> int:
        """Use the tool synchronously."""
        return base ** exponent
    
    async def _arun(self, base: int, exponent: int) -> int:
        """Use the tool asynchronously."""
        return base ** exponent

# Create the toolkit as a list of tools
arithmetic_tools = [
    multiply,
    divide,
    PowerCalculatorTool()  # Instantiate the BaseTool class
]

# Display toolkit tools
print("--- Toolkit Tools ---")
for t in arithmetic_tools:
    print(f"Tool Name: {t.name}, Description: {t.description}")
```

### Using Toolkits with Agents

Toolkits are passed to agents during initialization. The agent can then choose which tool to use based on user queries:
```python
from langchain.agents import initialize_agent, AgentType
from langchain_openai import ChatOpenAI

# Initialize LLM
llm = ChatOpenAI(temperature=0)

# Initialize the agent with the toolkit
agent = initialize_agent(
    arithmetic_tools,  # Pass the toolkit
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# The agent will automatically select appropriate tools based on queries
# Examples:
# agent.invoke("What is 5 multiplied by 10?")
# agent.invoke("Divide 100 by 4 and then raise the result to the power of 2.")
```

**How It Works:**
- The agent receives access to all tools in the toolkit
- Based on the user's query, the LLM-powered agent decides which tool(s) to use
- Tools are logically organized and easily discoverable
- Simplifies passing multiple related tools to agents

**Availability:**
- LangChain offers built-in toolkits for common integrations
- Users can create custom toolkits by grouping related tools

---

## Summary

Tools are the bridge between LLM reasoning and real-world action. They transform LLMs from systems that can only "think" into agents that can "act." Understanding how to create and use tools is fundamental to building effective AI agents with LangChain.

**Key Takeaways:**
- Tools provide LLMs with capabilities to perform specific tasks
- Built-in tools cover common use cases with minimal setup
- Custom tools enable integration with unique systems and business logic
- Three methods for creating custom tools, each with different levels of customization
- Toolkits group related tools for easier agent management
- Tools + LLMs = AI Agents
