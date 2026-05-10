# Conditional Workflows in LangGraph
### CampusX Agentic AI Series — Episode 7

---

## Core Concept

Conditional workflows route execution to **one path out of many** based on logical outcomes — analogous to `if-else` statements. Unlike:

- **Sequential Workflows** → tasks run one after another (linear)
- **Parallel Workflows** → multiple branches run simultaneously

**Conditional Workflows** evaluate a condition and activate only the matching branch. This makes agents *adaptive* — their behavior changes based on actual data or model output, not a fixed path.

The key LangGraph primitive is `add_conditional_edges`, which binds a routing function to the graph so it can dispatch to different nodes dynamically.

---

## Project 1 — Quadratic Equation Solver

Solves `ax² + bx + c = 0` by routing to the correct mathematical case based on the discriminant.

### State Definition

```python
from typing import TypedDict, Optional

class QuadState(TypedDict):
    a: float
    b: float
    c: float
    equation: str          # Human-readable equation string
    discriminant: float    # b² - 4ac
    result: str            # Final answer
```

### Node 1 — Show Equation

Formats raw coefficients into a readable string and stores it in state.

```python
def show_equation(state: QuadState) -> QuadState:
    a, b, c = state["a"], state["b"], state["c"]
    state["equation"] = f"{a}x² + {b}x + {c} = 0"
    return state
```

### Node 2 — Calculate Discriminant

```python
def calculate_discriminant(state: QuadState) -> QuadState:
    a, b, c = state["a"], state["b"], state["c"]
    state["discriminant"] = b**2 - 4*a*c
    return state
```

### Nodes 3a/3b/3c — Root Calculation Branches

```python
def real_roots(state: QuadState) -> QuadState:          # D > 0
    a, b, D = state["a"], state["b"], state["discriminant"]
    x1 = (-b + D**0.5) / (2*a)
    x2 = (-b - D**0.5) / (2*a)
    state["result"] = f"Two distinct real roots: x₁ = {x1:.4f}, x₂ = {x2:.4f}"
    return state

def repeated_root(state: QuadState) -> QuadState:       # D == 0
    a, b = state["a"], state["b"]
    x = -b / (2*a)
    state["result"] = f"One repeated real root: x = {x:.4f}"
    return state

def no_real_roots(state: QuadState) -> QuadState:       # D < 0
    state["result"] = "No real roots (discriminant is negative)"
    return state
```

### Router Function

The router does **not** modify state — it only reads the discriminant and returns a string key that LangGraph maps to a node.

```python
def check_condition(state: QuadState) -> str:
    D = state["discriminant"]
    if D > 0:
        return "real_roots"
    elif D == 0:
        return "repeated_root"
    else:
        return "no_real_roots"
```

### Graph Assembly

```python
from langgraph.graph import StateGraph, END

graph = StateGraph(QuadState)

graph.add_node("show_equation", show_equation)
graph.add_node("calculate_discriminant", calculate_discriminant)
graph.add_node("real_roots", real_roots)
graph.add_node("repeated_root", repeated_root)
graph.add_node("no_real_roots", no_real_roots)

graph.set_entry_point("show_equation")
graph.add_edge("show_equation", "calculate_discriminant")

graph.add_conditional_edges(
    "calculate_discriminant",
    check_condition,
    {
        "real_roots":     "real_roots",
        "repeated_root":  "repeated_root",
        "no_real_roots":  "no_real_roots",
    }
)

graph.add_edge("real_roots",    END)
graph.add_edge("repeated_root", END)
graph.add_edge("no_real_roots", END)

app = graph.compile()
```

### `add_conditional_edges` — Key Signature

```python
graph.add_conditional_edges(
    source_node,      # Node whose output triggers routing
    router_fn,        # Function that returns a string key
    path_map          # Dict mapping keys to destination node names
)
```

---

## Project 2 — LLM-Based Customer Review Handler

An advanced workflow where **an LLM's structured output** drives the routing decision, not hardcoded logic.

### State Definition

```python
from typing import TypedDict, Optional

class ReviewState(TypedDict):
    review: str
    sentiment: Optional[str]
    diagnosis: Optional[dict]
    response: Optional[str]
```

### Structured Output Schemas

LangChain's `.with_structured_output()` forces the LLM to return a validated Pydantic object — eliminating fragile string parsing.

```python
from pydantic import BaseModel
from typing import Literal

class SentimentSchema(BaseModel):
    sentiment: Literal["Positive", "Negative"]

class DiagnosisSchema(BaseModel):
    issue_type: Literal["UX", "Bug", "Performance", "Other"]
    tone: Literal["Frustrated", "Angry", "Neutral", "Disappointed"]
    urgency: Literal["High", "Medium", "Low"]
```

### Node 1 — Sentiment Extraction

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")
sentiment_extractor = llm.with_structured_output(SentimentSchema)

def extract_sentiment(state: ReviewState) -> ReviewState:
    result = sentiment_extractor.invoke(
        f"Analyze the sentiment of this review:\n\n{state['review']}"
    )
    state["sentiment"] = result.sentiment
    return state
```

### Router Function

```python
def check_sentiment(state: ReviewState) -> str:
    return "positive" if state["sentiment"] == "Positive" else "negative"
```

### Node 2a — Positive Response

```python
def positive_response(state: ReviewState) -> ReviewState:
    prompt = f"Write a warm thank-you response to this positive review:\n\n{state['review']}"
    state["response"] = llm.invoke(prompt).content
    return state
```

### Node 2b — Run Diagnosis (Negative Path)

When the review is negative, a second LLM call extracts structured issue metadata before generating a response. `.model_dump()` converts the Pydantic object to a plain dict so downstream nodes can access individual fields.

```python
diagnosis_extractor = llm.with_structured_output(DiagnosisSchema)

def run_diagnosis(state: ReviewState) -> ReviewState:
    result = diagnosis_extractor.invoke(
        f"Analyze this negative review for issue type, tone, and urgency:\n\n{state['review']}"
    )
    state["diagnosis"] = result.model_dump()
    return state
```

> **Why `.model_dump()`?** Pydantic objects aren't directly serializable into LangGraph state. Converting to a dict ensures clean propagation to downstream nodes.

### Node 3 — Negative Response (Post-Diagnosis)

Uses the enriched diagnosis context to craft a personalized, empathetic reply — not a generic apology.

```python
def negative_response(state: ReviewState) -> ReviewState:
    diagnosis = state["diagnosis"]
    prompt = (
        f"A customer left a negative review. "
        f"Issue type: {diagnosis['issue_type']}, "
        f"Tone: {diagnosis['tone']}, "
        f"Urgency: {diagnosis['urgency']}.\n\n"
        f"Review: {state['review']}\n\n"
        f"Write a personalized, empathetic resolution message."
    )
    state["response"] = llm.invoke(prompt).content
    return state
```

### Graph Assembly

```python
graph = StateGraph(ReviewState)

graph.add_node("extract_sentiment", extract_sentiment)
graph.add_node("positive_response", positive_response)
graph.add_node("run_diagnosis",     run_diagnosis)
graph.add_node("negative_response", negative_response)

graph.set_entry_point("extract_sentiment")

graph.add_conditional_edges(
    "extract_sentiment",
    check_sentiment,
    {
        "positive": "positive_response",
        "negative": "run_diagnosis",
    }
)

graph.add_edge("positive_response", END)
graph.add_edge("run_diagnosis",     "negative_response")
graph.add_edge("negative_response", END)

app = graph.compile()
```

### Full Workflow Diagram

```
                    ┌─────────────────────┐
                    │   extract_sentiment  │
                    └──────────┬──────────┘
                               │
              check_sentiment() routes here
                               │
             ┌─────────────────┴──────────────────┐
             │ "positive"                "negative" │
             ▼                                     ▼
   ┌──────────────────┐               ┌─────────────────────┐
   │ positive_response│               │    run_diagnosis     │
   └────────┬─────────┘               └──────────┬──────────┘
            │                                    │
           END                                   ▼
                                    ┌─────────────────────┐
                                    │  negative_response   │
                                    └──────────┬──────────┘
                                               │
                                              END
```

---

## Summary — Conditional vs. Other Workflow Types

| Pattern | Behavior | LangGraph Method |
|---|---|---|
| Sequential | All nodes run in order | `add_edge` |
| Parallel | Multiple nodes run simultaneously | `add_edge` to a fan-out node |
| **Conditional** | **One branch activated based on logic** | **`add_conditional_edges`** |

### Key Principles

- The **router function** reads state and returns a string — it never modifies state.
- The **path map** in `add_conditional_edges` translates router strings to node names.
- **Structured LLM outputs** (via `.with_structured_output()`) make LLM-driven routing reliable and type-safe.
- **`.model_dump()`** is necessary to convert Pydantic objects to dicts for clean state propagation.
- Multi-step branching (Sentiment → Diagnosis → Response) enables fine-grained, context-aware agent behavior.

[Hierarchical Workflows in LangGraph | Agentic AI using LangGraph | Video 7 | CampusX](https://www.youtube.com/watch?v=I-dvZqTz-Wc&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=8)


