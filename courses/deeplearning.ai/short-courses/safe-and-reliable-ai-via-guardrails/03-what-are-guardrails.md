# AI Guardrails — What They Are & How They Work

---

## Definition

Guardrails are **secondary checks or validations** that ensure inputs or outputs of an LLM call are as expected. They represent a shift from *blindly trusting* the LLM to **explicitly verifying** that success criteria are met.

Expectations can range from simple to complex:
- Simple: Is the output correctly formatted (list, JSON schema)?
- Complex: Is the output hallucinated? Is there a jailbreak attempt?

---

## Where Guardrails Fit in the Architecture

### Standard (Unguarded) LLM Flow
```
[Prompt + RAG Retrieval + System Prompt] → [LLM] → [Output] → [Application]
```

### Guarded LLM Flow
```
[Prompt + RAG + System Prompt]
        ↓
  [INPUT GUARD] ← validates before sending to LLM
        ↓
      [LLM]
        ↓
 [OUTPUT GUARD] ← validates before returning to app
        ↓
  [Application]
```

---

## Input Guard — What It Checks

Applied **before** the prompt is sent to the LLM:

- Does the prompt contain **PII**? (block if so)
- Is the prompt **off-topic** for the intended use case?
- Is there a **jailbreak attempt** detected?

---

## Output Guard — What It Checks

Applied **after** the LLM returns a response:

- Are there **hallucinations** present?
- Are there **off-topic responses**?
- Is there **profanity or unsafe content**?
- Does the output meet any other custom success criteria?

---

## How Guardrails Work Under the Hood

Guardrails can use one or a **combination** of these technologies:

| Type | Description | Example Use Case |
|------|-------------|-----------------|
| **Rules Engine** | Regex or pattern matching | Detect PII formats, banned words |
| **Fine-tuned ML Models** | Small models running in-process | Named entity detection, topic classification, profanity detection |
| **Secondary LLM Calls** | Another LLM scores or evaluates output | "Score how well this answers the user's question" or rule alignment checks |
| **Hybrid (most common)** | Mix and match of above | Complex, nuanced validation combining speed + accuracy |

---

## Why Guardrails Increase Reliability

### 1. Enforce Inviolable Constraints
Some behaviors carry **real financial or legal consequences** if violated (e.g., leaking PII). Guardrails explicitly verify these hard constraints are never broken — not just hoped to be avoided via prompting.

### 2. Measure Undesirable Behavior
Guardrails break down the vague question *"how well does my chatbot perform?"* into **specific, measurable checks**, such as:
- How often does the LLM refuse to answer?
- How often does it go off-topic?

This gives concrete insight into system performance over time.

### 3. Contain Cascading Errors in Agentic/Multi-Step Applications
In complex multi-step or agentic workflows, errors compound across sequential LLM calls. Guardrails **draw a bounding box** around what each LLM call can do, preventing errors from propagating and amplifying downstream.

### 4. Limit Worst-Case Risk
Combining all of the above significantly reduces the ceiling of how badly things can go — making GenAI applications safer to deploy in production.

---

## Key Principle

> Don't blindly trust the LLM. **Explicitly verify** every input and output against your defined success criteria.
