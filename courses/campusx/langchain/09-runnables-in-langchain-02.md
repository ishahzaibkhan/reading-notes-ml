# LangChain Runnables - Part 2: Primitives & LCEL

## Overview
This guide covers LangChain Runnable Primitives and introduces the LangChain Expression Language (LCEL) for building complex AI workflows.

## Background: The Runnable Architecture

### The Problem
Early LangChain components lacked standardization:
- **PromptTemplate**: Used `format()` method
- **LLMs**: Used `predict()` method  
- **Parsers**: Used `parse()` method
- **Retrievers**: Used `get_relevant_documents()` method

This inconsistency made it difficult to flexibly connect components and create complex workflows.

### The Solution
LangChain introduced the **Runnable** abstract class:
- All components now inherit from this base class
- Enforces implementation of standardized methods (notably `invoke()`)
- Enables seamless component connection where output of one becomes input of the next

## Runnable Categories

### 1. Task-Specific Runnables
Core LangChain components converted to Runnables, each with a specific purpose:
- `ChatOpenAI` - LLM interaction
- `PromptTemplate` - Prompt design
- `Retriever` - Document retrieval
- `StringOutputParser` - Output parsing

### 2. Runnable Primitives
Fundamental building blocks for orchestrating workflows. They define how runnables interact (sequentially, in parallel, conditionally).

Built-in primitives:
- `RunnableSequence`
- `RunnableParallel`
- `RunnableBranch`
- `RunnableLambda`
- `RunnablePassthrough`

---

## RunnableSequence

**Purpose**: Connects two or more runnables sequentially, where the output of each becomes the input of the next.

### Basic Example: Joke Generation
```python
from langchain.prompts import PromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StringOutputParser
from langchain.schema.runnable import RunnableSequence

prompt = PromptTemplate.from_template("Write a joke about {topic}")
model = ChatOpenAI()
parser = StringOutputParser()

chain = RunnableSequence(prompt, model, parser)
chain.invoke({"topic": "AI"})
```

### Extended Example: Joke with Explanation
```python
prompt1 = PromptTemplate.from_template("Write a joke about {topic}")
prompt2 = PromptTemplate.from_template("Explain the following joke:\n{text}")

chain = RunnableSequence(prompt1, model, parser, prompt2, model, parser)
chain.invoke({"topic": "AI"})
```

This chain generates a joke, then passes it to the explanation prompt, producing a final explanation of the joke.

---

## RunnableParallel

**Purpose**: Executes multiple runnables in parallel, each receiving the same input independently.

**Output Format**: Returns a dictionary where keys are the names you assign and values are the outputs from each parallel branch.

### Example: Social Media Content Generation
```python
from langchain.schema.runnable import RunnableParallel

prompt_tweet = PromptTemplate.from_template("Generate a tweet about {topic}")
prompt_linkedin = PromptTemplate.from_template("Generate a LinkedIn post about {topic}")

parallel_chain = RunnableParallel(
    tweet=RunnableSequence(prompt_tweet, model, parser),
    linkedin=RunnableSequence(prompt_linkedin, model, parser)
)

result = parallel_chain.invoke({"topic": "AI"})
# Output: {'tweet': '...', 'linkedin': '...'}
```

You can access individual outputs: `result['tweet']` and `result['linkedin']`

---

## RunnablePassthrough

**Purpose**: Returns its input as output without any modification or processing.

**Use Cases**: 
- Passing input through a chain while performing other operations in parallel
- Maintaining context or original data in the output
- Preserving intermediate results

### Basic Demonstration
```python
from langchain.schema.runnable import RunnablePassthrough

RunnablePassthrough().invoke("Hello")  # Returns: "Hello"
RunnablePassthrough().invoke({"name": "Nitish"})  # Returns: {"name": "Nitish"}
```

### Advanced Example: Joke Generation with Original
**Problem**: When chaining joke generation and explanation, only the explanation appears in output.

**Solution**: Use RunnablePassthrough to preserve the original joke.
```python
prompt2 = PromptTemplate.from_template("Explain the following joke:\n{text}")

joke_gen_chain = RunnableSequence(prompt, model, parser)

parallel_chain = RunnableParallel(
    joke=RunnablePassthrough(),
    explanation=RunnableSequence(prompt2, model, parser)
)

final_chain = RunnableSequence(joke_gen_chain, parallel_chain)
result = final_chain.invoke({"topic": "Cricket"})
# Output: {'joke': '...', 'explanation': '...'}
```

---

## RunnableLambda

**Purpose**: Converts any Python function into a Runnable, enabling custom logic integration into LangChain chains.

**Use Cases**:
- Custom pre-processing or post-processing
- Data transformation
- Any operation not covered by existing LangChain components

### Basic Demonstration
```python
from langchain.schema.runnable import RunnableLambda

# Using a defined function
def word_counter(text):
    return len(text.split())

runnable_word_counter = RunnableLambda(word_counter)
runnable_word_counter.invoke("This is a test string")  # Returns: 5

# Using a lambda function directly
RunnableLambda(lambda x: len(x.split())).invoke("This is a test string")
```

### Advanced Example: Joke with Word Count
```python
prompt = PromptTemplate.from_template("Write a joke about {topic}")
word_count_func = lambda text: len(text.split())

joke_gen_chain = RunnableSequence(prompt, model, parser)

parallel_chain = RunnableParallel(
    joke=RunnablePassthrough(),
    word_count=RunnableLambda(word_count_func)
)

final_chain = RunnableSequence(joke_gen_chain, parallel_chain)
result = final_chain.invoke({"topic": "AI"})
# Output: {'joke': '...', 'word_count': 42}
```

---

## RunnableBranch

**Purpose**: Creates conditional chains, acting as an if-else statement within LangChain workflows. Directs flow to different sub-chains based on conditions.

**Use Cases**:
- Handling different types of user input (complaint vs. refund vs. general query)
- Different data processing paths based on content characteristics
- Any scenario requiring conditional logic

**Structure**: 
```python
RunnableBranch(
    (condition_function, runnable_if_true),
    (condition_function, runnable_if_true),
    default_runnable  # optional else case
)
```

### Example: Conditional Report Summarization
**Scenario**: Generate a detailed report. If the report exceeds 500 words, summarize it; otherwise, return it as is.
```python
from langchain.schema.runnable import RunnableBranch

prompt_report = PromptTemplate.from_template("Write a detailed report on {topic}")
prompt_summarize = PromptTemplate.from_template("Summarize the following text:\n{text}")

report_gen_chain = RunnableSequence(prompt_report, model, parser)

# Implementation logic:
# 1. Generate report using report_gen_chain
# 2. Use RunnableLambda to count words
# 3. Use RunnableBranch to check if word_count > 500
#    - If true: apply summarization chain
#    - If false: use RunnablePassthrough to return original report
```

---

## LCEL (LangChain Expression Language)

**Definition**: LangChain's declarative way to compose runnable chains using the `|` operator (similar to Unix pipes).

LCEL provides a more concise syntax for creating the same chains covered in this guide, building upon these primitive concepts.

---

## Key Takeaways

1. **Standardization**: All LangChain components now inherit from the Runnable base class, enabling consistent interaction
2. **Composability**: Primitives allow flexible combination of components for complex workflows
3. **Execution Patterns**: 
   - Sequential: `RunnableSequence`
   - Parallel: `RunnableParallel`
   - Conditional: `RunnableBranch`
4. **Custom Integration**: `RunnableLambda` enables any Python function to join the workflow
5. **Context Preservation**: `RunnablePassthrough` maintains data throughout the chain
6. **LCEL**: Provides syntactic sugar for creating these chains more elegantly