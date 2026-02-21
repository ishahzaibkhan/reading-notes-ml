# Topic Guardrail — Keeping Your Chatbot On-Topic
### Using Zero-Shot Classification to Prevent Off-Topic Responses

---

## The Problem

The pizza chatbot, when asked about Ford F-150 vs. Ford Ranger differences, returned a detailed comparison — completely ignoring its intended purpose. System prompt instructions alone were insufficient to prevent this.

---

## Core Model: Zero-Shot Topic Classification (BART)

The model used is **`facebook/bart-large-mnli`** (Meta). It works similarly to NLI:

| NLI Role | Topic Classification Equivalent |
|----------|-------------------------------|
| Premise | Input text to classify |
| Hypothesis | *"This sentence contains discussions of [TOPIC]"* |
| Output | Probability score per topic (entailed = likely on-topic) |

**Toy Example** — Sentence: *"Chick-fil-A is closed on Sundays"*

| Topic | Score |
|-------|-------|
| Food | 0.672 |
| Business | 0.179 |
| Politics | 0.027 |

> Only topics scoring above a configurable **threshold** are considered detected.

---

## Zero-Shot Classifier vs. LLM for Classification

| Factor | LLM (e.g. GPT-4o-mini) | Zero-Shot Classifier (BART) |
|--------|------------------------|----------------------------|
| **Consistency** | ❌ Stochastic — different answers each run | ✅ Deterministic — same result every time |
| **Speed** | ❌ ~30s for 10 requests (network dependent) | ✅ Seconds on M1 chip locally |
| **Cost** | ❌ API rate limits, pricing, uptime dependency | ✅ Runs locally, no third-party dependency |
| **Data Privacy** | ❌ User messages sent to third-party | ✅ Data stays within your own environment |
| **Quality** | Varies by model; better models = better results | Consistent, good enough for production |

> For production guardrails, the **small local model is preferred** over LLMs for classification tasks due to speed, consistency, and privacy.

---

## Implementation

### Step 1: Topic Detection Function
```python
def detect_topics(text: str, topics: list[str], threshold: float = 0.5) -> list[str]:
    classifier = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
    result = classifier(
        text,
        candidate_labels=topics,
        hypothesis_template="This sentence above contains discussions of the following topics: {}."
    )
    return [
        topic for topic, score in zip(result["labels"], result["scores"])
        if score > threshold
    ]
```

Returns only topics whose confidence score exceeds the threshold.

### Step 2: Constrained Topic Validator
```python
@register_validator(name="constrained_topic", data_type="string")
class ConstrainedTopicValidator(Validator):
    def __init__(self, banned_topics: list[str], threshold: float = 0.5, **kwargs):
        self.banned_topics = banned_topics
        self.threshold = threshold
        super().__init__(**kwargs)

    def validate(self, value: str, metadata: dict) -> ValidationResult:
        detected = detect_topics(value, self.banned_topics, self.threshold)
        if detected:
            return FailResult(
                error_message=f"Banned topics detected: {detected}"
            )
        return PassResult()
```

### Step 3: Build and Test the Guard
```python
topic_guard = Guard().use(
    ConstrainedTopicValidator(
        banned_topics=["automobiles", "politics", "sports"],
        on_fail=OnFailAction.EXCEPTION
    )
)
```

**Test result** on a political statement: Guard fails with `"politics detected, automobiles detected"` — as expected.

---

## Deploying via Guardrails Server
```python
guarded_client = OpenAI(
    base_url="http://localhost:<port>/guards/topic_guard/openai/v1"
)
```

Same one-line swap pattern. The guard runs topic classification on LLM **output** before it reaches the user.

**Result on Ford F-150 prompt:** `ValidationError` raised — automobiles detected in output, blocked before reaching user.

---

## Guardrails Hub

For production, Guardrails offers a **state-of-the-art pre-built topic classifier** available on the **Guardrails Hub**. Install it in your local environment and use it directly in your server config — no need to build from scratch.

---

## Graceful Error Handling

As with previous guards, catch exceptions rather than surfacing raw errors:
```python
try:
    response = guarded_client.chat.completions.create(...)
except ValidationError:
    response = "I can only assist with questions related to our pizzeria. Please ask about our menu, hours, or delivery!"
```
