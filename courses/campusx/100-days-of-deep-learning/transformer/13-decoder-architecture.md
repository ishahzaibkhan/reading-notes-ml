# Transformer Decoder Architecture — Training Phase

## Overview

A Transformer consists of two main components:
- **Encoder** — encodes the input sequence
- **Decoder** — decodes the encoded input and generates the output sequence

Both the Encoder and Decoder are stacks of **6 identical blocks** (same architecture, different learned parameters).

---

## Encoder Block (Quick Recap)
Each of the 6 Encoder blocks contains:
1. Self-Attention
2. Feed Forward Neural Network

The output of each block feeds into the next.

---

## Decoder Block Structure
Each of the 6 Decoder blocks contains:
1. **Masked Multi-Head Self-Attention**
2. **Cross-Attention** (Encoder-Decoder Attention)
3. **Feed Forward Neural Network**

Each of these is followed by an **Add & Norm** layer.

---

## Example Used
**Machine Translation:** English → Urdu
`"We are friends"` → `"ہم دوست ہیں"`

The Encoder has already processed the English sentence and produced contextual embeddings.

---

## Phase 1 — Input Preparation for Decoder

### 1. Shift Right (Add Start Token)
- A `<SOS>` (Start of Sentence) token is prepended to the target sentence.
- `"ہم دوست ہیں"` → `"<SOS> ہم دوست ہیں"`

### 2. Tokenization
- The sentence is split into individual tokens.
- `[<SOS>, ہم, دوست, ہیں]`

### 3. Embedding
- Each token is converted into a **512-dimensional vector**.
- `<SOS>` → e₁, `ہم` → e₂, `دوست` → e₃, `ہیں` → e₄

### 4. Positional Encoding
- A unique **512-dimensional positional vector** is generated for each position.
- Position 1 → p₁, Position 2 → p₂, etc.
- Added to embeddings: `e₁ + p₁ = x₁`, `e₂ + p₂ = x₂`, etc.
- Final input vectors: **x₁, x₂, x₃, x₄**

---

## Phase 2 — Operations Inside Each Decoder Block

### 2.1 Masked Multi-Head Self-Attention
- Processes input vectors (x₁, x₂, x₃, x₄).
- **Masking Rule:** When computing the contextual embedding for a token, only that token and all **previous** tokens are visible. Future tokens are masked out.

| Token | Can Attend To |
|-------|--------------|
| x₁ (`<SOS>`) | x₁ only |
| x₂ (`ہم`) | x₁, x₂ |
| x₃ (`دوست`) | x₁, x₂, x₃ |
| x₄ (`ہیں`) | x₁, x₂, x₃, x₄ |

- Output: contextual vectors **z₁, z₂, z₃, z₄**

### 2.2 Add & Norm (After Masked Attention)
- **Residual Connection:** `zᵢ + xᵢ = zᵢ'`
- **Layer Normalization:** Stabilizes training by keeping values in a small range.
- Output: normalized vectors **z₁ₙ, z₂ₙ, z₃ₙ, z₄ₙ**

---

### 2.3 Cross-Attention (Encoder-Decoder Attention)
- The key step where the **source (English)** and **target (Urdu)** sequences interact.
- **Two inputs:**
  - Decoder's previous output (z₁ₙ–z₄ₙ) → provides **Query (Q)** vectors
  - Encoder's final output embeddings → provides **Key (K)** and **Value (V)** vectors
- Computes similarity scores between each Urdu token and all English tokens.
- Output: cross-contextual vectors **zc₁, zc₂, zc₃, zc₄**

### 2.4 Add & Norm (After Cross-Attention)
- **Residual Connection:** `zcᵢ + zᵢₙ = zcᵢ'`
- **Layer Normalization** applied.
- Output: normalized vectors **zc₁ₙ, zc₂ₙ, zc₃ₙ, zc₄ₙ**

---

### 2.5 Feed Forward Neural Network
- Identical architecture to the Encoder's FFN.
- **Layer 1:** 2048 neurons, ReLU activation | Weights: 512 × 2048, Biases: 2048
- **Layer 2:** 512 neurons, Linear activation | Weights: 2048 × 512, Biases: 512
- Input processed as a batch matrix: **4 × 512**
- Purpose: introduces non-linearity and further refines contextual representations.
- Output: vectors **y₁, y₂, y₃, y₄** (still 512-dimensional)

### 2.6 Add & Norm (After FFN)
- **Residual Connection:** `yᵢ + zcᵢₙ = yᵢ'`
- **Layer Normalization** applied.
- Output: **y₁ₙ, y₂ₙ, y₃ₙ, y₄ₙ** — these are the final outputs of one Decoder block.

---

## Phase 3 — Stacking Decoder Blocks

- Output of Block 1 (y₁ₙ–y₄ₙ) becomes the input to Block 2.
- This repeats across all **6 Decoder blocks**.
- Each block has **unique parameters** but the same architecture.
- Final output from Block 6: **yf₁ₙ, yf₂ₙ, yf₃ₙ, yf₄ₙ** (f = final)

---

## Phase 4 — Output Block

### 4.1 Linear Layer
- Input: 512-dimensional final vectors (batch: 4 × 512)
- **Neurons:** equal to vocabulary size **v** (e.g., 10,000 unique Urdu words)
- Each neuron corresponds to one unique word in the target vocabulary.
- Weights: 512 × v | Biases: v
- Output: unnormalized scores (**logits**) — a vector of shape 1 × v per token

### 4.2 Softmax Layer
- Converts logits into a **probability distribution** over the vocabulary.
- Sum of all probabilities = 1 for each token position.
- The word with the **highest probability** is selected as the predicted output word.

| Input Vector | Represents | Predicts |
|-------------|-----------|---------|
| yf₁ₙ | `<SOS>` | ہم (1st word) |
| yf₂ₙ | ہم | دوست (2nd word) |
| yf₃ₙ | دوست | ہیں (3rd word) |
| yf₄ₙ | ہیں | `<EOS>` (end of sentence) |

- Generation stops when `<EOS>` (End of Sentence) token is predicted.

---

## Complete Decoder Flow Summary
```
Input Sentence (Urdu, shifted) 
        ↓
[Shift Right → Tokenization → Embedding → Positional Encoding]
        ↓
     x₁, x₂, x₃, x₄
        ↓
┌─────────────────────────────────┐
│        Decoder Block × 6        │
│  1. Masked Multi-Head Attention │
│  2. Add & Norm                  │
│  3. Cross-Attention  ←── Encoder Output (K, V)
│  4. Add & Norm                  │
│  5. Feed Forward Network        │
│  6. Add & Norm                  │
└─────────────────────────────────┘
        ↓
   yf₁ₙ, yf₂ₙ, yf₃ₙ, yf₄ₙ
        ↓
   Linear Layer (512 → v)
        ↓
   Softmax → Probability Distribution
        ↓
   Predicted Output Words
```
