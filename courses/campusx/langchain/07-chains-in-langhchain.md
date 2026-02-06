# Chains in LangChain

## Overview

Chains are the fundamental concept in LangChain (the framework is literally named after them). They enable the creation of pipelines that connect multiple steps in LLM-based applications, automating the flow of data from one component to the next.

## What are Chains?

**Definition**: Chains create pipelines of smaller, interconnected steps in an LLM application.

**Core Principle**: The output of one step automatically becomes the input for the next step. You only provide input to the first step, and the entire pipeline executes automatically to produce the final output.

## Why Use Chains?

### The Problem
- LLM applications involve multiple discrete steps (getting user input, sending to LLM, processing response, displaying output)
- Manually executing each step becomes laborious and complex in larger applications

### The Solution
- Chains connect these steps into automated pipelines
- Significantly simplifies development of complex applications
- Enables sophisticated application architectures

## Chain Structures

Chains support various execution patterns:

1. **Sequential Pipelines**: Linear execution of steps
2. **Parallel Processing**: Execute multiple chains simultaneously
3. **Conditional Chains**: Execute different chains based on conditions

## Types of Chains

### 1. Simple Chain

**Purpose**: Basic linear connection of components

**Implementation**:
```python
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.schema.output_parser import StringOutputParser
from dotenv import load_dotenv

load_dotenv()

# Define components
prompt = PromptTemplate(
    template="Generate five interesting facts about {topic}"
)
model = ChatOpenAI()
parser = StringOutputParser()

# Form chain using LCEL pipe operator
chain = prompt | model | parser

# Invoke chain
result = chain.invoke({'topic': 'Cricket'})
print(result)

# Visualize chain structure
chain.get_graph().print_ascii()
```

**Flow**: PromptInput → PromptTemplate → ChatOpenAI → StringOutputParser → Output

### 2. Sequential Chain

**Purpose**: Multiple LLM calls in sequence, where each step processes the output of the previous step

**Use Case**: Generate a detailed report, then summarize it into key points

**Implementation**:
```python
# Define two prompt templates
prompt1 = PromptTemplate(
    template="Generate a detailed report on {topic}"
)

prompt2 = PromptTemplate(
    template="Generate a five-pointer summary from the following text: {text}"
)

model = ChatOpenAI()
parser = StringOutputParser()

# Form sequential chain
chain = prompt1 | model | parser | prompt2 | model | parser

# Invoke chain
result = chain.invoke({'topic': 'Unemployment in India'})
print(result)
```

**Key Feature**: The output of the first LLM call (detailed report) automatically becomes the `text` input for the second prompt template.

### 3. Parallel Chain

**Purpose**: Execute multiple tasks concurrently, then merge results

**Use Case**: Generate both notes and a quiz from a text document, then combine them

**Architecture**: Input text → (Parallel: Generate Notes + Generate Quiz) → Merge into single document → Output

**Key Concept: RunnableParallel**
- Allows execution of multiple chains simultaneously
- Takes a dictionary where keys are output names and values are the chains
- All chains execute in parallel and results are combined

**Implementation**:
```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain.prompts import PromptTemplate
from langchain.schema.output_parser import StringOutputParser
from langchain.schema.runnable import RunnableParallel

# Initialize models
model1 = ChatOpenAI()
model2 = ChatAnthropic(model_name='claude-3-opus-20240229')
parser = StringOutputParser()

# Define prompt templates
prompt1_notes = PromptTemplate(
    template="Generate short and simple notes from the following text: {text}"
)

prompt2_quiz = PromptTemplate(
    template="Generate five short question answers from the following text: {text}"
)

prompt3_merge = PromptTemplate(
    template="Merge the provided notes and quiz into a single document. Notes: {notes}, Quiz: {quiz}"
)

# Create parallel chain
parallel_chain = RunnableParallel({
    'notes': prompt1_notes | model1 | parser,
    'quiz': prompt2_quiz | model2 | parser
})

# Create merge chain
merge_chain = prompt3_merge | model1 | parser

# Form final chain
chain = parallel_chain | merge_chain

# Invoke with sample text
text = "Sample text about Support Vector Machines..."
result = chain.invoke({'text': text})
print(result)
```

**Flow**: The parallel chains execute simultaneously, producing `notes` and `quiz` outputs, which are then passed to the merge chain.

### 4. Conditional Chain

**Purpose**: Execute different chains based on runtime conditions (like if-else logic)

**Use Case**: Classify user feedback sentiment, then provide appropriate response based on whether it's positive or negative

**Architecture**: User Feedback → Classify Sentiment → (If Positive: Send Positive Reply | If Negative: Send Negative Reply) → Output

**Key Concept: RunnableBranch**
- Enables conditional execution similar to if-else statements
- Takes a list of tuples: `(condition, chain_to_execute_if_true)`
- Includes a default chain if no conditions are met

**Ensuring Consistent Output with Pydantic**

Problem: LLM outputs might vary (e.g., "The sentiment is Negative" instead of just "Negative")

Solution: Use `PydanticOutputParser` with a structured model to ensure consistent output format

**Implementation**:
```python
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.schema.output_parser import StringOutputParser
from langchain.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field
from typing import Literal
from langchain.schema.runnable import RunnableBranch, RunnableLambda

# Define Pydantic model for structured sentiment output
class Feedback(BaseModel):
    sentiment: Literal['Positive', 'Negative'] = Field(
        description="The sentiment classification"
    )

# Initialize components
model = ChatOpenAI()
parser = StringOutputParser()
parser2 = PydanticOutputParser(pydantic_object=Feedback)

# Classification prompt with format instructions
prompt1 = PromptTemplate(
    template="Classify the sentiment of the following feedback text into Positive or Negative.\n{format_instructions}\nFeedback: {feedback}",
    partial_variables={"format_instructions": parser2.get_format_instructions()}
)

# Response prompts
prompt2 = PromptTemplate(
    template="Write an appropriate thankful reply for the following positive feedback: {feedback}"
)

prompt3 = PromptTemplate(
    template="Write an appropriate regretful reply for the following negative feedback: {feedback}"
)

# Create classification chain
classify_chain = prompt1 | model | parser2

# Create branch chain with conditions
branch_chain = RunnableBranch(
    (
        lambda x: x.sentiment == 'Positive',
        prompt2 | model | parser
    ),
    (
        lambda x: x.sentiment == 'Negative',
        prompt3 | model | parser
    ),
    RunnableLambda(lambda x: "Could not find sentiment")  # Default
)

# Form final chain
chain = classify_chain | branch_chain

# Invoke with different feedback
result = chain.invoke({'feedback': "This is a terrible phone"})
# or
result = chain.invoke({'feedback': "This is a wonderful phone"})
print(result)
```

**Key Points**:
- The `Feedback` Pydantic model with `Literal` type ensures output is always exactly "Positive" or "Negative"
- `PydanticOutputParser` structures the LLM's sentiment output for reliable branching
- Lambda functions check the sentiment and route to appropriate response chain
- `RunnableLambda` provides a default fallback if sentiment is unclear

## LangChain Expression Language (LCEL)

**Pipe Operator (`|`)**: Used to connect chain components. Data flows automatically from left to right through the pipeline.

## Visualizing Chains

Use `chain.get_graph().print_ascii()` to visualize the structure of any chain, showing the flow of data through components and any parallel or conditional branches.

## Key Takeaways

- Chains automate complex multi-step LLM workflows
- Sequential chains enable step-by-step processing
- Parallel chains improve efficiency by running tasks simultaneously
- Conditional chains enable dynamic, context-aware behavior
- Structured outputs (Pydantic) ensure reliable conditional logic
- LCEL provides an intuitive syntax for building chains