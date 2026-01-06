# Output Parsers in LangChain - Complete Notes

## Overview
Output parsers are LangChain classes that convert raw LLM textual responses into structured formats (JSON, CSV, Pydantic models), ensuring consistency, validation, and ease of use in applications.

---

## Why Output Parsers?

### The Problem
- LLMs return unstructured text by default
- Unstructured text cannot be directly sent to databases or APIs
- Need structured output with schema for system integration

### Two Types of LLMs

**1. Native Structured Output Support**
- Some LLMs (e.g., GPT models) fine-tuned for structured output
- Use LangChain's `with_structured_output()` function

**2. No Native Structured Output**
- Many open-source models lack this capability
- **Output Parsers** bridge this gap
- Can also be used with native models for seamless handling

---

## Four Main Output Parsers

### 1. String Output Parser

**Purpose**: Converts LLM response to plain string

**Why Use It?**
- LLM responses are `ChatGeneration` objects with content + metadata
- Normally access via `result.content`
- String parser simplifies by directly returning string

**Key Use Case: LangChain Chains**
- Essential for multi-step LLM interactions
- Cleanly passes output from one step as input to next

**Without Parser** (Manual Approach):
```python
result1 = model.invoke(prompt1)
content1 = result1.content  # Manual extraction
result2 = model.invoke(prompt2_with_content1)
content2 = result2.content  # Manual extraction again
```

**With Parser** (Chain Approach):
```python
from langchain_core.output_parsers import StringOutputParser

parser = StringOutputParser()
chain = template_one | model | parser | template_two | model | parser
result = chain.invoke(input_data)
```

**Benefits**: Cleaner code, automatic content extraction, seamless multi-step processing

---

### 2. JSON Output Parser

**Purpose**: Forces LLM to return output in JSON format (quickest way to get JSON)

**Usage**:
```python
from langchain_core.output_parsers import JsonOutputParser

parser = JsonOutputParser()
template = PromptTemplate(
    ...,
    partial_variables={'format_instructions': parser.get_format_instructions()}
)
```

**How It Works**:
- `get_format_instructions()` generates string telling LLM expected JSON structure
- Use `partial_variables` to inject format instructions into prompt

**Manual Parsing**:
```python
result = model.invoke(prompt)
parsed = parser.parse(result.content)  # Converts JSON string to Python dict
```

**Chain Approach** (Preferred):
```python
chain = template | model | parser
result = chain.invoke({})  # Pass empty dict if no input variables
```

**MAJOR LIMITATION**: 
- **No schema enforcement** - LLM decides JSON structure
- Cannot control whether output is list vs key-value pairs
- Example: Facts might be `["fact1", "fact2"]` or `{"fact_1": "...", "fact_2": "..."}`

---

### 3. Structured Output Parser

**Purpose**: Extracts structured JSON based on **pre-defined field schemas**

**Advantage Over JSON Parser**: Schema enforcement - you control the structure

**Usage**:
```python
from langchain.output_parsers import StructuredOutputParser, ResponseSchema

# Define schema
schema = [
    ResponseSchema(name="fact_1", description="Fact one about the topic"),
    ResponseSchema(name="fact_2", description="Fact two about the topic")
]

# Create parser
parser = StructuredOutputParser.from_response_schemas(schema)

# Use in template
template = PromptTemplate(
    ...,
    partial_variables={'format_instructions': parser.get_format_instructions()}
)
```

**Manual Parsing**:
```python
parsed = parser.parse(result.content)
```

**Chain Approach** (Preferred):
```python
chain = template | model | parser
result = chain.invoke(input_data)
```

**LIMITATION**: 
- **No data validation**
- If field defined as integer but LLM returns "35 years" (string), parser accepts it
- No type checking or constraint enforcement

**Note**: Located in `langchain.output_parsers` (not `langchain_core`)

---

### 4. Pydantic Output Parser

**Purpose**: Uses Pydantic models for **schema validation AND structured output**

**Advantages Over Structured Parser**:
1. **Strict schema enforcement** - Define specific data types
2. **Data validation** - Apply constraints (e.g., age > 18)
3. **Type safety** - Performs type coercion when appropriate
4. **Error handling** - Raises validation errors for constraint violations
5. Seamless integration with other components

**Usage**:
```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

# Define Pydantic model
class Person(BaseModel):
    name: str = Field(description="Name of the person")
    age: int = Field(gt=18, description="Age must be greater than 18")
    email: str = Field(description="Email address")

# Create parser
parser = PydanticOutputParser(pydantic_object=Person)

# Use in template
template = PromptTemplate(
    ...,
    partial_variables={'format_instructions': parser.get_format_instructions()}
)
```

**How It Works**:
- `get_format_instructions()` generates detailed prompt with JSON schema from Pydantic model
- Includes field types, descriptions, and validation rules

**Manual Parsing**:
```python
parsed = parser.parse(result.content)
```

**Chain Approach** (Preferred):
```python
chain = template | model | parser
result = chain.invoke(input_data)
```

---

## Parser Comparison & When to Use

| Parser | Schema Control | Validation | Use Case |
|--------|---------------|------------|----------|
| **String** | None | None | Chaining operations, plain text output |
| **JSON** | None | None | Quick JSON output, no schema requirements |
| **Structured** | Yes | No | Specific JSON structure needed |
| **Pydantic** | Yes | Yes | Strict schema + data validation (production) |

### Decision Guide

**Use String Output Parser when**:
- Need plain string output
- Building multi-step chains

**Use JSON Output Parser when**:
- Need JSON format
- Don't care about specific structure
- Fastest JSON implementation

**Use Structured Output Parser when**:
- Need specific JSON schema/structure
- No validation requirements

**Use Pydantic Output Parser when**:
- Need strict schema enforcement
- Require data validation (types, constraints)
- Building production applications
- Need type safety

---

## Common Pattern: partial_variables

All parsers (except String) use this pattern:
```python
template = PromptTemplate(
    template="...",
    input_variables=[...],
    partial_variables={'format_instructions': parser.get_format_instructions()}
)
```

**Purpose**: Dynamically inject formatting instructions into prompt to tell LLM expected output structure

---

## Chain Syntax (Recommended)

**General Pattern**:
```python
chain = prompt_template | model | parser
result = chain.invoke(input_data)
```

**Benefits**:
- Cleaner code
- Automatic parsing
- Easier multi-step workflows
- No manual content extraction needed

**Note**: When no input variables needed, pass empty dict: `chain.invoke({})`