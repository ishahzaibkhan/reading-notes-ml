# Why Self-Attention is Called "Self"

## Core Question
Why is Self-Attention called "**Self**" Attention? (Common interview question)

## Understanding "Attention" in Self-Attention

### Traditional Attention Mechanism: Encoder-Decoder Architecture

#### Problem: Language Translation (e.g., English to Hindi)

**Architecture Components**:
- **Encoder**: Processes input sentence word-by-word
  - Maintains hidden states: h₀, h₁, h₂, h₃, h₄
  - Final hidden state (h₄) = Context Vector

**Context Vector Issue**:
- Single vector must summarize entire input sentence
- For sentences >30 words: information loss, degraded translation quality

#### Solution: Attention Mechanism

**Key Innovation**: Send different context vectors to decoder at each time step

**Context Vector Calculation** (e.g., C₁ for first output word):
```
C₁ = α₁₁h₁ + α₁₂h₂ + α₁₃h₃ + α₁₄h₄
```

**Where**:
- αᵢⱼ = attention weights (indicate which input word is useful for current output)
- hⱼ = encoder hidden states

### Luong Attention Equations

**Three-Step Process**:

1. **Calculate Alignment Scores** (eᵢⱼ):
```
   eᵢⱼ = Sᵢ · Hⱼ
```
   - Sᵢ = decoder hidden state at current time step
   - Hⱼ = encoder hidden state from input sequence
   - Dot product measures similarity

2. **Calculate Attention Weights** (αᵢⱼ):
```
   αᵢⱼ = Softmax(eᵢⱼ)
```

3. **Calculate Context Vector** (C):
```
   C = Σ(αᵢⱼ × Hⱼ)
```
   - Weighted sum of encoder hidden states

## Self-Attention vs Luong Attention: The Similarity

### Self-Attention Setup
- **Single sentence**: "Turn off the Lights"
- **No separate encoder/decoder**
- **Goal**: Create contextual embeddings for each word

### Process Comparison

#### Self-Attention Vectors
For every word, create three vectors:
- **Query (Q)**: What information am I looking for?
- **Key (K)**: What information do I contain?
- **Value (V)**: The actual information to pass forward

#### Generating Contextual Embedding (Example: "Turn")

**Formula**:
```
Y_turn = w₁₁ × V_turn + w₁₂ × V_off + w₁₃ × V_the + w₁₄ × V_lights
```

**Steps**:

1. **Calculate Similarity Scores** (s):
```
   s₁₁ = Q_turn · K_turn
   s₁₂ = Q_turn · K_off
   s₁₃ = Q_turn · K_the
   s₁₄ = Q_turn · K_lights
```

2. **Calculate Weights** (w):
```
   w = Softmax(s)
```

3. **Weighted Sum of Values**:
```
   Y_turn = Σ(wᵢ × Vᵢ)
```

### Direct Mapping: Luong ↔ Self-Attention

| Luong Attention | Self-Attention | Role |
|----------------|----------------|------|
| Decoder Hidden States (Sᵢ) | Query vectors (Q) | "Querying" for relevant information |
| Encoder Hidden States (Hⱼ) | Key vectors (K) | Providing similarity information |
| Encoder Hidden States (Hⱼ) | Value vectors (V) | Actual information content |

### Mathematical Equivalence

Both mechanisms share the same three core operations:
1. **Dot product** → Calculate alignment/similarity scores
2. **Softmax** → Convert scores to attention weights
3. **Weighted sum** → Generate context vector/embedding

**Conclusion**: Self-Attention **IS** an attention mechanism because it follows the same mathematical formulation.

## Why "Self"? Inter-Sequence vs Intra-Sequence

### Luong/Bahdanau Attention (Traditional)
- **Type**: Inter-Sequence Attention
- **Operation**: Calculate attention **between** two different sequences
- **Example**: English input sequence ↔ Hindi output sequence
- Different source and target sequences

### Self-Attention
- **Type**: Intra-Sequence Attention
- **Operation**: Calculate attention **within** the same sequence
- **Example**: "Turn off the Lights" attends to itself
- Input and output sequences are identical

### The "Self" Definition

**Self-Attention is called "Self" because**:
- The attention mechanism is applied to the sequence **itself**
- It learns relationships **within** a single sequence
- Each word attends to all words (including itself) in the **same** sequence

## Summary

### Two Key Points

1. **Why "Attention"?**
   - Mathematical formulation identical to traditional attention mechanisms
   - Uses: similarity scores (dot product) → weights (softmax) → context (weighted sum)

2. **Why "Self"?**
   - Operates on a single sequence (intra-sequence)
   - Unlike traditional attention that operates between two distinct sequences (inter-sequence)
   - The sequence attends to itself

### Conceptual Foundation
Understanding these distinctions is crucial for:
- Multi-Head Attention
- Positional Encoding
- Full Transformer Architecture
