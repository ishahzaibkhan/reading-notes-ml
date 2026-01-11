# LangChain Components 

## Core Components of LangChain
LangChain consists of six essential components:

1. **Models**  
2. **Prompts**  
3. **Chains**  
4. **Indexes**  
5. **Memory**  
6. **Agents**

---

### 1. Models
- **Definition:** Interface to interact with AI models.  
- **Problem Solved:** LLMs are expensive to run locally; APIs vary across providers.  
- **LangChain’s Role:** Standardizes interaction, enabling easy switching between providers.  
- **Types of Models:**  
  - **Language Models (LLMs):** Input → text, Output → text (e.g., chatbots, agents).  
  - **Embedding Models:** Input → text, Output → vector (used for semantic search).  
- **Examples:** OpenAI, Anthropic, Mistral AI, Azure, Vertex AI, Bedrock, Hugging Face, IBM, Llama.

---

### 2. Prompts
- **Definition:** Inputs provided to an LLM.  
- **Importance:** Output quality depends heavily on prompt design → **Prompt Engineering**.  
- **LangChain’s Role:** Provides flexibility for reusable and dynamic prompts.  
- **Types of Prompts:**  
  - **Dynamic Templates:** Placeholders filled based on user input.  
  - **Role-Based Prompts:** Define system roles (e.g., “You are an experienced teacher”).  
  - **Few-Shot Prompts:** Provide examples to guide the LLM’s responses.

---

### 3. Chains
- **Definition:** Pipelines that automate passing outputs between components.  
- **Functionality:** Eliminates manual coding for multi-step workflows.  
- **Example:** English text → Hindi translation → summarization.  
- **Types of Chains:**  
  - **Sequential Chains:** Steps executed one after another.  
  - **Parallel Chains:** Multiple LLMs process input simultaneously, outputs combined.  
  - **Conditional Chains:** Flow depends on conditions (e.g., send email only if feedback is negative).

---

### 4. Indexes
- **Definition:** Connect applications to external knowledge sources (PDFs, websites, databases).  
- **Problem Solved:** LLMs lack access to private/proprietary data.  
- **Components (RAG System):**  
  - **Document Loader:** Imports external data.  
  - **Text Splitter:** Breaks large documents into chunks.  
  - **Vector Store:** Stores embeddings for efficient semantic search.  
  - **Retriever:** Matches user queries with relevant chunks and sends them to the LLM.

---

### 5. Memory
- **Problem Solved:** LLM API calls are stateless, making chatbots forgetful.  
- **Solution:** LangChain adds memory to conversations.  
- **Types of Memory:**  
  - **Conversation Buffer Memory:** Stores entire chat history.  
  - **Buffer Window Memory:** Stores only the last *N* interactions.  
  - **Summarizer Memory:** Summarizes chat history to reduce cost.  
  - **Custom Memory:** Stores specialized info (e.g., user preferences).

---

### 6. Agents
- **Definition:** Components that enable building AI Agents.  
- **Chatbot vs. Agent:**  
  - **Chatbot:** Understands language and generates text.  
  - **AI Agent:** Chatbot with “superpowers” → can perform real-world actions.  
- **Capabilities:**  
  - **Reasoning:** Decides which actions to take.  
  - **Tool Access:** Interacts with external APIs (e.g., calculator, flight booking).  
- **Example:** Travel agent bot that answers queries, finds cheapest flights, and books tickets.

---

## Key Takeaway
LangChain’s six components — **Models, Prompts, Chains, Indexes, Memory, Agents** — form the backbone of building powerful, flexible, and context-aware LLM applications.