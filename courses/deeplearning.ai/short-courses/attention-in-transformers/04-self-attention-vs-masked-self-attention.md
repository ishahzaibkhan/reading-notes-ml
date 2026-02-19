```markdown
# Self-Attention vs Masked Self-Attention in Transformers

## Word Embeddings

### The Problem with Random Number Assignment
- Simple approach: assign each word a random number
- **Weakness:** Similar words (e.g., "great" vs "awesome") get unrelated numbers, forcing the network to learn them independently — increasing complexity and training requirements

### What Good Word Embeddings Should Do
- **Similar words → similar numbers** so learning one helps learn the other
- **Multiple numbers per word** to capture different contexts (e.g., "great" used positively vs. sarcastically)

### How Word Embedding Networks Work
1. Create one input node per unique word
2. Connect all inputs to **activation functions** (the number of activation functions = number of embedding dimensions)
3. Weights on these connections = the word embedding values (initialized randomly)
4. Train the network to **predict the next word** from the current word(s)

**Result after training:** Similar words used in similar contexts cluster together in embedding space (e.g., "great" and "awesome" end up with similar embeddings).

### Adding More Context
- Using multiple preceding words (e.g., "the pizza came out" → predicts "of") creates richer embeddings than single-word prediction
- **Problem:** This approach ignores word order — "the pizza came out of" and "pizza out came the of" produce identical inputs/outputs
- Word order is critical: *"Squatch eats pizza"* vs *"Pizza eats Squatch"* — same words, opposite meanings

---

## Positional Encoding & Attention in Transformers

- **Positional Encoding Layer:** Incorporates word order into embeddings
- **Attention Layer:** Establishes relationships among words
- Together they create **Context-Aware (Contextualized) Embeddings**

### Context-Aware Embeddings vs Word Embeddings
| | Word Embeddings | Context-Aware Embeddings |
|---|---|---|
| Clusters | Individual words | Sentences & documents |
| Context sensitivity | No | Yes |

---

## Types of Transformers

### Encoder-Only Transformers (use Self-Attention)
- **Self-attention** looks at words **before AND after** the word of interest
- Produces **Context-Aware Embeddings**
- **Use cases:**
  - Sentence/document clustering
  - Sentiment classification (e.g., positive/negative tweets about pizza)
  - Input features for logistic regression or neural network classifiers

### Decoder-Only Transformers (use Masked Self-Attention)
- **Masked self-attention** looks **only at preceding words** — future words are hidden/masked
- Example: When processing "the", only "the" itself is used; when processing "it", only words up to "it" are used
- **Training:** Given the start of a sentence, weights are adjusted until the model generates the correct continuation
- Produces **generative outputs** — text that follows a prompt
- **Example:** ChatGPT is a decoder-only transformer → called a *generative model*

---

## Key Difference Summary

| | Self-Attention | Masked Self-Attention |
|---|---|---|
| Looks at | Words before **and** after | Words before only |
| Transformer type | Encoder-Only | Decoder-Only |
| Output | Context-Aware Embeddings | Generated text/tokens |
| Best for | Classification, clustering | Text generation |
```
