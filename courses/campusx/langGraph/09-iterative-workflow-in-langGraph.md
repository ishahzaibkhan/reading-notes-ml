# Iterative Workflows in LangGraph
### CampusX Agentic AI Series — Episode 8

---

## Core Concept

Iterative (looping) workflows allow a system to **perform a task, evaluate quality, and repeatedly refine output** until a success criterion is met or a maximum iteration limit is reached.

The loop structure is:

```
Generate → Evaluate → [Approved? → END] or [Needs Improvement? → Optimize → Evaluate → ...]
```

This completes the four foundational workflow types in LangGraph:

| Pattern | Behavior |
|---|---|
| Sequential | Tasks run linearly |
| Parallel | Multiple branches run simultaneously |
| Conditional | One branch activated based on logic |
| **Iterative** | **A cycle repeats until a quality threshold is met** |

---

## Use Case — Automated Social Media Agent

**Problem:** Manual social media content creation is time-consuming, and naive single-prompt LLM automation produces mediocre, generic output.

**Solution:** A three-agent feedback loop where each agent has a distinct role:

1. **Generator** — creates the initial draft
2. **Evaluator** — critiques the draft (humor, originality, virality, length)
3. **Optimizer** — rewrites the draft using specific evaluator feedback

The loop continues until the evaluator approves or the iteration limit is hit.

**Model Strategy:**
- `GPT-4o` → Generator and Optimizer (higher quality output)
- `GPT-4o-mini` → Evaluator (faster, cheaper critic)

---

## Implementation

### State Definition

```python
from typing import TypedDict, Annotated
import operator

class TweetState(TypedDict):
    topic: str
    tweet: str
    evaluation: str        # "Approved" or "Needs Improvement"
    feedback: str
    iteration: int
    max_iteration: int     # Hard stop — set to 5
    tweet_history: Annotated[list[str], operator.add]      # Appends each draft
    feedback_history: Annotated[list[str], operator.add]   # Appends each critique
```

> **Why `Annotated` with `operator.add`?**  
> By default, LangGraph overwrites state fields on each update. Wrapping a field with `Annotated[list, operator.add]` tells LangGraph to use `operator.add` as a **reducer** — meaning each new value is *appended* to the existing list rather than replacing it. This is how history accumulates across iterations.

---

### Node 1 — Generate Tweet

Uses system + human prompts to enforce a specific tone.

```python
from langchain_openai import ChatOpenAI

gpt4o = ChatOpenAI(model="gpt-4o")

def generate_tweet(state: TweetState) -> TweetState:
    messages = [
        SystemMessage(content="You are a social media expert who writes funny and clever tweets."),
        HumanMessage(content=f"Write a tweet about: {state['topic']}")
    ]
    result = gpt4o.invoke(messages)
    state["tweet"] = result.content
    state["tweet_history"] = [result.content]   # Appended via reducer
    return state
```

---

### Node 2 — Evaluate Tweet

Uses **Structured Output** via Pydantic to guarantee a clean, parseable schema — no fragile string parsing.

```python
from pydantic import BaseModel
from typing import Literal

class EvaluationSchema(BaseModel):
    evaluation: Literal["Approved", "Needs Improvement"]
    feedback: str

gpt4o_mini = ChatOpenAI(model="gpt-4o-mini")
evaluator = gpt4o_mini.with_structured_output(EvaluationSchema)

def evaluate_tweet(state: TweetState) -> TweetState:
    messages = [
        SystemMessage(content=(
            "You are a harsh social media critic. Evaluate tweets on: "
            "humor, originality, virality potential, and length. "
            "Be specific in your feedback."
        )),
        HumanMessage(content=f"Evaluate this tweet:\n\n{state['tweet']}")
    ]
    result = evaluator.invoke(messages)
    state["evaluation"] = result.evaluation
    state["feedback"] = result.feedback
    state["feedback_history"] = [result.feedback]   # Appended via reducer
    return state
```

---

### Node 3 — Optimize Tweet

Injects both the original draft and evaluator feedback into the prompt, then increments the iteration counter.

```python
def optimize_tweet(state: TweetState) -> TweetState:
    messages = [
        SystemMessage(content="You are a social media expert. Rewrite tweets based on critic feedback."),
        HumanMessage(content=(
            f"Original tweet:\n{state['tweet']}\n\n"
            f"Critic feedback:\n{state['feedback']}\n\n"
            f"Rewrite the tweet addressing all feedback points."
        ))
    ]
    result = gpt4o.invoke(messages)
    state["tweet"] = result.content
    state["tweet_history"] = [result.content]   # Appended via reducer
    state["iteration"] = state["iteration"] + 1
    return state
```

---

### Router Function

Checks the evaluator's verdict and enforces the max iteration hard stop.

```python
def check_evaluation(state: TweetState) -> str:
    if state["evaluation"] == "Approved":
        return "approved"
    elif state["iteration"] >= state["max_iteration"]:
        return "approved"   # Exit loop even if not approved — prevent infinite loops
    else:
        return "needs_improvement"
```

---

### Graph Assembly

```python
from langgraph.graph import StateGraph, END
from langchain_core.messages import SystemMessage, HumanMessage

graph = StateGraph(TweetState)

graph.add_node("generate_tweet", generate_tweet)
graph.add_node("evaluate_tweet", evaluate_tweet)
graph.add_node("optimize_tweet", optimize_tweet)

graph.set_entry_point("generate_tweet")
graph.add_edge("generate_tweet", "evaluate_tweet")

graph.add_conditional_edges(
    "evaluate_tweet",
    check_evaluation,
    {
        "approved":          END,
        "needs_improvement": "optimize_tweet",
    }
)

# Self-loop: optimizer feeds back into evaluator
graph.add_edge("optimize_tweet", "evaluate_tweet")

app = graph.compile()
```

---

### Workflow Diagram

```
┌────────────────┐
│ generate_tweet │
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ evaluate_tweet │◄──────────────────┐
└───────┬────────┘                   │
        │                            │
  check_evaluation()                 │
        │                            │
   ┌────┴────┐                       │
   │         │                       │
"approved" "needs_improvement"       │
   │         │                       │
   ▼         ▼                       │
  END  ┌──────────────┐              │
       │ optimize_tweet│─────────────┘
       └──────────────┘    (self-loop)
```

---

## History Tracking & Debugging

### Why History Matters

After the graph terminates, `tweet_history` and `feedback_history` contain every draft and critique from every iteration. This lets you **audit the full refinement arc** — essential for debugging and understanding model improvement.

```python
# Run the graph
result = app.invoke({
    "topic": "Machine learning engineers",
    "tweet": "",
    "evaluation": "",
    "feedback": "",
    "iteration": 0,
    "max_iteration": 5,
    "tweet_history": [],
    "feedback_history": []
})

# Audit the improvement over iterations
for i, (tweet, feedback) in enumerate(zip(result["tweet_history"], result["feedback_history"])):
    print(f"--- Iteration {i+1} ---")
    print(f"Tweet:    {tweet}")
    print(f"Feedback: {feedback}\n")
```

### Debugging the Loop

When testing with capable models like GPT-4o, the evaluator may approve the first draft immediately — making it impossible to verify the loop logic. To force multiple iterations during development:

- Switch to **smaller/weaker models** temporarily
- Use **nonsensical or low-quality inputs** as the topic
- Temporarily **tighten evaluation criteria** in the system prompt

---

## Key Principles

- **`Annotated[list, operator.add]`** is the correct pattern for any state field that should accumulate values across iterations rather than overwrite them.
- **Structured Output** (`with_structured_output`) on the evaluator node is critical — it guarantees the evaluation flag is always a clean, matchable string.
- **Max iteration guard** in the router prevents infinite loops when the model never reaches the approval threshold.
- The **self-loop** (`optimize_tweet` → `evaluate_tweet`) is what makes iterative workflows distinct from conditional ones — the graph cycles back on itself rather than terminating.
- Separating **generation** from **optimization** into two distinct nodes allows each to have its own prompt strategy, rather than forcing one node to handle both cold creation and targeted refinement.

---

## What's Next

Future modules in the series will extend iterative workflows with:

- **Tool Use** — giving agents access to external APIs and functions
- **Human-in-the-Loop (HITL)** — allowing a human to review and approve output before the graph terminates, adding oversight to automated pipelines

[Stateful Workflows in LangGraph | Agentic AI using LangGraph | Video 8 | CampusX](https://www.youtube.com/watch?v=7CbSqrovcsE&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=9&t=2s)

