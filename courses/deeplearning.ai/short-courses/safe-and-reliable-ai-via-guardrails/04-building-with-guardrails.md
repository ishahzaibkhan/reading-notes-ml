# Implementing Your First Guardrail
### Use Case: Preventing Information Leakage About "Project Colosseum"

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **Validator** | Core logic that checks inputs/outputs conform to a specific rule. The fundamental building block of a guardrail. |
| **Guard** | A container in your application stack that holds one or more validators and handles processing of inputs/outputs. |
| **OnFailAction** | Configuration that specifies what happens when a validator fails (e.g., raise exception, fix, log). |
| **PassResult** | Returned by validator when the value meets the criteria — no arguments needed. |
| **FailResult** | Returned by validator when the value violates the rule — contains an error message and optional fixed value. |

> A **guard** can contain **multiple guardrails** (validators) and run them all simultaneously.

---

## Required Imports (GuardrailsAI SDK)
```python
from guardrails import Guard, OnFailAction, settings
from guardrails.validators import Validator, register_validator
from guardrails.types import ValidationResult, PassResult, FailResult
```

- `Guard` — Container for multiple validators
- `OnFailAction` — Defines failure behavior
- `Validator` — Base class to subclass for custom validators
- `register_validator` — Registers validator by name for the orchestration framework
- `PassResult` / `FailResult` — Return types for validation outcomes
- `ValidationResult` — Used for type hinting

---

## The Problem Being Solved

The RAG chatbot, despite system prompt instructions saying *"do not respond to questions about Project Colosseum"*, still leaked proprietary pizza recipe details (e.g., crust ratios) when prompted by a user attempting to extract secrets.

---

## Step 1: Build a Custom Validator
```python
@register_validator(name="colosseum_detector", data_type="string")
class ColosseumDetector(Validator):
    def validate(self, value: str, metadata: dict) -> ValidationResult:
        if "Colosseum" in value:
            return FailResult(
                error_message="Colosseum detected.",
                fix_value="I'm sorry, I can't answer questions about Project Colosseum."
            )
        return PassResult()
```

**How it works:**
- Subclasses `Validator` from GuardrailsAI
- Registered with the name `colosseum_detector`
- `validate()` takes the input string and metadata
- Returns `FailResult` with an error message and a graceful fallback response if "Colosseum" is found
- Returns `PassResult` if safe

---

## Step 2: Create a Guard Using the Validator
```python
colosseum_guard = Guard(name="Colosseum Guard")
    .use(ColosseumDetector(on_fail=OnFailAction.EXCEPTION))
```

- `on_fail=OnFailAction.EXCEPTION` → raises an exception immediately on detection
- `on_fail=OnFailAction.FIX` → returns the `fix_value` gracefully instead of crashing (better for UX)
- By default, guards run on **output**; configured here to run on **input** (messages)

---

## Step 3: Connect to Guardrails Server

The **Guardrails Server** wraps your LLM API call with input/output guards. It is:
- **OpenAI API-compatible** — swap it in for your OpenAI client with one line
- Runnable **locally** or **hosted online**
- Independently scalable (including GPU resources for ML-based validators)
- Easy to containerize for cloud deployment
```python
# Replace standard OpenAI client with guarded client
guarded_client = OpenAI(base_url="http://localhost:<guardrails_server_port>/guards/Colosseum Guard/openai/v1")
```

Then initialize the chatbot as before, simply swapping `client` for `guarded_client`.

---

## Result

**Before guardrail:** Chatbot reveals secret pizza crust ratios and recipe details.

**After guardrail:** 
- `EXCEPTION` mode → `ValidationError: Colosseum detected.` — blocks the flow entirely
- `FIX` mode → Returns: *"I'm sorry, I can't answer questions about Project Colosseum."* — graceful UX

---

## OnFailAction Options Summary

| Action | Behavior |
|--------|----------|
| `EXCEPTION` | Raises an exception, halts application flow |
| `FIX` | Returns the `fix_value` defined in FailResult — graceful fallback |
| *(others)* | Can also log silently without blocking |

---

## Guard Flow Recap
```
User Input
    ↓
[INPUT GUARD — ColosseumDetector runs]
    ↓ (if passes)
[LLM Call]
    ↓
[OUTPUT GUARD — optional additional validators]
    ↓
Response to User
```

---

## Key Takeaways

- Custom validators are simple Python classes that subclass `Validator`
- A single `Guard` can house many validators for layered protection
- `OnFailAction` gives fine-grained control over failure handling
- Guardrails Server makes it trivial to plug guards into any OpenAI-compatible LLM
- Even simple regex/string-matching validators can prevent significant data leakage
