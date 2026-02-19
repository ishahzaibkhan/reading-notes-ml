# Encoder-Decoder Attention (Cross-Attention)

## Context: Three Types of Transformers

### 1. Encoder-Only Transformers
- Use **self-attention** to create **context-aware embeddings**
- Applications: semantic clustering, document similarity, classification model inputs

### 2. Decoder-Only Transformers
- Use **masked self-attention** to generate new tokens autoregressively

### 3. Encoder-Decoder Transformers (the original)
- The **first transformer ever built**
- Contains both an encoder and a decoder connected together
- Enables a third attention mechanism: **encoder-decoder attention**

---

## Encoder-Decoder Attention (Cross-Attention)

### How Q, K, V are sourced — the key difference:
| Matrix | Source |
|---|---|
| **Keys (K)** | Output from the **encoder** |
| **Values (V)** | Output from the **encoder** |
| **Queries (Q)** | Output from the **decoder's masked self-attention** |

### Calculation
Once Q, K, V are assembled, attention is computed **exactly like standard self-attention** — using all similarities (no masking).

---

## Original Use Case: Seq2Seq Translation

The original encoder-decoder transformer was designed for **sequence-to-sequence (seq2seq)** tasks, primarily **machine translation**.

**Example flow:**
1. Input: *"Pizza is great."* (English)
2. **Encoder** computes self-attention on the input
3. **Decoder** uses encoder output → computes encoder-decoder attention
4. Output: *"¡La pizza es genial!"* (Spanish)

---

## Where the Names Come From

- Researchers found the **encoder alone** was powerful for representation tasks → **encoder-only transformers**
- Researchers found the **decoder alone** could generate text, including translations → **decoder-only transformers**
- The original full architecture is now less common for pure language modeling

---

## Modern Application: Multimodal Models

Encoder-decoder attention still thrives in **multimodal models**, where different data types need to be bridged:

| Component | Role |
|---|---|
| **Encoder** | Trained on images or audio → produces context-aware embeddings |
| **Decoder** | Text-based; receives encoder output via cross-attention |
| **Output** | Image captions, audio prompt responses, etc. |

---

## Key Takeaway

> Encoder-decoder attention (cross-attention) simply requires **flexibility in how Q, K, and V are sourced** — queries from one stream, keys and values from another.
