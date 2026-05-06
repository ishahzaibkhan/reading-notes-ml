# Agentic AI — Foundational Study Notes
*Source: CampusX*

---

## 1. What is Agentic AI?

Agentic AI is a **software paradigm** where a system, given a high-level goal, **autonomously plans and executes tasks** with minimal human intervention.

### Generative AI vs. Agentic AI

| Dimension | Generative AI | Agentic AI |
|---|---|---|
| Nature | Reactive | Proactive |
| Interaction | Prompt-by-prompt | Goal-driven |
| Autonomy | Low | High |
| Execution | Single response | Multi-step workflow |

### Example — Hiring a Back-End Engineer
- **Generative AI:** Waits for explicit instructions at every step.
- **Agentic AI:** Accepts the goal once, then autonomously:
  1. Drafts the job description
  2. Posts it to relevant platforms
  3. Monitors incoming applicants
  4. Screens and ranks resumes
  5. Schedules interviews
  6. Manages onboarding

---

## 2. Key Characteristics of Agentic AI

Every true agentic system exhibits **six core traits**:

### 2.1 Autonomy
- Makes decisions and takes actions **independently**, without step-by-step human instruction.
- Controllable via:
  - **Scope limitation** — restricting the domains the agent can act in
  - **Human-in-the-loop checkpoints** — requiring approval at critical stages
  - **Override capabilities** — allowing humans to intervene or stop execution

### 2.2 Goal-Oriented
- Operates with a **persistent objective** that guides all planning and execution.
- The goal acts as a **compass** — every sub-task and decision is evaluated against it.

### 2.3 Planning
- Breaks high-level goals into a **sequence of sub-goals**.
- The planning process involves:
  1. **Generating** candidate plans
  2. **Evaluating** them based on efficiency, tool availability, and cost
  3. **Selecting** the optimal plan for execution

### 2.4 Reasoning
- A cognitive process applied during **both planning and execution**.
- Used to:
  - Interpret incoming information
  - Draw logical conclusions
  - Handle errors (e.g., retrying or rerouting if a tool fails)

### 2.5 Adaptability
- The capacity to **modify plans or strategies** in response to unexpected events or failures mid-execution.
- Closely linked to reasoning — the agent reasons about what went wrong, then adapts.

### 2.6 Context Awareness
- Retains information across the workflow using **memory**:

| Memory Type | Scope | Examples |
|---|---|---|
| **Short-term** | Current session | Active goal, recent tool outputs, current state |
| **Long-term** | Across sessions | User preferences, historical decisions, learned patterns |

---

## 3. Core Components of Agentic AI

Every agentic architecture is built on **five primary components**:

### 3.1 Brain (LLM)
- The **cognitive core** of the agent.
- Responsible for:
  - Interpreting high-level goals
  - Reasoning through ambiguity
  - Planning sub-task sequences
  - Selecting appropriate tools

### 3.2 Orchestrator
- The **nervous system** of the architecture.
- Manages:
  - Task sequencing and ordering
  - Conditional routing (if/else logic across steps)
  - Retry logic on failure
  - Delegation between the LLM and human agents when needed

### 3.3 Tools
- **External capabilities** the agent uses to interact with the real world.
- Examples: REST APIs, resume parsers, calendar integrations, email clients, web scrapers.
- Tools extend the agent beyond pure language — they give it *actuators*.

### 3.4 Memory
- The **storage and retrieval system** for context, history, and state.
- Divided into:
  - **Short-term memory:** In-session context (current goal state, tool results)
  - **Long-term memory:** Persistent storage (user preferences, past decisions)

### 3.5 Supervisor
- A **higher-level management component** that monitors overall execution.
- Handles **escalations** — situations the agent cannot resolve autonomously.
  - *Example:* A strong candidate who doesn't perfectly match the defined hiring criteria is flagged for human review rather than auto-rejected.
- Acts as the safety net between full autonomy and human oversight.

---

## 4. Component Interaction Summary

    High-Level Goal
          |
          v
      [ Brain / LLM ] ---- reasons, plans, selects tools
          |
          v
      [ Orchestrator ] ---- sequences tasks, routes conditionally, retries on failure
          |         |
          v         v
      [ Tools ]  [ Memory ] ---- act on world / retain context
          |
          v
      [ Supervisor ] ---- monitors, escalates edge cases to humans

---

## Key Takeaway

> Agentic AI = **Brain** (LLM) + **Orchestrator** (control flow) + **Tools** (world interaction) + **Memory** (context) + **Supervisor** (oversight), all driven by six traits — Autonomy, Goal-Orientation, Planning, Reasoning, Adaptability, and Context Awareness.

[CampusX - LangChain vs LangGraph | Agentic AI (Video 1)](https://www.youtube.com/watch?v=GWnSsjT4V68&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=3)

