# Building a Hallucination Detection Validator
### Using Natural Language Inference (NLI) for Groundedness Checking

---

## Why Hallucination Detection Matters

Real-world consequences of LLM hallucinations include legal liability (e.g., lawyers citing fake cases), political lawsuits, and enterprise fines. In the pizza chatbot context, the model fabricated a full recipe (ingredients + cooking steps) that simply didn't exist in the knowledge base — despite RAG and prompt engineering being in place.

---

## Core Concept: Groundedness via NLI Entailment

**Hallucination in RAG context** = LLM output that is not faithful to the retrieved source documents.

**Natural Language Inference (NLI)** checks faithfulness by classifying the relationship between:

| Term | Definition |
|------|-----------|
| **Premise** | Trusted source text (your vector DB documents) |
| **Hypothesis** | Statement to be verified (LLM-generated sentence) |

The NLI model (a classifier) predicts one of three labels:

| Label | Meaning |
|-------|---------|
| **Entailment** | Hypothesis is faithful/truthful given the premise ✅ |
| **Contradiction** | Hypothesis contradicts the premise — hallucinated ❌ |
| **Neutral** | Hypothesis is unrelated to the premise — treated as not grounded ❌ |

---

## Model Setup
```python
from transformers import pipeline
import nltk
from sentence_transformers import SentenceTransformer

# NLI model from GuardrailsAI HuggingFace org
nli_pipeline = pipeline("text-classification", model="guardrails-ai/provenance-v1")

# Embedding model for semantic similarity
embedding_model = SentenceTransformer("all-MiniLM-L6-v2")
```

- **NLI Model**: `guardrails-ai/provenance-v1` — fine-tuned for groundedness detection
- **Embedding Model**: `all-MiniLM-L6-v2` — small but high-performing on leaderboards

> **Important**: Use the **same embedding model** for both source documents and LLM-generated sentences when computing similarity.

---

## Hallucination Validator Architecture

High-level flow:
```
LLM Output
    ↓
[Split into sentences]
    ↓
[For each sentence → find top-5 most relevant source chunks via cosine similarity]
    ↓
[Run NLI: source chunks = premise, sentence = hypothesis]
    ↓
Entailed → grounded ✅   |   Contradiction/Neutral → hallucinated ❌
    ↓
If any hallucinated sentences → FailResult
Else → PassResult
```

---

## Implementation: Step by Step

### Step 1: Sentence Splitter
```python
def sentence_splitter(self, text: str) -> list[str]:
    return nltk.sent_tokenize(text)
```

Splits LLM output into individual sentences for granular checking.

### Step 2: Find Relevant Sources
```python
def find_relevant_sources(self, sentences: list[str], sources: list[str]) -> dict:
    sentence_embeddings = self.embedding_model.encode(sentences)
    source_embeddings = self.embedding_model.encode(sources)

    relevant_sources = {}
    for i, sentence_emb in enumerate(sentence_embeddings):
        cosine_scores = cosine_similarity([sentence_emb], source_embeddings)[0]
        top_5_indices = cosine_scores.argsort()[-5:][::-1]
        relevant_sources[i] = [sources[j] for j in top_5_indices]
    return relevant_sources
```

For each sentence, retrieves the **top 5 most semantically similar** source chunks.

### Step 3: Check Entailment
```python
def check_entailment(self, sentence: str, sources: list[str]) -> bool:
    for source in sources:
        result = self.nli_pipeline(f"{source} [SEP] {sentence}")
        if result[0]["label"] == "entailment":
            return True
    return False
```

Returns `True` if **any** of the top sources entail the sentence; `False` if all are contradictory or neutral.

### Step 4: Full Validate Function
```python
def validate(self, value: str, metadata: dict) -> ValidationResult:
    sentences = self.sentence_splitter(value)
    relevant_sources = self.find_relevant_sources(sentences, self.sources)

    hallucinated = []
    entailed = []

    for i, sentence in enumerate(sentences):
        if not self.check_entailment(sentence, relevant_sources[i]):
            hallucinated.append(sentence)
        else:
            entailed.append(sentence)

    if hallucinated:
        return FailResult(error_message=f"Hallucinated sentences: {hallucinated}")
    return PassResult()
```

### Step 5: Class Initialization
```python
@register_validator(name="hallucination_detector", data_type="string")
class HallucinationDetector(Validator):
    def __init__(self, sources: list[str], entailment_model, **kwargs):
        self.embedding_model = SentenceTransformer("all-MiniLM-L6-v2")
        self.sources = sources
        self.nli_pipeline = pipeline("text-classification", model=entailment_model)
        super().__init__(**kwargs)
```

---

## NLI Model Quick Demo

| Premise | Hypothesis | Prediction | Score |
|---------|-----------|------------|-------|
| "Sun rises in the East, sets in the West" | "The sun rises in the East" | **Entailment** | 0.869 |
| "Sun rises in the East, sets in the West" | "The sun rises in the West" | **Contradiction** | 0.864 |

---

## Testing the Validator
```python
hallucination_validator = HallucinationDetector(
    sources=my_source_documents,
    entailment_model="guardrails-ai/provenance-v1"
)

result = hallucination_validator.validate(llm_output, metadata={})
print(result)  # FailResult if hallucinated, PassResult if grounded
```

---

## Key Design Decisions

- **Sentence-level granularity** — validates each sentence individually, not the whole response as one block
- **Top-5 source retrieval** — only checks against the most semantically relevant sources, not all documents
- **Neutral = not grounded** — only `entailment` is treated as safe; neutral and contradiction both trigger failure
- **Same embedding model** for both sources and sentences ensures comparable vector space
