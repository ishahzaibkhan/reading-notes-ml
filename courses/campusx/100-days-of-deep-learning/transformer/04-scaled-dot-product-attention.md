# Scaled Dot Product Attention

## Overview
Scaled Dot Product Attention is a crucial component in the Transformer architecture. The key distinction from basic self-attention is the inclusion of a scaling factor that divides the dot product by √d_k to ensure stable training.

## Self-Attention Recap

### Process Flow
1. **Input Sentence**: Example: "Money Bank Grows"

2. **Embeddings**: Each word gets an embedding vector
   - e_money, e_bank, e_grows

3. **Weight Matrices**: Three learnable parameter matrices
   - **Wq** (Query weight matrix)
   - **Wk** (Key weight matrix)
   - **Wv** (Value weight matrix)
   - Learned during training via backpropagation

4. **Vector Generation**: Each embedding is transformed
   - **Query vector**: q = embedding · Wq
   - **Key vector**: k = embedding · Wk
   - **Value vector**: v = embedding · Wv

5. **Matrix Formation**: Vectors are stacked
   - **Q** (Query Matrix): All query vectors stacked
   - **K** (Key Matrix): All key vectors stacked
   - **V** (Value Matrix): All value vectors stacked

6. **Attention Calculation**:
```
   Attention(Q, K, V) = Softmax(Q K^T) V
```

## The Scaling Factor

### What is d_k?
- **d_k** = dimension of the key vector
- Examples:
  - 3-dimensional key vectors → d_k = 3
  - 512-dimensional key vectors → d_k = 512
- Dimensionality is determined by:
  - Embedding dimension
  - Shape of weight matrices (Wq, Wk, Wv)
- Note: d_k = d_q = d_v (all have the same dimension)

### Why Scaling is Necessary

#### The Dot Product Variance Problem

**Key Insight**: As vector dimensionality increases, the variance of dot products increases proportionally.

**Experimental Evidence** (using random vector pairs):
- **3D vectors**: Dot products range from -2 to 2 (low variance)
- **10D vectors**: Dot products range from -10 to 10 (higher variance)
- **1000D vectors**: Dot products range from -30 to 30 (very high variance)

**Mathematical Relationship**:
- 1D vectors: Variance = Var(x)
- 2D vectors: Variance ≈ 2 × Var(x)
- 3D vectors: Variance ≈ 3 × Var(x)
- **D-dimensional vectors: Variance = D × Var(x)**

#### Impact on Softmax and Training

**Softmax Behavior**:
- Converts numbers to probabilities (sum = 1)
- Exponential function: sensitive to input magnitude
- Example:
  - Input [0, 0] → Output [0.5, 0.5] (balanced)
  - Input [1, 10] → Output [very low, very high] (extreme)

**Training Problems with High Variance**:
1. High variance in Q K^T → extreme values
2. Softmax produces extreme probabilities (close to 0 or 1)
3. Backpropagation focuses only on large probabilities
4. Parameters associated with small probabilities don't update
5. **Vanishing gradient problem** → unstable, inefficient training

**Analogy**: A classroom where the teacher only notices tall students and ignores short students. High variance = large height differences = some students (parameters) get ignored.

**The Dilemma**:
- High-dimensional embeddings are needed for rich information extraction
- But high dimensions cause high variance and unstable training

## The Solution: Scaling by √d_k

### Variance Reduction Principle
Dividing numbers by a constant reduces variance proportionally.

Example:
- Numbers [10, 20, ..., 70] with Variance = 400
- Divide by 10: [1, 2, ..., 7] with Variance = 4

### Determining the Scaling Factor

**Variance Rule**: If Var(X) = σ², then Var(cX) = c² × σ²

**Application**:
- **1D**: Variance = Var(x) → No scaling needed
- **2D**: Variance = 2 × Var(x) → Divide by √2
  - Var(Y / √2) = (1/√2)² × 2 × Var(x) = Var(x) ✓
- **3D**: Variance = 3 × Var(x) → Divide by √3
  - Var(Z / √3) = (1/√3)² × 3 × Var(x) = Var(x) ✓

**General Formula**: For D-dimensional vectors, divide by √D to maintain constant variance.

Since D = d_k (dimension of key vectors), we divide by **√d_k**.

## Complete Scaled Dot Product Attention Formula
```
Attention(Q, K, V) = Softmax(Q K^T / √d_k) V
```

**Components**:
1. **Q K^T**: Dot product of query and key matrices
2. **/ √d_k**: Scaling factor (normalizes variance)
3. **Softmax**: Converts to probability distribution
4. **V**: Final transformation to contextual embeddings

**Result**: Stable training with controlled variance, regardless of embedding dimensionality.
