# How Transformer LLMs Work - Course Notes

## Course Overview
**Instructors:** Jay Alammar and Maarten Grootendorst (authors of "Hands on Large Language Models")

**Course Goal:** Understand the main components of the LLM transformer architecture and gain the ability to read and comprehend research papers describing model architectures.

---

## Introduction to Transformers

### Origins
- **Published:** 2017 paper "Attention is All You Need" by Ashish Vaswani et al.
- **Original Purpose:** Machine translation tasks (e.g., English → German)
- **Evolution:** Extended to prompt-response tasks (questions → answers), leading to the rise of large language models

### Original Architecture Components
The transformer consists of two main parts:

1. **Encoder**
   - Preprocesses the entire input text
   - Extracts context needed for the task
   - Example: Processing English sentence for translation

2. **Decoder**
   - Uses encoder context to generate output
   - Example: Producing German translation

---

## Modern Applications

### Encoder Models
- **Functionality:** Provides rich, context-sensitive representations of input text
- **Key Example:** BERT model
- **Primary Use:** Embedding models for RAG (Retrieval-Augmented Generation) applications

### Decoder Models
- **Functionality:** Text generation tasks
- **Applications:**
  - Summarizing text
  - Writing code
  - Answering questions
- **Examples:** Models from OpenAI, Anthropic, Cohere, and Meta

---

## Course Content Structure

### 1. Historical Development
- Trace the evolution of LLMs
- Understand the sequence of building blocks that led to modern transformers

### 2. Tokenization
- **Process:** Breaking text into tokens (words or word fragments)
- **Purpose:** Converting text into format that can be fed into the LLM

### 3. Transformer Network Architecture
**Focus:** Decoder-only models

#### Text Generation Process
Generative models work by generating one token at a time:

**Step 1: Token Embedding**
- Each input token is mapped to an embedding vector
- Embedding captures the semantic meaning of the token

**Step 2: Transformer Blocks**
- Token embeddings pass through a stack of transformer blocks
- Each block contains:
  - **Attention Layer:** Learns relationships between tokens
  - **Feed-Forward Network:** Processes information
- Architecture designed to:
  - Learn flexibly from data
  - Scale efficiently on GPUs

**Step 3: Language Modeling Head**
- Takes output vectors from transformer blocks
- Generates the output token

---

## Key Insights

### The "Magic" of LLMs
The effectiveness of LLMs comes from **two components**:

1. **Transformer Architecture**
   - The neural network structure itself
   - Efficient, scalable design

2. **Rich Training Data**
   - Vast amounts of high-quality data
   - Diverse linguistic patterns and knowledge

### Why Understanding Architecture Matters
- Provides intuition about model behavior
- Helps understand why LLMs behave in certain ways
- Improves ability to effectively use LLMs
- Enables reading and understanding research papers

---

## Course Philosophy
The course aims to make transformer architecture accessible and understandable, demystifying the technology behind modern language models while acknowledging that both architecture and data contribute to their capabilities.
