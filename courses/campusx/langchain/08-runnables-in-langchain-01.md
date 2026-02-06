# LangChain Runnables: Complete Guide

## Introduction

Runnables are a highly technical but crucial concept for understanding LangChain as a framework. They form the foundation upon which chains operate internally, making them essential for effective LangChain usage.

---

## Understanding Chains (Recap)

### What Are Chains?

Chains are fundamental components in LangChain that connect multiple components into a pipeline. They work behind the scenes using runnables.

### Three Types of Chains

1. **Sequential Chain** - Executes components in a specific order
2. **Parallel Chain** - Executes components simultaneously
3. **Conditional Chain** - Executes components based on conditions

---

## Historical Context: Why Runnables Exist

### The Beginning (November 2022)

OpenAI released ChatGPT and opened APIs to the public, enabling developers to build LLM-based applications. The LangChain team recognized the future demand for such applications.

### Current Scenario

Many companies now have:
- Chatbots (LLM-based applications)
- PDF readers (allowing interaction and questions)
- AI agents that utilize LLMs

### LangChain's Original Goal

Create a framework to simplify the development of LLM-based applications.

---

## Problems LangChain Solved

### Problem 1: Diverse LLM APIs

**Challenge:** Multiple companies (Anthropic, Google, Mistral, etc.) built LLMs, each with different API behaviors.

**Solution:** Build a framework allowing interaction with any LLM API with minimal code changes. This was LangChain's initial breakthrough.

### Problem 2: LLM Interaction Is Just One Component

**Challenge:** LangChain realized that interacting with an LLM is only a small part of building a complete LLM-based application.

#### Example: PDF Reader Application Workflow

1. Load PDF document
2. Split into smaller parts (chapters, pages, paragraphs)
3. Generate embeddings for each part
4. Store embeddings in a vector database
5. Use a retriever for semantic search to find relevant chunks
6. Send the most relevant chunk to the LLM
7. Receive LLM response, parse it, and display to the user

**Key Insight:** LLM interaction is just one component among many.

### Solution 2: Componentization

LangChain created separate components for every important part of the development process:

- **Document Loaders** - For loading documents
- **Text Splitters** - For splitting data
- **Embedding Models** - For generating embeddings
- **Vector Databases and Retrievers** - For storage and semantic search
- **Parsers** - For parsing output

This made LangChain a powerful library offering comprehensive support for building LLM applications.

---

## The Evolution to Chains

### The Eureka Moment

LangChain noticed that AI engineers commonly use similar patterns when connecting components. These patterns could be automated and reused.

### Common Pattern Identified

Creating a prompt and sending it to an LLM is ubiquitous across chatbots, PDF readers, and AI agents.

### The Automation Idea

Instead of manually:
1. Creating a PromptTemplate
2. Creating an LLM
3. Calling `format()` to generate the prompt
4. Sending to `LLM.predict()` to get the result

Create a built-in function where developers provide the LLM and PromptTemplate, and the function handles everything automatically.

### Birth of Chains

This concept of connecting multiple components with built-in functions was named "Chains" because it joins two or more components into a pipeline.

### LLM Chain

The simplest chain designed to:
- Take an LLM and a prompt
- Generate the prompt
- Send it to the LLM
- Return the result

This eliminated manual steps and significantly simplified code.

---

## More Complex Chains

### RAG Implementation with Chains

**Retrieval Process:**
- User provides a query
- Query is searched in vector database to find relevant chunks
- Example: Searching for "linear regression assumptions" in a machine learning book

**Prompt Construction:**
- Retrieved relevant text and original query create a new prompt
- LLM is asked to answer the query based on provided text
- LLM provides response

### Retrieval QA Chain

This built-in chain automates the entire RAG process:
- Loading
- Splitting
- Embedding
- Retrieving
- Prompt construction
- LLM interaction

**Impact:** Reduced complex PDF reader applications from 36 lines to 32 lines of code.

---

## The Proliferation of Chains

### Top Chain Types (Examples)

1. **Simple Sequential Chain** - Combines two or more LLM Chains
   - Example: Generate a joke, then generate an explanation for that joke

2. **Sequential Chain** - Multiple LLM Chains with multiple inputs and outputs

3. **SQL Database Chains** - For interacting with SQL databases

4. **API Chain** - For interacting with APIs

5. **LLM Math Chain** - For math-based problems

### The Vision

By joining components into chains, AI engineers could easily build LLM-based applications.

---

## The Big Problem: Too Many Chains

### What Went Wrong

LangChain created an overwhelming number of chains trying to solve every use case.

### Disadvantages

1. **Large Codebase**
   - Codebase became very large
   - Difficult to actively maintain

2. **Steep Learning Curve**
   - New AI engineers struggled to learn LangChain
   - Couldn't understand which chain to use for which use case

**Result:** This contradicted LangChain's original goal of making development easier.

---

## Root Cause Analysis

### LangChain's Vision vs. Reality

**Intent:** Enable AI engineers to seamlessly plug and connect various components (LLM, Prompt, Parsers, Retrievers) like LEGO blocks to create flexible workflows.

**Reality:** To achieve this, they relied on creating numerous chains, leading to a heavy codebase and steep learning curve.

### The Core Issue: Non-Standardized Components

The fundamental components were not standardized:
- Developed independently
- Behaved differently
- Used different methods:
  - LLM component uses `predict()`
  - PromptTemplate uses `format()`
  - Retriever uses `get_relevant_documents()`
  - Parsers use `parse()`

**Consequence:** Since components weren't designed to connect seamlessly, LangChain had to write manual functions (chains) to bridge incompatibilities.

**Problem Escalation:** Every new use case required a new custom chain function.

### The Ideal Scenario

If components followed the same standards, they could connect seamlessly without custom code, eliminating the need for numerous custom functions.

**Conclusion:** This lack of standardization was LangChain's biggest mistake.

---

## What Are Runnables?

### The Solution

LangChain realized they needed to rebuild components to be standardized and seamlessly connectable. This is where Runnables come in.

### Definition of Runnables

#### 1. Unit of Work
- A runnable is a "unit of work"
- Each runnable has a specific purpose
- Takes an input, processes it, and provides an output

#### 2. Common Interface
Every runnable follows a common interface with the same set of methods:

- **`invoke()`** - Provides an input and gets an output (most popular method)
- **`batch()`** - Processes multiple inputs simultaneously and returns multiple outputs
- **`stream()`** - Enables streaming output from the runnable

#### 3. Connectable
- Since all runnables follow a common interface, they can be connected to each other
- Connection means: R1's output automatically becomes R2's input
- This allows building arbitrarily complex workflows

#### 4. Workflows Are Also Runnables
- When runnables are connected to form a workflow, that workflow itself is also a runnable
- Example: If R1+R2+R3 form one workflow, and R4+R5 form another, both resulting workflows are runnables
- These larger workflows can also be connected, enabling recursive complexity

---

## The LEGO Block Analogy

Runnables work like LEGO blocks:

### 1. Unit of Work
Each LEGO block has its own purpose (single, wedge, U-shape, L-shape)

### 2. Common Interface
Despite different structures, all LEGO blocks have the same connecting studs

### 3. Connectable
LEGO blocks can be connected to each other

### 4. Resultant Structures Are Blocks
When two LEGO blocks are connected, the resulting structure is itself a LEGO block (it still has connecting studs), allowing further connections to build more complex structures

**Key Takeaway:** Runnables are essentially the LEGO blocks of the LangChain universe.

---

## Core Concepts Summary

### The Journey

1. **Initial Problem** - Multiple LLM APIs with different behaviors
2. **First Solution** - Unified interface for LLM interaction
3. **Second Problem** - LLM interaction is just one small component
4. **Second Solution** - Create comprehensive components for all aspects
5. **Third Problem** - Common patterns needed automation
6. **Third Solution** - Create chains to automate patterns
7. **Fourth Problem** - Too many chains, non-standardized components
8. **Final Solution** - Runnables with common interface

### Why Runnables Matter

- **Standardization** - All components follow the same interface
- **Flexibility** - Components can be connected like LEGO blocks
- **Simplicity** - No need for custom chains for every use case
- **Scalability** - Workflows are runnables, enabling recursive complexity
- **Maintainability** - Smaller codebase, easier to maintain

### The Power of Runnables

Runnables solved LangChain's fundamental problem by providing:
- A common interface for all components
- Seamless connectivity between components
- The ability to build complex workflows without custom code
- True plug-and-play functionality

This represents LangChain's evolution from a collection of disconnected tools to a truly unified framework.