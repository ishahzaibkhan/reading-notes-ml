# Competitor Mention Detection Guardrail
### Preventing Reputational Risk from Competitor References in LLM Output

---

## The Problem

The pizza chatbot, when asked to compare Alfredo's Pizza Cafe vs. Pizza by Alfredo, responded with a detailed breakdown of both competitors — despite being explicitly instructed not to. This poses reputational and legal risk.

---

## Why Simple Word Matching Isn't Enough

Companies are often referred to by multiple names or abbreviations:
- "JPMorgan" vs. "JPMC" vs. "JP Morgan Chase"
- "GMC" vs. "General Motors"

A simple string match would miss variants. The solution is a **cascading three-stage approach**.

---

## Cascading Competitor Detection Strategy
```
Input Text
    ↓
[Stage 1: Exact Match] ──→ Found? → FAIL immediately
    ↓ Not found
[Stage 2: Named Entity Recognition (NER)] → Extract all named entities
    ↓
[Stage 3: Vector Similarity Matching] → Compare entity embeddings to known competitors
    ↓
Similarity > threshold? → FAIL   |   No match → PASS
```

---

## Implementation

### Setup: Models Required
```python
from transformers import pipeline
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

# NER pipeline — BERT base model
ner_pipeline = pipeline("ner", model="dbmdz/bert-large-cased-finetuned-conll03-english")

# Embedding model for similarity
embedding_model = SentenceTransformer("all-MiniLM-L6-v2")
```

### Stage 1: Exact Match Function
```python
def exact_match(self, text: str) -> list[str]:
    text_lower = text.lower()
    return [c for c in self.competitors if c in text_lower]
```

Simple regex/substring check. If any competitor name is found → fail immediately.

### Stage 2: Named Entity Recognition
```python
def extract_entities(self, text: str) -> list[str]:
    results = self.ner_pipeline(text)
    entities = []
    for entity in results:
        if entity["entity"].startswith("B-") or entity["entity"].startswith("I-"):
            entities.append(entity["word"].strip())
    return entities
```

- **B- marker**: Beginning of a named entity
- **I- marker**: Inside/continuation of a named entity
- Entity categories (person, org, location) are ignored — similarity matching handles relevance

### Stage 3: Vector Similarity Matching
```python
def vector_similarity_match(self, entities: list[str], threshold: float = 0.85) -> list[str]:
    if not entities:
        return []
    entity_embeddings = self.embedding_model.encode(entities)
    matches = []
    for i, entity_emb in enumerate(entity_embeddings):
        similarities = cosine_similarity([entity_emb], self.competitor_embeddings)[0]
        if max(similarities) > threshold:
            matches.append(entities[i])
    return matches
```

Pre-compute competitor embeddings once in `__init__` to avoid redundant computation.

### Full Validator Class
```python
@register_validator(name="competitor_check", data_type="string")
class CompetitorCheckValidator(Validator):
    def __init__(self, competitors: list[str], **kwargs):
        self.competitors = [c.lower() for c in competitors]
        self.embedding_model = SentenceTransformer("all-MiniLM-L6-v2")
        self.ner_pipeline = ner_pipeline  # initialized above
        # Pre-compute competitor embeddings
        self.competitor_embeddings = self.embedding_model.encode(self.competitors)
        super().__init__(**kwargs)

    def validate(self, value: str, metadata: dict) -> ValidationResult:
        # Stage 1: Exact match
        exact_matches = self.exact_match(value)
        if exact_matches:
            return FailResult(error_message=f"Competitor(s) detected: {exact_matches}")

        # Stage 2: NER
        entities = self.extract_entities(value)

        # Stage 3: Vector similarity
        similar_matches = self.vector_similarity_match(entities)
        if similar_matches:
            return FailResult(error_message=f"Potential competitor reference detected: {similar_matches}")

        return PassResult()
```

---

## Deploying via Guardrails Server (Hub Validator)

For production, use the **pre-built competitor check validator from Guardrails Hub** — a state-of-the-art version of this pattern.
```python
guarded_client = OpenAI(
    base_url="http://localhost:<port>/guards/competitor_guard/openai/v1"
)
```

**Result on comparison prompt:** `ValidationError: Pizza by Alfredo detected in output` — competitor mention blocked before reaching the user.

---

## Key Design Decisions

| Decision | Reason |
|----------|--------|
| Exact match first | Fast, zero-cost check for obvious cases |
| NER before similarity | Reduces noise — only compare named entities, not all words |
| Pre-compute competitor embeddings | Avoids re-embedding the same names on every validation call |
| Configurable threshold | Tune sensitivity — lower = stricter, catches more variants; higher = more permissive |
| Run on **output** side | Prevents competitor names from appearing in responses to users |

---

## Graceful Error Handling
```python
try:
    response = guarded_client.chat.completions.create(...)
except ValidationError:
    response = "I can only speak to what makes Alfredo's Pizza Cafe great. How can I help you with your order today?"
```

---

## Course Summary — Failure Modes & Their Guardrails

| Failure Mode | Guardrail Approach | Model/Tool Used |
|---|---|---|
| **Hallucinations** | NLI groundedness check | `guardrails-ai/provenance-v1` (NLI) |
| **Off-topic responses** | Zero-shot topic classification | `facebook/bart-large-mnli` |
| **PII leakage (input)** | Entity detection + blocking | Microsoft Presidio |
| **PII leakage (output)** | Real-time streaming redaction | Microsoft Presidio + Hub validator |
| **Competitor mentions** | Exact match → NER → vector similarity | BERT NER + SentenceTransformers |
| **Forbidden content** | Keyword/substring detection | Simple regex validator |
