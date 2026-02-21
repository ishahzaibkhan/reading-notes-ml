# Safe and Reliable AI via Guardrails
### Built in partnership with GuardrailsAI

---

## What Are Guardrails?

Guardrails are **safety mechanisms and validation tools** built into AI applications that use LLMs. They ensure at runtime that the application:
- Follows specific rules
- Operates within predefined boundaries
- Produces outputs aligned with developer expectations

They provide a **critical layer of control and oversight**, supporting safe and responsible AI development.

---

## The Core Problem with LLMs

Despite advances in prompting, fine-tuning, RLHF, and RAG, LLM outputs remain **hard to predict**. This causes challenges in:
- Industries with **strict regulatory requirements** (healthcare, government, finance)
- Applications demanding **high consistency**
- Moving beyond proof-of-concept to **production-ready** systems

> Techniques like RLHF and RAG alone are insufficient to meet stringent reliability and compliance standards.

---

## How Guardrails Solve This

Guardrails are additional components that **check inputs/outputs** of LLMs to ensure they conform to rules, preventing:
- Incorrect or irrelevant information
- Sensitive/PII data leakage
- Hallucinations
- Off-topic responses

---

## The Validator — Core Component

A **validator** is a function that:
1. Takes as input a **user prompt** and/or **LLM response**
2. Checks that it conforms to a **predefined rule**
3. Takes action (e.g., throws exception) if the rule is violated

### Types of Validators

| Type | Method | Example Use Case |
|------|--------|-----------------|
| Simple | Regex expressions | Detect PII (phone numbers, emails) |
| Intermediate | ML models (Transformers) | Stay on topic, block specific words |
| Advanced | Other LLMs or NLI models | Hallucination detection |

### Specific Validator Examples
- **PII Validator** — Regex checks for emails/phone numbers; throws exception if found
- **Topic Guard** — Checks responses against a list of allowed discussion subjects
- **Word Blocklist** — Prevents trademarked terms or competitor names from appearing
- **Hallucination Validator** — Uses a **Natural Language Inference (NLI)** model to verify that LLM answers are grounded in retrieved source text (for RAG systems)

---

## Key Advantages of Guardrails

- **Flexible** — Can use many smaller ML models for validation tasks
- **Performant** — Smaller models keep applications fast
- **More reliable** than using LLMs alone for certain failure modes
- **Pre-built options** available via the **Guardrails Hub**

---

## What You'll Learn in This Course

1. Build guardrails from scratch to mitigate common LLM failure modes
2. Implement individual guardrails for hallucinations, PII leakage, and more
3. Use the core **validator coding pattern** used at GuardrailsAI
4. Access and use pre-built guardrails from the **Guardrails Hub**
5. Customize guardrails for your specific product or use case
