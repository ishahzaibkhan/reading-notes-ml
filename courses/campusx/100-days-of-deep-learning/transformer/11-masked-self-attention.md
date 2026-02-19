# Masked Multi-Head Attention in the Transformer Decoder

## Overview
Masked Multi-Head Attention is a critical component of the Transformer Decoder — a modified version of standard self-attention that prevents the model from "seeing the future" during training, while still enabling parallel processing.

---

## Transformer Architecture: Encoder vs. Decoder

The Transformer is divided into two main parts:

- **Encoder**: Fully parallel; processes all input tokens simultaneously via self-attention.
- **Decoder**: More architecturally complex. Shares many components with the Encoder (Multi-head Attention, Positional Encoding, Add & Norm, Feed-Forward) but adds two new components:
  1. **Masked Self-Attention** ← focus of these notes
  2. **Cross-Attention** — queries come from the Decoder, keys and values come from the Encoder output

---

## Core Principle

> **The Transformer Decoder is autoregressive at inference time and non-autoregressive at training time.**

---

## What is an Autoregressive Model?

A model that generates each element in a sequence by conditioning on all **previously generated** elements.

**Example — English to Urdu translation ("How are you" → "آپ کیسے ہیں"):**

| Step | Input | Output |
|------|-------|--------|
| 1 | `<start>` | آپ |
| 2 | آپ | کیسے |
| 3 | کیسے | ہیں |
| 4 | ہیں | `<end>` |

Each word depends on the one before it — you cannot generate the full sequence in a single pass.

---

## Inference Time — Always Sequential

1. Encoder receives the full source sentence and processes it **in parallel** → produces context vectors for each token.
2. Decoder takes `<start>` + encoder output → predicts word 1.
3. Predicted word 1 + encoder output → predicts word 2.
4. Continues until `<end>` token is generated.

**Sequential processing at inference is unavoidable** — the next word is unknown until the previous one is generated.

---

## Training Time — The Problem with Sequential Processing

During training, the **full target sequence is already known**, so we don't need to generate one word at a time.

### Naive Approach: Autoregressive Training with Teacher Forcing

Instead of feeding the model's (possibly wrong) predicted word back as input, we feed the **correct word from the dataset** at each step. This is called **Teacher Forcing**.

**Example — Training on "How are you" → "آپ کیسے ہیں":**

| Step | Input to Decoder | Prediction | Correct? |
|------|-----------------|------------|----------|
| 1 | `<start>` | تم (wrong) | ✗ |
| 2 | آپ (correct, teacher forced) | کیسے | ✓ |
| 3 | کیسے (correct, teacher forced) | تھے (wrong) | ✗ |
| 4 | ہیں (correct, teacher forced) | `<end>` | ✓ |

Loss is calculated across all predictions → backpropagation updates weights.

### Why This Is Extremely Slow

- A sentence of **N words** requires **N+1 sequential decoder passes**.
- A paragraph of **300 words** → **301 sequential passes**.
- Multiplied across a large dataset → **training becomes impractically slow**.

### The Key Insight

Since we already have the entire target sequence during training, we can feed **all tokens at once** in parallel — but this creates a new problem.

---

## The Data Leakage Problem (Why Naive Parallelism Fails)

If the full target sequence "آپ کیسے ہیں" is fed to the self-attention block all at once:

- **"آپ"** attends to "کیسے" and "ہیں" → sees future words ❌
- **"کیسے"** attends to "ہیں" → sees a future word ❌

The model learns to **cheat** by using information it would never have access to at inference time. This breaks the autoregressive property and produces a model that cannot generate sequences correctly in the real world.

---

## The Solution: Masking

Masking forces each token to only attend to **itself and all preceding tokens**, even when all tokens are processed in parallel.

### Self-Attention Mathematics (Quick Recap)

Given input embeddings, the attention output is computed as:
```
Attention(Q, K, V) = Softmax( Q·Kᵀ / √d_k ) · V
```

Where:
- **Q** (Query), **K** (Key), **V** (Value) = input embeddings × learned weight matrices W_Q, W_K, W_V
- **d_k** = dimension of the key vectors (used for scaling to stabilize gradients)
- The result is a **contextual embedding** for each token

### Where the Mask Is Applied

After computing the scaled scores `Q·Kᵀ / √d_k`, a **mask matrix** of the same dimensions is added before the Softmax.

### Structure of the Mask Matrix

For a 3-token sequence (آپ, کیسے, ہیں):
```
         آپ    کیسے   ہیں
آپ   [    0     -∞     -∞  ]
کیسے [    0      0     -∞  ]
ہیں  [    0      0      0  ]
```

- **0** → position is allowed (current or past token)
- **−∞** → position is blocked (future token)

### Why −∞ Works

When the mask is added to the scaled attention scores:
```
masked_score = (Q·Kᵀ / √d_k) + mask
```

Positions with −∞ become −∞. When Softmax is applied:
```
Softmax(−∞) = 0
```

So the attention weight for any future token becomes exactly **0** — the model receives no information from those positions.

### Resulting Attention Pattern

| Token | Can Attend To |
|-------|--------------|
| آپ | آپ only |
| کیسے | آپ, کیسے |
| ہیں | آپ, کیسے, ہیں |

This perfectly mirrors what would be available at inference time, one step at a time.

---

## Best of Both Worlds

| | Inference | Training |
|---|---|---|
| **Processing** | Sequential (unavoidable) | Parallel (via masking) |
| **Data leakage** | N/A | Prevented by mask |
| **Speed** | Inherently slow | Much faster |
| **Autoregressive property** | ✓ | ✓ (enforced by mask) |

Masking allows the Transformer Decoder to:
- **Train fast** by processing all tokens in parallel
- **Behave correctly** at inference by never having learned to cheat on future tokens

---

## Summary

1. The Decoder must be autoregressive — each output token depends on previous ones.
2. At inference, this is sequential and unavoidable.
3. At training, sequential processing is unnecessarily slow since the full target is known.
4. Feeding all tokens in parallel without masking causes data leakage (model sees the future).
5. The **mask matrix** (filled with −∞ at future positions) is added after scaled dot-product attention and before Softmax, zeroing out all future attention weights.
6. This gives parallel training speed **with** autoregressive correctness — the core innovation of Masked Multi-Head Attention.
