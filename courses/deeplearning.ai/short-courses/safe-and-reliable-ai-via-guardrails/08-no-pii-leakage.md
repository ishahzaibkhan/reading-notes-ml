# PII Detection & Handling in GenAI Applications
### Using Microsoft Presidio for Personally Identifiable Information Guardrails

---

## Why PII Matters in GenAI

**PII (Personally Identifiable Information)** includes names, emails, phone numbers, social security numbers, and anything that could identify a person. Two critical risk vectors exist in LLM applications:

| Direction | Risk |
|-----------|------|
| **User → LLM (Input)** | Customers inadvertently share sensitive info that gets forwarded to third-party LLM providers |
| **LLM → User (Output)** | LLM accidentally reveals private organizational data (employees, other customers) in its responses |

> Most developers use third-party LLM APIs, meaning **customer data crosses the internet** to an external provider. Regulated industries (healthcare, banking, government) have strict legal requirements around this.

---

## Tool: Microsoft Presidio

An open-source project by Microsoft for **analyzing and anonymizing PII**. Two core components:

| Component | Function |
|-----------|---------|
| **AnalyzerEngine** | Detects PII entities in text — returns type, position, and confidence |
| **AnonymizerEngine** | Replaces detected PII with placeholder labels, preserving text usability |

### Presidio in Action

**Input:** *"Can you tell me the orders I've placed in the last month? My name is Hank Tate and my phone number is 555-867-5309."*

**Analyzer Output:**
| Entity | Type | Position |
|--------|------|---------|
| Date reference | DATETIME | chars 43–60 |
| Hank Tate | PERSON | chars 73–82 |
| 555-867-5309 | PHONE_NUMBER | end of string |

**Anonymizer Output:**
*"Can you tell me the orders I've placed in \<DATETIME\>? My name is \<PERSON\> and my phone number is \<PHONE_NUMBER\>."*

> The anonymized text remains usable for answering the user's actual question, while sensitive data is stripped out.

---

## Implementation: PII Input Validator

### Step 1: PII Detection Function
```python
from presidio_analyzer import AnalyzerEngine

def detect_pii(text: str, entities: list[str] = ["PERSON", "PHONE_NUMBER"]) -> list:
    analyzer = AnalyzerEngine()
    results = analyzer.analyze(text=text, entities=entities, language="en")
    return results  # list of detected PII entities
```

Configure `entities` based on what your industry/use case requires. Full entity list available in Microsoft Presidio docs.

### Step 2: PII Validator Class
```python
@register_validator(name="pii_detector", data_type="string")
class PIIDetector(Validator):
    def validate(self, value: str, metadata: dict) -> ValidationResult:
        detected = detect_pii(value)
        if detected:
            entity_types = [r.entity_type for r in detected]
            return FailResult(
                error_message=f"PII detected: {entity_types}",
                metadata={"detected_entities": detected}
            )
        return PassResult()
```

### Step 3: Guard Setup (Input Side)
```python
pii_guard = Guard().use(
    PIIDetector(on_fail=OnFailAction.EXCEPTION)
)
```

**Result on Hank's message:** `ValidationError: PII detected — PERSON, PHONE_NUMBER`

---

## Deploying via Guardrails Server (Hub Validator)

For production, use the **pre-built PII validator from Guardrails Hub** — it supports:
- Many more entity types than a custom implementation
- **Real-time streaming validation**
```python
# Input-side guard — blocks PII before it reaches the LLM
guarded_client = OpenAI(
    base_url="http://localhost:<port>/guards/pii_input_guard/openai/v1"
)
```

**What happens:** Before the message is sent to the LLM, the PII guard intercepts it. If PII is detected, an exception is raised immediately — the data never reaches the third-party API or gets stored in the backend.

**Verified by checking message history:** Only the system message is stored; Hank's message containing PII is never persisted.

---

## Output-Side PII Filtering (Streaming)

Even if PII isn't in user input, the LLM might retrieve and expose it from your vector database in its response. Run the PII guard on the **output side** with streaming:
```python
# Streaming output validation
for chunk in guarded_client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=messages,
    stream=True
):
    validated_chunk = pii_output_guard.validate(chunk)
    # PII in LLM-generated text is redacted in real-time
```

**Result:** Even if the LLM generates phone numbers or names from retrieved context, they are detected and redacted before reaching the user — with minimal latency impact.

---

## Two-Layer PII Protection Strategy
```
User Input
    ↓
[INPUT PII GUARD] — blocks/anonymizes before sending to LLM
    ↓
[LLM + RAG Retrieval]
    ↓
[OUTPUT PII GUARD] — redacts any PII in LLM response
    ↓
Clean Response to User
```

---

## Best Practices

- **Sanitize data before loading into vector DB** — prevention is better than detection
- **Use anonymization over blocking** where possible — preserve text usability while removing sensitive data
- **Define entity types per use case** — a pizzeria cares about name/phone; a hospital cares about SSN, DOB, medical record numbers
- **Align with organizational policy** — detection is step one; your org's data handling policy determines what happens next (alert, anonymize, reject, audit log)
- **Authorization-aware filtering** — not all users should see all data; output PII filtering can enforce access levels dynamically
