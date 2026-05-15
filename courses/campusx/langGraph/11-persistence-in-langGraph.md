# Persistence & Time Travel in LangGraph

## Core Concept: What is Persistence?

Persistence is the capability to **save and restore workflow state** across time — so execution history is never lost when a program exits or a node crashes.

LangGraph is built on two pillars:

| Pillar | Description |
|---|---|
| **Graph Structure** | Complex goals decomposed into discrete nodes linked by edges |
| **State Management** | A dictionary-based memory that nodes read from and write to |

**Without persistence:** state lives only in RAM and is erased the moment execution completes — only the final output survives.

**With persistence:** every intermediate state is saved automatically at each step via **Checkpointers**.

### How Checkpointers Work
A checkpointer automatically snapshots the full state at every **superstep** (each node execution cycle) of the graph. This creates a complete, inspectable history of the entire run.

---

## Implementation

### Threads & Thread IDs
To manage multiple concurrent or historical workflow runs, LangGraph uses **Threads**:

- Every execution is assigned a unique **Thread ID**
- Thread IDs isolate conversation histories and task results from one another within the database
- Multiple users or sessions can run the same graph independently without state collision

### Setup: MemorySaver (In-Memory Checkpointer)

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver

# Define your graph as usual
graph = StateGraph(YourState)
# ... add nodes and edges ...

# Attach a checkpointer at compile time
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)
```

> **Production Note:** `MemorySaver` stores state in RAM — suitable for development only. In production, swap it for a persistent backend such as **PostgreSQL** or **Redis** using their respective LangGraph checkpointer integrations.

### Invoking with a Thread ID

```python
config = {"configurable": {"thread_id": "thread-001"}}

# First invocation — starts the workflow
result = app.invoke({"input": "your input here"}, config=config)

# Second invocation on the SAME thread — resumes with full prior context
result = app.invoke({"input": "follow-up input"}, config=config)
```

### Inspecting State

```python
# Get the current (latest) state of a thread
current_state = app.get_state(config)
print(current_state.values)        # state dictionary
print(current_state.next)          # which node runs next (empty if done)

# Get the full history of all checkpoints for a thread
history = list(app.get_state_history(config))
# history[0] → most recent checkpoint
# history[-1] → initial checkpoint
```

Each checkpoint in history contains:
- `values` — the full state dictionary at that moment
- `checkpoint_id` — unique identifier for that exact superstep
- `next` — which node was scheduled to run next

---

## Four Strategic Benefits

### 1. Short-Term Memory (Conversational Context)
By invoking the same `thread_id` repeatedly, the graph accumulates message history in state. Chatbots can naturally "remember" prior turns without any additional memory management code.

```python
config = {"configurable": {"thread_id": "chat-session-42"}}

app.invoke({"messages": [HumanMessage("What is LangGraph?")]}, config)
app.invoke({"messages": [HumanMessage("Can you give an example?")]}, config)
# Second call has full access to the first message in state
```

### 2. Fault Tolerance (Resume from Checkpoint)
If a node fails mid-execution (e.g., an API timeout), the workflow does not restart from scratch. Instead, it resumes from the **last successful checkpoint**, saving time and API costs.

```python
# After a crash, simply re-invoke with the same thread_id
# LangGraph detects the last valid checkpoint and continues from there
app.invoke({"input": "..."}, config=config)
```

### 3. Human-in-the-Loop (HITL)
Persistence enables pausing a workflow mid-execution to wait for human approval or input, then resuming exactly where it stopped.

```python
# Compile with an interrupt point before a sensitive node
app = graph.compile(
    checkpointer=checkpointer,
    interrupt_before=["sensitive_node"]
)

# Workflow pauses — state is saved automatically
app.invoke(initial_input, config)

# Human reviews state, approves, then resumes
app.invoke(None, config)  # passing None resumes from the saved checkpoint
```

### 4. Time Travel (Advanced Debugging)
The most powerful benefit — the ability to **replay, inspect, and fork** past executions. Covered in detail in the next section.

---

## Time Travel: Advanced Debugging

Time Travel allows you to revisit any historical checkpoint to debug behaviour, test alternative inputs, or fork execution into a new branch.

### Step 1 — Inspect History and Identify a Checkpoint

```python
history = list(app.get_state_history(config))

for checkpoint in history:
    print(checkpoint.checkpoint_id)
    print(checkpoint.values)
    print(checkpoint.next)
    print("---")
```

### Step 2 — Replay from a Past Checkpoint

Pass the target `checkpoint_id` in the config to re-run from that exact superstep:

```python
target_config = {
    "configurable": {
        "thread_id": "thread-001",
        "checkpoint_id": "<checkpoint_id_from_history>"
    }
}

# Re-executes the graph from that point forward
result = app.invoke(None, config=target_config)
```

### Step 3 — State Manipulation Before Replay (Forking)

Modify state at a historical checkpoint before replaying to test a different path:

```python
# Example: the graph originally generated a joke about 'Pizza'
# We want to re-run with 'Samosa' as the topic instead

app.update_state(
    config=target_config,
    values={"topic": "Samosa"}  # override specific state keys
)

# Now replay — LangGraph creates a NEW execution branch from this point
result = app.invoke(None, config=target_config)
```

**What happens internally:** `update_state` creates a new checkpoint with the modified values. Re-invoking from it spawns a **forked branch** — the original execution history is fully preserved and unaffected.

---

## Architecture Summary

```
Invoke(thread_id="abc")
        │
        ▼
   [Node A] ──── checkpoint saved ────▶ state_A
        │
        ▼
   [Node B] ──── checkpoint saved ────▶ state_B
        │
        ▼
   [Node C] ──── checkpoint saved ────▶ state_C (final)

get_state_history("abc") → [state_C, state_B, state_A, initial]

update_state(checkpoint_B) + invoke(None)
        │
        ▼
   [Fork Branch] ──── new checkpoints ────▶ state_B' → state_C'
   (original branch untouched)
```

---

## Key Concepts Cheatsheet

| Concept | Summary |
|---|---|
| **Checkpointer** | Auto-saves state at every superstep; attach at `compile()` |
| **MemorySaver** | In-memory checkpointer for dev; swap with DB in production |
| **Thread ID** | Unique key isolating each workflow run's history |
| **`get_state(config)`** | Returns the latest state snapshot for a thread |
| **`get_state_history(config)`** | Returns all historical checkpoints for a thread |
| **`interrupt_before`** | Pauses graph before specified node for HITL approval |
| **`update_state(config, values)`** | Overwrites state at a checkpoint — used for forking |
| **Time Travel** | Replay from any `checkpoint_id`; fork without losing original |
| **Superstep** | One full cycle of node execution; each gets its own checkpoint |

[Persistence in LangGraph | Time Travel in LangGraph | Agentic AI using LangGraph | Video 11 | CampusX](https://www.youtube.com/watch?v=_IPP7_Bi8uA&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=12)


