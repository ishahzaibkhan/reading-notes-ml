# Parallel Workflows in LangGraph

## Core Concept

Sequential workflows execute nodes one after another. Parallel workflows **fan out** from a single node to multiple independent nodes simultaneously, then **fan in** to an aggregation node — dramatically reducing latency when tasks are independent of each other.

---

## Workflow 1: Cricket Statistics (Non-LLM)

### Problem Statement
Given a batter's input — `runs`, `balls`, `fours`, `sixes` — compute three metrics **simultaneously**:
- **Strike Rate** = (runs / balls) × 100
- **Balls per Boundary** = balls / (fours + sixes)
- **Boundary Percentage** = ((fours × 4 + sixes × 6) / runs) × 100

### State Definition

```python
from typing import TypedDict

class CricketState(TypedDict):
    runs: int
    balls: int
    fours: int
    sixes: int
    strike_rate: float
    balls_per_boundary: float
    boundary_percentage: float
```

### Node Definitions
Each node performs one calculation and returns **only its own key**.

```python
def calculate_strike_rate(state: CricketState):
    return {"strike_rate": (state["runs"] / state["balls"]) * 100}

def calculate_balls_per_boundary(state: CricketState):
    boundaries = state["fours"] + state["sixes"]
    return {"balls_per_boundary": state["balls"] / boundaries}

def calculate_boundary_percentage(state: CricketState):
    boundary_runs = (state["fours"] * 4) + (state["sixes"] * 6)
    return {"boundary_percentage": (boundary_runs / state["runs"]) * 100}
```

### Graph Construction

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(CricketState)

graph.add_node("strike_rate_node", calculate_strike_rate)
graph.add_node("balls_per_boundary_node", calculate_balls_per_boundary)
graph.add_node("boundary_percentage_node", calculate_boundary_percentage)

# Fan-out: START → all three nodes in parallel
graph.add_edge(START, "strike_rate_node")
graph.add_edge(START, "balls_per_boundary_node")
graph.add_edge(START, "boundary_percentage_node")

# Fan-in: all three nodes → END
graph.add_edge("strike_rate_node", END)
graph.add_edge("balls_per_boundary_node", END)
graph.add_edge("boundary_percentage_node", END)

app = graph.compile()
```

### ⚠️ Critical: The State Conflict Error

**Problem:** If a parallel node returns the **entire state** (all keys), LangGraph detects conflicting updates for shared keys from multiple nodes writing simultaneously → raises an `InvalidUpdateError`.

**Fix — Partial State Updates:** Each node must return **only the key it owns**.

```python
# ❌ Wrong — returns full state, causes conflict
def calculate_strike_rate(state):
    state["strike_rate"] = (state["runs"] / state["balls"]) * 100
    return state  # DO NOT do this

# ✅ Correct — returns only its own key
def calculate_strike_rate(state):
    return {"strike_rate": (state["runs"] / state["balls"]) * 100}
```

> **Best Practice:** Use partial state updates in *all* workflows — sequential or parallel — not just parallel ones.

---

## Workflow 2: LLM-Based UPSC Essay Evaluation

### Problem Statement
Evaluate a UPSC essay across three independent dimensions in parallel, then aggregate into a final report:
1. **Clarity of Thought** — logical structure, coherence
2. **Depth of Analysis** — evidence, nuance, critical thinking
3. **Language Quality** — grammar, vocabulary, expression

### Advanced Techniques Required

#### 1. Structured Output with Pydantic
LLMs return unstructured text by default. To reliably extract scores and feedback, bind a Pydantic schema to the model.

```python
from pydantic import BaseModel
from langchain_openai import ChatOpenAI

class EvaluationResult(BaseModel):
    score: int       # 1–10
    feedback: str    # Detailed textual feedback

llm = ChatOpenAI(model="gpt-4o-mini")
structured_llm = llm.with_structured_output(EvaluationResult)
```

The model now always returns a valid `EvaluationResult` object — no parsing, no hallucinated formats.

#### 2. Reducer Functions for List Merging
When multiple parallel nodes write to the **same list key** in state, the default behavior is to **overwrite** — only the last node's value survives.

**Fix — `operator.add` as a reducer:**

```python
from typing import Annotated
import operator

class EssayState(TypedDict):
    essay: str
    scores: Annotated[list, operator.add]  # merges lists instead of overwriting
    final_score: float
    final_feedback: str
```

`Annotated[list, operator.add]` tells LangGraph: when multiple nodes write to `scores`, **concatenate** the lists rather than replace. Each node appends `[score_value]` and the reducer merges them automatically.

### State Definition

```python
from typing import TypedDict, Annotated
import operator

class EssayState(TypedDict):
    essay: str
    scores: Annotated[list, operator.add]
    final_score: float
    final_feedback: str
```

### Node Definitions

```python
from langchain_core.prompts import ChatPromptTemplate

def evaluate_clarity(state: EssayState):
    prompt = ChatPromptTemplate.from_template(
        "Evaluate the clarity of thought in this UPSC essay. "
        "Assess logical structure and coherence. Essay: {essay}"
    )
    chain = prompt | structured_llm
    result = chain.invoke({"essay": state["essay"]})
    return {"scores": [result.score]}  # list so reducer can concatenate

def evaluate_depth(state: EssayState):
    prompt = ChatPromptTemplate.from_template(
        "Evaluate the depth of analysis in this UPSC essay. "
        "Assess evidence, nuance, and critical thinking. Essay: {essay}"
    )
    chain = prompt | structured_llm
    result = chain.invoke({"essay": state["essay"]})
    return {"scores": [result.score]}

def evaluate_language(state: EssayState):
    prompt = ChatPromptTemplate.from_template(
        "Evaluate the language quality of this UPSC essay. "
        "Assess grammar, vocabulary, and expression. Essay: {essay}"
    )
    chain = prompt | structured_llm
    result = chain.invoke({"essay": state["essay"]})
    return {"scores": [result.score]}
```

### Final Aggregation Node
Runs **after** all three parallel evaluators complete (fan-in). Uses a plain (non-structured) LLM to synthesize feedback.

```python
def final_evaluation(state: EssayState):
    avg_score = sum(state["scores"]) / len(state["scores"])

    plain_llm = ChatOpenAI(model="gpt-4o-mini")
    prompt = ChatPromptTemplate.from_template(
        "An essay was evaluated across three dimensions with scores: {scores}. "
        "The average score is {avg_score:.1f}/10. "
        "Write a concise, constructive summary feedback for the student."
    )
    chain = prompt | plain_llm
    feedback = chain.invoke({
        "scores": state["scores"],
        "avg_score": avg_score
    })

    return {
        "final_score": avg_score,
        "final_feedback": feedback.content
    }
```

### Graph Construction

```python
graph = StateGraph(EssayState)

graph.add_node("evaluate_clarity", evaluate_clarity)
graph.add_node("evaluate_depth", evaluate_depth)
graph.add_node("evaluate_language", evaluate_language)
graph.add_node("final_evaluation", final_evaluation)

# Fan-out
graph.add_edge(START, "evaluate_clarity")
graph.add_edge(START, "evaluate_depth")
graph.add_edge(START, "evaluate_language")

# Fan-in
graph.add_edge("evaluate_clarity", "final_evaluation")
graph.add_edge("evaluate_depth", "final_evaluation")
graph.add_edge("evaluate_language", "final_evaluation")

graph.add_edge("final_evaluation", END)

app = graph.compile()
```

### Invocation

```python
result = app.invoke({"essay": "<your essay text here>", "scores": []})
print(f"Score: {result['final_score']:.1f}/10")
print(f"Feedback: {result['final_feedback']}")
```

> **Validation:** Test with a high-quality essay (expects high scores) and a deliberately poor essay (expects low scores) to confirm the pipeline responds dynamically to content quality.

---

## Architecture Summary

```
                    ┌──────────────────┐
                    │      START       │
                    └────────┬─────────┘
           ┌─────────────────┼─────────────────┐
           ▼                 ▼                 ▼
   [evaluate_clarity] [evaluate_depth] [evaluate_language]
           └─────────────────┼─────────────────┘
                             ▼
                    ┌─────────────────┐
                    │ final_evaluation │
                    └────────┬────────┘
                             ▼
                           END
```

---

## Key Concepts Cheatsheet

| Concept | Rule |
|---|---|
| Partial state updates | Always return only the keys a node modifies |
| State conflict error | Caused by parallel nodes returning overlapping keys |
| `Annotated[list, operator.add]` | Merges lists from parallel writes instead of overwriting |
| `with_structured_output(Schema)` | Binds Pydantic model to LLM for guaranteed JSON output |
| Fan-out | `add_edge(START, node)` for each parallel node |
| Fan-in | `add_edge(node, aggregator)` from each parallel node |
| Structured vs plain LLM | Use structured LLM for extraction; plain LLM for generation |

[Parallel Workflows in LangGraph | Agentic AI using LangGraph | Video 6 | CampusX](https://www.youtube.com/watch?v=O6ryuSpqdOw)

