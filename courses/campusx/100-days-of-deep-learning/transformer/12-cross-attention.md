# Cross Attention in Transformer Architecture

## Overview
Cross Attention is a key mechanism in the **Transformer decoder** that enables the model to relate an output sequence to an input sequence. It is the **second block in the decoder architecture**.

> **Defining feature:** In a standard Multi-Head Attention block, all three vectors (Q, K, V) come from the same source. In Cross Attention, **K and V come from the encoder**, while **Q comes from the decoder**.

---

## Transformer Architecture — Context

### Encoder Components (covered prior)
- Embeddings → Positional Encoding → Self-Attention → Multi-Head Attention → Normalization

### Decoder Components
- **1st block:** Masked Multi-Head Self-Attention
- **2nd block:** Cross Attention ← *focus of these notes*
- **3rd block:** Feed Forward Network

---

## What is Cross Attention?

**Definition:** A mechanism in Transformer architectures for sequence-to-sequence tasks that allows the model to focus on relevant parts of the **input sequence** while generating each token of the **output sequence**.

### Intuition via Machine Translation
**Example:** *"I like eating ice cream"* → *"مجھے آئس کریم کھانا پسند ہے"*

When generating the word **"کھانا"**, the decoder depends on two things:

| Factor | Mechanism |
|---|---|
| What has been generated so far in the output | **Self-Attention** |
| Which input words are relevant to the current output word | **Cross Attention** |

**Cross Attention captures inter-sequence relationships:**
- "آئس کریم" ↔ "ice cream" (high similarity)
- "پسند" ↔ "like" (high similarity)
- "کھانا" ↔ "eating" (high similarity)

---

## Self-Attention vs. Cross Attention

### 1. Input Difference

| | Self-Attention | Cross Attention |
|---|---|---|
| Takes as input | **One** sequence | **Two** sequences |
| Example | "We are friends" | "We are friends" + "ہم دوست ہیں" |
| Embeddings used | From the single sequence | From **both** input and output sequences |

---

### 2. Processing Difference (Q, K, V Generation)

#### Self-Attention
1. Generate embeddings for each token in the sequence
2. Multiply each embedding by **Wq, Wk, Wv** to get Q, K, V — all from the **same sequence**
3. Attention score = dot product of each token's **Q** with every other token's **K** → produces an attention score matrix (intra-sequence similarity)
4. Output = weighted sum of **V** vectors using those scores → contextual embeddings

#### Cross Attention
1. Generate embeddings for **both** sequences
2. **Q vectors** → derived from the **output sequence** (e.g., Urdu words) using **Wq**
3. **K and V vectors** → derived from the **input sequence** (e.g., English words) using **Wk** and **Wv**
4. Attention score = dot product of **Q (output)** × **K (input)** → inter-sequence similarity matrix
5. Output = weighted sum of **V (input)** using those attention scores → contextual embeddings for output tokens

> In the Transformer diagram: **Q enters from the bottom** (decoder), **K and V enter from the top** (encoder output).

---

### 3. Output Difference

| | Self-Attention | Cross Attention |
|---|---|---|
| Number of outputs | One contextual embedding per **input** token | One contextual embedding per **output** token |
| Influenced by | All tokens in the **same** sequence | All tokens in the **input** sequence |
| Example | CE("We") = weighted blend of "We", "are", "friends" | CE("ہم") = 50% "We" + 30% "are" + 20% "friends" |

**Key insight:** Cross Attention produces contextual embeddings for the **output sequence**, but each embedding's value is determined by the **similarity to input sequence tokens** — bridging the two sequences.

---

## Historical Connection — Bahdanau & Luong Attention

Cross Attention is conceptually rooted in older **RNN-based attention mechanisms**:

| Mechanism | Architecture | Similarity Calculation |
|---|---|---|
| Bahdanau Attention | RNN encoder-decoder | Neural network |
| Luong Attention | RNN encoder-decoder | Dot product |
| Cross Attention (Transformer) | Transformer encoder-decoder | Scaled dot product |

**Shared concept across all three:** To generate the next output token, compute similarity between the current decoder state and **all encoder hidden states**, then use a weighted sum to pull in relevant input information.

The original Transformer paper (*"Attention Is All You Need"*) explicitly acknowledges this inspiration, calling Cross Attention **"encoder-decoder attention"** and noting:
- Queries come from the **previous decoder layer**
- Keys and Values come from the **encoder output**
- This allows the decoder to attend to **all positions** in the input sequence

---

## Cross Attention — Step-by-Step Summary
```
Input Sequence  →  Encoder  →  K, V vectors
                                     ↓
Output Sequence →  Decoder  →  Q vectors
                                     ↓
                     Attention Score = Q · Kᵀ
                                     ↓
                     Weights = Softmax(scores)
                                     ↓
                     Output = Weights · V
                     (Contextual embeddings for output sequence)
```

---

## Use Cases of Cross Attention

Cross Attention applies whenever there are **two distinct sequences** — an input and an output, even across different modalities:

| Task | Input | Output |
|---|---|---|
| Machine Translation | Text (language A) | Text (language B) |
| Question Answering | Context/Question | Answer |
| Image Captioning | Image features | Text description |
| Text-to-Image Generation | Text prompt | Image |
| Text-to-Speech | Text | Speech/Audio |

> Cross Attention is a foundational concept in **LLMs** and **Generative AI**, critical to understanding modern deep learning systems.
