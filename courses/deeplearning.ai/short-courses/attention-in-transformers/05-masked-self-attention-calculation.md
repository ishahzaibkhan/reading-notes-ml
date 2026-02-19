# Masked Self-Attention: Matrix Math

## Overview

Masked self-attention is nearly identical to standard self-attention, with **one key difference**: a mask matrix **M** is added to the scaled similarities.

---

## Step-by-Step Calculation

### 1. Input Preparation
- Start with the input prompt (e.g., *"write a poem"*)
- Create **word embeddings** for each token
- Add **positional encoding** to produce encoded values

### 2. Compute Q, K, V Matrices
- Calculate **Query (Q)**, **Key (K)**, and **Value (V)** matrices exactly as in standard self-attention

### 3. Calculate Scaled Similarities
- Compute dot product of Q and K to get similarity scores
- Scale the similarities (divide by √d_k)

### 4. Apply the Mask Matrix M
Add mask matrix **M** to the scaled similarities, where:
- **0** is added to positions we want to **include** (leaves values unchanged)
- **−∞** is added to positions we want to **exclude** (effectively zeroes them out after softmax)

#### Mask Structure (lower-triangular)
| | write | a | poem |
|---|---|---|---|
| **write** | 0 | −∞ | −∞ |
| **a** | 0 | 0 | −∞ |
| **poem** | 0 | 0 | 0 |

#### Why this works:
- Adding **0** → scaled similarity unchanged
- Adding **−∞** → after softmax, e^(−∞) = 0, so those positions get **0% attention weight**

### 5. Apply Softmax (Row-wise)
After masking, softmax is applied to each row:

| Token | write | a | poem |
|---|---|---|---|
| **write** | 100% | 0% | 0% |
| **a** | ~% | ~% | 0% |
| **poem** | ~% | ~% | ~% |

- **"write"** attends only to itself
- **"a"** attends to "write" and itself
- **"poem"** attends to all tokens

### 6. Multiply by Value Matrix V
The final masked self-attention output for each token:
- **"write"** → influenced only by itself
- **"a"** → influenced by "write" and "a"
- **"poem"** → influenced by all tokens ("write", "a", "poem")

---

## Purpose of Masking

Masking enforces **causality** — each token can only attend to itself and tokens that came **before** it. This prevents the model from "cheating" by looking at future tokens during training or generation.

---

## Summary Formula

$$\text{MaskedAttention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}} + M\right)V$$
