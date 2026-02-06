# Models in LangChain

## 1. Introduction to Models
- The **Model component** in LangChain facilitates interactions with various Language Models and Embedding Models.
- It provides a **common interface** to connect with different AI models, regardless of company-specific behaviors.

## 2. Types of Models
### Language Models
- Input: Text (e.g., *"What is the capital of Pakistan?"*).
- Output: Text (e.g., *"Islamabad"*).
- Applications: Building chatbots and other text-based applications.

### Embedding Models
- Input: Text.
- Output: A series of numbers (embeddings/vectors) representing the text’s context.
- Applications: Semantic search and Retrieval-Augmented Generation (RAG) based applications.

## 3. Open vs Closed Models
### Part 1: Language Models
- **Closed Source Models (paid):**
  - OpenAI’s GPT Models
  - Anthropic’s Claude Model
  - Google’s Gemini Model
- **Open Source Models (free):**
  - Hugging Face models
- Task: Build a small Text Generation Application.

### Part 2: Embedding Models
- **Closed Source Embedding Models:**
  - OpenAI’s Embedding Model
- **Open Source Embedding Models:**
  - Hugging Face Embedding Models
- Task: Build a small Document Similarity Application.

## 4. Language Models: LLMs vs. Chat Models
### Large Language Models (LLMs)
- General-purpose models for text generation, summarization, code generation, and question answering.
- Input/Output: Plain text string → Plain text string.
- Limitations:
  - No memory (do not retain conversation history).
  - No role awareness (cannot be assigned specific roles).
- Examples: `text-davinci-003`, `GPT-3.5-turbo-instruct`.
- Status: Gradually being deprecated in LangChain in favor of Chat Models.

### Chat Models
- Specialized for conversational tasks, ideal for building chatbots.
- Input/Output: Sequence of messages → Chat messages.
- Advantages:
  - Fine-tuned on chat datasets.
  - Support for conversation history.
  - Role awareness (e.g., system-level roles like *"You are a knowledgeable doctor"*).
- Recommended: LangChain strongly advises using Chat Models for new projects.
- Examples: `GPT-3.5-turbo`, `GPT-4`, `Claude`, `Gemini`.
