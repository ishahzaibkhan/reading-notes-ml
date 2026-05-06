# LangGraph Core Concepts — Foundational Guide
*Source: CampusX*

---

## 1. What is LangGraph?

LangGraph is an **orchestration framework** for building intelligent, stateful, multi-step LLM workflows.

- Represents workflows as a **directed graph**:
  - **Nodes** → individual tasks (LLM call, tool use, decision-making)
  - **Edges** → execution flow between tasks
- Natively supports:
  - Parallel execution
  - Loops and cycles
  - Conditional branching
  - Memory via state tracking
  - Resumability (pause and continue)

---

## 2. LLM Workflows

An **LLM workflow** is a series of steps where one or more tasks rely on LLMs to achieve a goal.

### 2.1 Prompt Chaining
- **Sequential** execution of LLM calls.
- Output of one step becomes input for the next.
- Example: Generate outline → expand into full report → format output.
- Best for: Linear, dependent tasks where order matters.

### 2.2 Routing
- A **central LLM acts as a decision-maker**.
- Analyzes the user's intent and routes the query to the appropriate specialized sub-agent.
- Example: A customer support router directing billing queries to a billing agent and technical queries to a tech agent.
- Best for: Systems with multiple specialized agents handling distinct domains.

### 2.3 Parallelization
- Breaks a large task into **independent sub-tasks executed simultaneously**.
- Sub-tasks have no dependency on each other and can run concurrently.
- Example: Running toxicity check, spam check, and factuality check on a piece of content all at once.
- Best for: Tasks that can be decomposed and don't need to wait on each other.

### 2.4 Orchestrator-Worker
- A **dynamic pattern** where an orchestrator node decides the nature and structure of sub-tasks **at runtime**, based on the specific input received.
- Unlike static parallelization, the sub-tasks are not predefined — the orchestrator generates them on the fly.
- Best for: Complex, open-ended tasks where the breakdown strategy depends on the input.

### 2.5 Evaluator-Optimizer
- An **iterative feedback loop**:
  1. A **generator** node produces a solution.
  2. An **evaluator** node critiques it and provides feedback.
  3. The loop repeats until the solution meets the defined criteria.
- Best for: Tasks requiring quality control — writing, code generation, planning.

### Workflow Pattern Summary

| Pattern | Execution Style | Key Mechanism |
|---|---|---|
| Prompt Chaining | Sequential | Output → next input |
| Routing | Conditional | Intent-based branching |
| Parallelization | Concurrent | Independent sub-tasks |
| Orchestrator-Worker | Dynamic | Runtime task decomposition |
| Evaluator-Optimizer | Iterative | Generate → critique → refine |

---

## 3. Core Building Blocks: Graphs, Nodes, and Edges

### Nodes
- Represent a **single, atomic task** in the workflow.
- Implemented in code as **plain Python functions**.
- Examples: Call an LLM, invoke a tool, make a decision, format output.

### Edges
- Define **when and how** to move from one node to the next.
- Three types:

| Edge Type | Behavior |
|---|---|
| **Sequential** | Always moves to a fixed next node |
| **Conditional (Branched)** | Chooses the next node based on state/logic |
| **Cyclic** | Loops back to a previous node (enables iteration) |

---

## 4. State and Reducers

### State
- A **shared, mutable memory object** that persists throughout the entire graph execution.
- Every node has access to the state — it can **read from it and write to it**.
- Defined as a `TypedDict` (typed Python dictionary) specifying all fields the workflow needs to track.
- Acts as the single source of truth across the workflow.

### Reducers
- Functions that define **how a node's updates are applied to the state**.
- Without a reducer: a node's output simply **replaces** the existing value.
- With a reducer: the output is **merged or appended** according to custom logic.

| Behavior | Without Reducer | With Reducer |
|---|---|---|
| Chat history | New message overwrites history | New message appended to history |
| Iterative drafts | Latest draft replaces previous | All drafts accumulated |
| Scores/metrics | Last value kept | Values aggregated |

- Reducers are **critical** for maintaining chat history, tracking iterative progress, and accumulating results across loop cycles.

---

## 5. LangGraph Execution Model

LangGraph's execution model is inspired by **Google Pregel** — a distributed graph processing system.

### Workflow Lifecycle

    1. Graph Definition  →  2. Compilation  →  3. Invocation

**Step 1 — Graph Definition:**
- Define all nodes (Python functions), edges (transitions), and the state schema (`TypedDict`).

**Step 2 — Compilation:**
- LangGraph verifies the graph structure.
- Checks for issues like orphaned nodes (nodes with no incoming or outgoing edges).
- Produces an executable graph object.

**Step 3 — Invocation/Execution:**
- An initial state is passed to the **entry node**.
- The graph begins execution from there.

---

### Message Passing & Supersteps

**Message Passing:**
- As each node finishes, it **updates the shared state**.
- The framework automatically passes this updated state along the outgoing edges to the next node(s).
- Nodes never call each other directly — they communicate purely through state updates.

**Supersteps:**
- A **superstep** is one complete round of execution.
- If multiple nodes are triggered simultaneously (e.g., after a parallel split), they are all grouped into a **single superstep** and executed concurrently.
- The workflow **terminates** when there are no active nodes and no messages left to pass.

### Execution Flow Diagram

    Initial State
         |
         v
    [ Entry Node ] → updates state → passes to next node(s)
         |
    [ Superstep N ] → parallel nodes execute concurrently
         |
    [ Next Node(s) ] → read updated state, continue execution
         |
         v
    No active nodes + no pending messages → STOP

---

## Key Takeaway

> LangGraph structures agentic workflows as **stateful graphs** — nodes do the work, edges control the flow, state carries the memory, reducers manage updates, and the Pregel-inspired execution engine handles parallelism, loops, and termination automatically.

[LangGraph Core Concepts | Agentic AI using LangGraph | Video 4 | CampusX - YouTube](https://www.youtube.com/watch?v=D5KhiCDM9XQ&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=6)

