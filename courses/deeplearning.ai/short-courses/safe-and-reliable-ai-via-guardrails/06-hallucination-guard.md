# Wrapping the Hallucination Validator in a Guard
### Deploying the NLI-Based Hallucination Detector in a Production Chatbot

---

## Validator vs. Guard — When to Use Which

| Use Case | Approach |
|----------|----------|
| Testing / debugging a single validator | Use **validator directly** — less abstraction, raw output |
| Production application | Use **guard** — more features, better integration |

---

## Benefits of Wrapping a Validator in a Guard

| Benefit | Description |
|---------|-------------|
| **Combine multiple guardrails** | Run hallucination detection + profanity check + PII detection all in one step |
| **Streaming support** | Validate LLM output in real-time as chunks stream to users — fast AND protected |
| **OpenAI-compatible endpoints** | Single line of code change to swap bare LLM for a guarded one |
| **Out-of-the-box logging** | Every guard execution is logged and analyzable for monitoring application performance |
| **Error handling** | Built-in structured exception handling |

---

## Setting Up the Hallucination Guard
```python
hallucination_guard = Guard().use(
    HallucinationDetector(
        embedding_model="all-MiniLM-L6-v2",
        entailment_model="guardrails-ai/provenance-v1",
        on_fail=OnFailAction.EXCEPTION
    )
)
```

---

## Important Nuance: Factual ≠ Grounded

A critical distinction demonstrated with a toy example:

| Statement | Sources | Factually True? | Grounded? | Result |
|-----------|---------|----------------|-----------|--------|
| "The sun rises in the East" | "Sun rises in East, sets in West" | ✅ | ✅ | **Pass** |
| "The sun is a star" | "Sun rises in East, sets in West; The sun is hot" | ✅ | ❌ | **Fail** |

> The guardrail does **not** check real-world truth — it checks **faithfulness to provided sources**. A factually correct statement will still fail if it isn't supported by the documents in your vector DB.

This is by design: in a RAG system, you want the LLM to only assert things your trusted documents can back up.

---

## Integrating with the Chatbot via Guardrails Server

Same one-line swap as before — replace the standard OpenAI client with a guarded client pointing to the Guardrails Server:
```python
guarded_client = OpenAI(
    base_url="http://localhost:<port>/guards/hallucination_guard/openai/v1"
)
```

Then initialize the chatbot identically, substituting `guarded_client` for `client`.

---

## Result on the Pizza Chatbot

**Before guard:** Chatbot generates a full fabricated pizza recipe with detailed cooking instructions.

**After guard:** Detailed `ValidationError` is raised, identifying the hallucinated sentences about pizza-making instructions.

---

## Handling Failures Gracefully

Instead of surfacing raw validation errors to users, catch exceptions and return friendly messages:
```python
try:
    response = guarded_client.chat.completions.create(...)
except ValidationError:
    response = "I'm sorry, I can't provide that information as it's not in our records."
```

---

## Full Pattern Summary
```
1. Build Validator   →   Custom class subclassing Validator
2. Wrap in Guard     →   Guard().use(MyValidator(...))
3. Serve via Server  →   Configure guardrails server with the guard
4. Swap Client       →   Point OpenAI client to guarded server endpoint
5. Handle Errors     →   Catch exceptions for graceful UX
```
