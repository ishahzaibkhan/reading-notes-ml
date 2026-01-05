# Structured Output in LangChain

## Overview
Structured output enables LLMs to return responses in well-defined data formats (like JSON) instead of unstructured text, making outputs machine-readable and enabling integration with databases, APIs, and tools.

---

## Why Structured Output?

### 1. Data Extraction
- Store LLM output in databases
- **Example**: Job portal extracting name, company, education from resumes into JSON for database insertion

### 2. Building APIs
- Process and return structured data via services
- **Example**: Analyzing Amazon reviews to extract topic, pros, cons, sentiment in structured format for API consumption

### 3. Building Agents
- Agents are "chatbots on steroids" that perform tasks using tools
- Tools require structured input (not raw text)
- **Example**: "Find square root of 2" → extract operation and number into structured format for calculator tool

---

## Methods to Get Structured Output in LangChain

### 1. `with_structured_output()` Function
- For LLMs with native structured output capability (e.g., OpenAI GPT)
- Specify desired data format directly
- Three ways to define format: TypedDict, Pydantic, JSON Schema

### 2. Output Parsers
- For LLMs without native structured output
- Convert unstructured text to structured format

---

## TypedDict Approach

### Definition
- Python typing module feature to define dictionary structure
- Specifies required keys and value types
- **Limitation**: No runtime validation (type hints only)

### Basic Example
```python
from typing import TypedDict

class Person(TypedDict):
    name: str
    age: int

new_person: Person = {'name': 'Shahzaib', 'age': 28}
```

### LangChain Integration
```python
from langchain_openai import ChatOpenAI
from typing import TypedDict

class Review(TypedDict):
    summary: str
    sentiment: str

model = ChatOpenAI()
chain = model.with_structured_output(Review)
result = chain.invoke(review_text)
# Returns: {'summary': '...', 'sentiment': 'positive'}
```

### Advanced TypedDict Features

**Annotations** - Add descriptions for clarity:
```python
from typing import Annotated, List, Literal, Optional

class Review(TypedDict):
    summary: Annotated[str, "A brief summary of the review"]
    sentiment: Annotated[Literal["POS", "NEG", "NEU"], "Overall tone"]
    key_themes: Annotated[List[str], "Key themes in list format"]
    pros: Annotated[Optional[List[str]], "List of pros"]
    cons: Annotated[Optional[List[str]], "List of cons"]
    reviewer_name: Annotated[Optional[str], "Name of reviewer"]
```

**Key Types**:
- `List[str]`: Multiple string values
- `Optional[...]`: Field may not be present (returns None if missing)
- `Literal["A", "B", "C"]`: Restrict to predefined values

---

## Pydantic Approach

### Advantages Over TypedDict
- **Runtime validation**: Throws errors if data doesn't match schema
- **Type coercion**: Automatically converts compatible types
- **Built-in validators**: Email, URL, etc.
- Production-ready (used in FastAPI)

### Basic Example
```python
from pydantic import BaseModel

class Student(BaseModel):
    name: str

# Valid
student = Student(**{'name': 'Shahzaib'})

# Invalid - throws ValidationError
student = Student(**{'name': 28})  # Error: Input should be valid string
```

### Advanced Pydantic Features

**1. Default Values**
```python
class Student(BaseModel):
    name: str = "Shahzaib"
```

**2. Optional Fields**
```python
from typing import Optional

class Student(BaseModel):
    name: str
    age: Optional[int] = None
```

**3. Type Coercion**
```python
class Student(BaseModel):
    age: int

# '32' (string) automatically converted to 32 (int)
student = Student(name='Test', age='28')
```

**4. Built-in Validations**
```python
from pydantic import EmailStr

class Student(BaseModel):
    email: EmailStr  # Validates email format
```

**5. Field Function - Granular Control**
```python
from pydantic import Field

class Student(BaseModel):
    name: str
    cgpa: float = Field(
        gt=0, 
        lt=10, 
        description="CGPA between 0-10"
    )
```

**Field Parameters**:
- `gt`, `lt`: Greater than, less than (exclusive)
- `ge`, `le`: Greater/less than or equal (inclusive)
- `default`: Set default value
- `description`: Add metadata for LLMs
- Pattern matching via regex supported

---

## JSON Schema

### Definition & Use Case
- Language-agnostic standard for describing JSON data structure
- Used for cross-platform/cross-language schema sharing
- Ideal when integrating LLMs with services that require JSON Schema for data validation or API specifications

### Basic Format
```json
{
  "type": "object",
  "properties": {
    "summary": {
      "type": "string",
      "description": "A brief summary"
    },
    "sentiment": {
      "type": "string",
      "enum": ["POS", "NEG", "NEU"]
    },
    "score": {
      "type": "number",
      "minimum": 0,
      "maximum": 10
    }
  },
  "required": ["summary", "sentiment"]
}
```

---

## When to Use Which Schema Definition Method?

### TypedDict
- **Use for**: Simple cases with basic structural guidance only
- **Best when**: You only need type hinting without validation
- **Limitation**: No data validation - code runs even with type mismatches

### Pydantic
- **Use for**: Data validation and robust control
- **Best when**: Data integrity is crucial (APIs, databases)
- **Features**: Type coercion, defaults, optional fields, custom validation
- **Recommended**: Production applications

### JSON Schema
- **Use for**: Cross-language/cross-system schema sharing
- **Best when**: Integrating with services requiring JSON Schema
- **Benefit**: Language-agnostic standard for interoperability

---

## How LLMs Generate Structured Output

### 1. Function Calling (Preferred Method)
- **Most reliable** for modern LLMs trained for structured output
- LLM outputs JSON representing function call with arguments
- Inherently understands required data structure
- Used by `with_structured_output()` function
- More robust - produces valid JSON in specified format

### 2. JSON Mode / Direct JSON Generation
- Explicit instruction to output in JSON format (via prompt/model setting)
- LLM attempts to follow requested structure
- **Less reliable** - may generate malformed JSON or deviate from schema
- Handled by Output Parsers (for models without native structured output support)

---

## How It Works Behind the Scenes
When using `with_structured_output()`, LangChain automatically:
1. Generates a system prompt instructing the LLM to extract structured insights in JSON
2. Defines expected fields and types
3. LLM returns JSON response
4. Python parses into dictionary/object


