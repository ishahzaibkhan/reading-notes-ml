# Building a Chatbot with LangGraph — Part 1: State, Memory & Persistence
### CampusX Agentic AI Series — Episode 9

---

## Project Roadmap

This series builds a single chatbot progressively from a basic conversational agent to a production-ready system. Planned features across the series:

| Feature | Description |
|---|---|
| Conversational Chat | Basic multi-turn dialogue |
| RAG | Document-based Q&A via retrieval |
| Tool Use | Agent actions via external integrations |
| Memory & Persistence | Conversation history via checkpoints |
| Human-in-the-Loop (HITL) | Manual oversight before graph proceeds |
| Fault Tolerance | Retry logic and error handling |
| UI / Deployment | Frontend + LangSmith observability |

---

## Part 1 — Basic Chatbot Architecture

### Design

A minimal sequential workflow with a **single node** wrapping an LLM. All conversational context is managed through the state.

### State Definition

The state holds a list of messages — LangChain `BaseMessage` objects covering all message types: `HumanMessage`, `AIMessage`, `SystemMessage`, `ToolMessage`.

```python
from typing import Annotated
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langchain_core.messages import HumanMessage

class State(TypedDict):
    messages: Annotated[list, add_messages]
```

> **Why `add_messages` as a reducer?**  
> Without a reducer, LangGraph overwrites the `messages` field on every state update. `add_messages` is a built-in reducer that *appends* new messages to the existing list, preserving the full conversation history across turns.

### Node Logic

The chatbot node passes the entire message history to the LLM and appends the AI response back to state.

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")

def chatbot(state: State) -> State:
    response = llm.invoke(state["messages"])
    return {"messages": [response]}   # add_messages reducer appends this
```

### Graph Assembly

```python
graph = StateGraph(State)

graph.add_node("chatbot", chatbot)
graph.set_entry_point("chatbot")
graph.add_edge("chatbot", END)

app = graph.compile()
```

### Basic Interaction Loop

```python
while True:
    user_input = input("You: ")
    if user_input.lower() == "quit":
        break
    result = app.invoke({"messages": [HumanMessage(content=user_input)]})
    print("AI:", result["messages"][-1].content)
```

---

## Part 2 — The Amnesia Problem

### What Goes Wrong

In the loop above, every call to `app.invoke()` passes only the **current message** with no prior history. The graph initializes a fresh state on each invocation — the LLM has no memory of previous turns.

```
Turn 1: invoke({"messages": [HumanMessage("My name is Shahzaib")]})  → state starts fresh
Turn 2: invoke({"messages": [HumanMessage("What is my name?")]})     → state starts fresh again → LLM has no idea
```

This is the **amnesia problem**: stateless invocations with no persistence between calls.

---

## Part 3 — Solving Amnesia with Checkpointers

### Concept

A **checkpointer** automatically saves the full graph state after every node execution. On the next invocation, LangGraph fetches the saved state for that session and resumes from where it left off.

### Implementation — `MemorySaver`

`MemorySaver` is LangGraph's built-in in-RAM checkpointer — suitable for development and demos.

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()

app = graph.compile(checkpointer=memory)   # Attach checkpointer at compile time
```

### Threading — Separating User Sessions

Every session is identified by a `thread_id` passed inside a `config` dict at invocation time. This allows multiple independent conversations to coexist in memory without bleeding into each other.

```python
config = {"configurable": {"thread_id": "user_session_1"}}

result = app.invoke(
    {"messages": [HumanMessage(content=user_input)]},
    config=config
)
```

### Full Persistent Chatbot Loop

```python
config = {"configurable": {"thread_id": "user_session_1"}}

while True:
    user_input = input("You: ")
    if user_input.lower() == "quit":
        break
    result = app.invoke(
        {"messages": [HumanMessage(content=user_input)]},
        config=config
    )
    print("AI:", result["messages"][-1].content)
```

On each iteration:
1. LangGraph checks `thread_id` against `MemorySaver`
2. Fetches existing `messages` list for that thread
3. Appends the new `HumanMessage` via `add_messages` reducer
4. Runs the chatbot node with full history
5. Appends the `AIMessage` response and saves updated state back to RAM

---

## How the Persistence Flow Works

```
invoke(new_message, config) 
        │
        ▼
  Checkpointer checks thread_id
        │
        ▼
  Load existing state (messages list) from memory
        │
        ▼
  add_messages reducer appends new message
        │
        ▼
  Chatbot node: LLM sees full history → generates response
        │
        ▼
  add_messages reducer appends AI response
        │
        ▼
  Checkpointer saves updated state back to memory
        │
        ▼
  Return result
```

---

## Critical Caveat — RAM vs. Production Persistence

| | `MemorySaver` | Production Checkpointer |
|---|---|---|
| Storage | RAM | Database (PostgreSQL, Redis, etc.) |
| Survives restart | ❌ No | ✅ Yes |
| Multi-process safe | ❌ No | ✅ Yes |
| Use case | Development / demos | Production systems |

`MemorySaver` is lost the moment the Python process restarts. Production deployments must swap it for a database-backed checkpointer to guarantee long-term persistence across sessions and deployments.

---

## Key Principles

- **`add_messages`** is the correct reducer for any message list field — never let LangGraph overwrite conversation history.
- The **checkpointer** is attached at `graph.compile()` time, not at invocation — making it straightforward to swap implementations.
- **`thread_id`** is the sole mechanism for session isolation — different users or contexts must use different thread IDs.
- The chatbot node always receives the **full message history**, so the LLM naturally handles multi-turn context without any extra prompt engineering.

---

## What's Next

The following video goes deeper into:
- **Checkpointer internals** — how state snapshots are structured and versioned
- **Advanced threading** — managing multiple concurrent sessions
- **Human-in-the-Loop (HITL)** — injecting human review steps before the graph proceeds

[How to build a Chatbot using LangGraph | Agentic AI using LangGraph | Video 10 | CampusX](https://www.youtube.com/watch?v=51Ve2tE3Zns&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=11)

