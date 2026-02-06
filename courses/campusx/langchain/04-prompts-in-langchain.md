# Prompts in LangChain

## 1. Introduction
- Prompts are the **second essential component** in LangChain, following Models.
- Goal: Provide a clear explanation of prompts, their necessity, and techniques for using them.

## 2. Definition
- A **prompt** is the message or input sent to an LLM during interaction.
- Prompts are not new; they were implicitly used in earlier demonstrations.

## 3. Types of Prompts
### Text-based Prompts
- Account for 99% of current LLM interactions.

### Multimodal Prompts
- Involve other modes such as images, sound, or video.
- Example: Uploading an image and asking questions about it.
- Expected to become more prominent in the future.

## 4. Importance of Prompts
- LLM output is highly dependent on prompts.
- Even slight changes in a prompt can significantly alter the response.
- Emergence of **Prompt Engineering** as a job profile dedicated to designing effective prompts.

## 5. Static vs. Dynamic Prompts
### Static Prompts
- Hardcoded directly into the application.
- Problems:
  - Too much user control.
  - Inconsistent or undesirable outputs.
  - User errors (typos, misaligned phrasing).
  - Risk of hallucinations or poor results, especially when user phrasing doesn’t align with intended style (e.g., analogies, mathematical details).

### Dynamic Prompts
- Solution to static prompt issues.
- Use pre-defined templates with placeholders.
- Users provide specific inputs (e.g., paper name, style, length).
- Ensures consistency while allowing customization.
- Example: Research Assistant Tool with dropdowns for paper selection, explanation style, and length.

## 6. PromptTemplate in LangChain
### Implementation
- Import `PromptTemplate` from `langchain_core.prompts`.
- Define template string with placeholders (e.g., `{paper_input}`, `{style_input}`, `{length_input}`).
- Instantiate `PromptTemplate` with template string and input variables.
- Use `template.invoke()` to fill placeholders with user values.

### Advantages over f-strings
- **Validation:**
  - Built-in validation (`validate_template=True`).
  - Ensures all declared variables are present and no extras exist.
  - Prevents runtime errors and helps during development.
- **Reusability:**
  - Templates can be saved to JSON files using `save()`.
  - They can be loaded and reused across different files or applications using `load_prompt()`.
  - Promotes modularity and reduces code bulk.
- **Integration:**
  - Seamlessly connects with LangChain components like Chains.
  - Example: `chain = template | model`.
  - Allows a single `chain.invoke()` call for both prompt formatting and LLM invocation.

## 7. ChatPromptTemplate in LangChain
- LangChain also provides **ChatPromptTemplate** for building prompts specifically tailored to chat models.
- Instead of a single text string, ChatPromptTemplate works with structured message types.
- Implementation:
  - Import `ChatPromptTemplate` from `langchain_core.prompts`.
  - Define a template with roles and placeholders (e.g., system, human).
  - Example:
    ```python
    from langchain_core.prompts import ChatPromptTemplate

    template = ChatPromptTemplate.from_messages([
        ("system", "You are a helpful assistant."),
        ("human", "Explain {topic} in {style} style.")
    ])
    ```
  - Use `template.invoke()` to fill placeholders with user-provided values.
- Benefits:
  - Provides role awareness (system, human, AI).
  - Ensures structured prompts for conversational tasks.
  - Integrates seamlessly with chat models like GPT-3.5, GPT-4, Claude, Gemini.

## 8. Messages: Managing Chat History and Context
### Problem with Basic Chatbots
- Lack context and memory.
- Cannot handle follow-up questions that depend on prior conversation.

### Solution: Chat History
- Maintain a list of all previous messages.
- Send entire history with each new prompt.
- Provides context for relevant responses.

### Problem with Simple Chat History
- A list of strings does not differentiate sender roles (User vs. AI).
- Can confuse the LLM as conversation grows.

### LangChain’s Solution: Message Types
- **SystemMessage:** Defines AI’s role/persona (e.g., "You are a helpful assistant").
- **HumanMessage:** Represents user input.
- **AIMessage:** Represents AI responses.

## 9. Building a Chatbot with Message Types
### Implementation Steps
1. Import `SystemMessage`, `HumanMessage`, and `AIMessage` from `langchain_core.messages`.
2. Initialize `chat_history` as an empty list.
3. Append a `SystemMessage` at the start to define AI’s role.
4. For user input, create a `HumanMessage` and append to `chat_history`.
5. Pass `chat_history` to `model.invoke()`.  
   - Important: `model.invoke()` is flexible and accepts a list of messages directly.
6. When AI responds, create an `AIMessage` and append to `chat_history`.

### Benefits
- Structured approach ensures clear context.
- Improves conversational coherence by distinguishing roles.
- Prevents errors in long conversations by maintaining role awareness.
