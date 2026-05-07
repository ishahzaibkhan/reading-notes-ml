# LangGraph Series #5 — Linear Sequential Workflows

## 1. Environment Setup

Create a dedicated project folder and isolate dependencies with a virtual environment.

**Install dependencies:**
```bash
pip install langgraph langchain langchain-openai python-dotenv
```

Use **Jupyter Notebook** as the development environment — it renders LangGraph visualizations inline, which is essential for understanding graph structure.

---

## 2. Core Concepts

### State
- Defined as a `TypedDict`
- Acts as the **single source of truth** shared across all nodes
- All fields persist throughout the entire graph execution
- Each node reads from and writes back to this shared state object

### Nodes
- Plain Python functions (or LLM-powered functions)
- Receive the current `State` as input, return a partial/full updated `State`

### Edges
- Define the execution order: `START → node_1 → node_2 → END`
- In linear workflows, edges are unconditional (no branching)

---

## 3. Workflow 1 — BMI Calculator (Non-LLM)

A pure Python sequential workflow. No LLM involved. Good for understanding the graph mechanics in isolation.

### State Definition
```python
from typing import TypedDict

class BMIState(TypedDict):
    weight: float
    height: float
    bmi: float
    label: str
```

### Node 1 — Calculate BMI
```python
def calculate_bmi(state: BMIState) -> BMIState:
    bmi = state["weight"] / (state["height"] ** 2)
    return {"bmi": round(bmi, 2)}
```

### Node 2 — Label BMI
```python
def label_bmi(state: BMIState) -> BMIState:
    bmi = state["bmi"]
    if bmi < 18.5:
        label = "Underweight"
    elif bmi < 25:
        label = "Normal"
    elif bmi < 30:
        label = "Overweight"
    else:
        label = "Obese"
    return {"label": label}
```

### Graph Assembly
```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(BMIState)

graph.add_node("calculate_bmi", calculate_bmi)
graph.add_node("label_bmi", label_bmi)

graph.add_edge(START, "calculate_bmi")
graph.add_edge("calculate_bmi", "label_bmi")
graph.add_edge("label_bmi", END)

app = graph.compile()
```

### Execution
```python
result = app.invoke({"weight": 70, "height": 1.75})
print(result)  # {'weight': 70, 'height': 1.75, 'bmi': 22.86, 'label': 'Normal'}
```

**Key insight:** Each node only returns the fields it updates. LangGraph merges the partial return with the existing state automatically.

---

## 4. Workflow 2 — Basic LLM QA

Demonstrates how LangChain LLM components plug directly into LangGraph nodes.

### State Definition
```python
class QAState(TypedDict):
    question: str
    answer: str
```

### Node — LLM QA
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

def llm_qa(state: QAState) -> QAState:
    question = state["question"]
    response = llm.invoke(question)
    return {"answer": response.content}
```

### Graph Assembly
```python
graph = StateGraph(QAState)
graph.add_node("llm_qa", llm_qa)
graph.add_edge(START, "llm_qa")
graph.add_edge("llm_qa", END)

app = graph.compile()
```

### Execution
```python
result = app.invoke({"question": "What is the capital of France?"})
print(result["answer"])
```

**Key insight:** The state persists both `question` and `answer` after execution — unlike a raw LangChain chain which only returns the final output.

---

## 5. Workflow 3 — Prompt Chaining (Blog Generator)

**Prompt Chaining** = decomposing a complex task into a sequence of smaller, focused LLM calls where each step's output feeds into the next.

### Why LangGraph over plain LangChain here?
In a standard LangChain chain, intermediate outputs are not retained — only the final result is returned. LangGraph preserves **all state fields** at every step, making it far better suited for multi-step, stateful pipelines.

### State Definition
```python
class BlogState(TypedDict):
    title: str
    outline: str
    content: str
```

### Node 1 — Create Outline
```python
def create_outline(state: BlogState) -> BlogState:
    prompt = f"Create a detailed outline for a blog post titled: {state['title']}"
    response = llm.invoke(prompt)
    return {"outline": response.content}
```

### Node 2 — Write Blog
```python
def create_blog(state: BlogState) -> BlogState:
    prompt = f"""Write a full blog post titled '{state['title']}' 
    based on the following outline:\n{state['outline']}"""
    response = llm.invoke(prompt)
    return {"content": response.content}
```

### Graph Assembly
```python
graph = StateGraph(BlogState)

graph.add_node("create_outline", create_outline)
graph.add_node("create_blog", create_blog)

graph.add_edge(START, "create_outline")
graph.add_edge("create_outline", "create_blog")
graph.add_edge("create_blog", END)

app = graph.compile()
```

### Execution
```python
result = app.invoke({"title": "The Future of Artificial Intelligence"})
print(result["outline"])
print(result["content"])
```

After execution, `result` contains all three fields — `title`, `outline`, and `content` — fully populated.

---

## 6. Homework — Add an Evaluation Node

Extend the Blog Generator by adding a third node that scores the generated blog.

### State Update
```python
class BlogState(TypedDict):
    title: str
    outline: str
    content: str
    score: int        # add this
    feedback: str     # optional but useful
```

### Node 3 — Evaluate Blog
```python
def evaluate_blog(state: BlogState) -> BlogState:
    prompt = f"""You are a content evaluator. Given the following blog outline and content, 
    rate the quality of the blog on a scale of 1–10 and provide brief feedback.

    Outline:\n{state['outline']}
    
    Content:\n{state['content']}
    
    Respond in this format:
    Score: <number>
    Feedback: <one or two sentences>"""
    
    response = llm.invoke(prompt)
    # parse score and feedback from response.content
    return {"score": ..., "feedback": ...}
```

### Updated Graph
```python
graph.add_node("evaluate_blog", evaluate_blog)
graph.add_edge("create_blog", "evaluate_blog")
graph.add_edge("evaluate_blog", END)
```

---

## Summary

| Workflow | Nodes | Uses LLM | Demonstrates |
|---|---|---|---|
| BMI Calculator | `calculate_bmi` → `label_bmi` | No | Basic graph mechanics, state propagation |
| QA Bot | `llm_qa` | Yes | LangChain + LangGraph integration |
| Blog Generator | `create_outline` → `create_blog` | Yes | Prompt chaining, full state persistence |
| Blog + Evaluator *(homework)* | + `evaluate_blog` | Yes | Extending graphs, structured LLM output |

**Core takeaway:** LangGraph's `State` object is what makes it superior to plain LangChain chains for multi-step tasks — every field survives every node, giving you full observability and control over the entire pipeline.

[Sequential Workflows in LangGraph | Agentic AI using LangGraph | Video 5 | CampusX](https://www.youtube.com/watch?v=bAWujyAl1Kk&list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL&index=7)


