# Geometric Interpretation of Self-Attention

## Overview
Self-Attention is a crucial component of the Transformer architecture that generates context-aware embeddings. This geometric interpretation explains what happens mathematically and visually when Self-Attention is applied to words.

## Self-Attention Process Recap

### Example Sentence: "Money Bank"

#### Step 1: Embedding Vectors
- Obtain embedding vectors for each word
- E_Money and E_Bank

#### Step 2: Weight Matrices
Three learned matrices (trained during backpropagation):
- **W_Q** (Query weight matrix)
- **W_K** (Key weight matrix)
- **W_V** (Value weight matrix)

#### Step 3: Generate Q, K, V Vectors
Each embedding is multiplied by all three weight matrices:
- E_Money → Q_Money, K_Money, V_Money
- E_Bank → Q_Bank, K_Bank, V_Bank

#### Step 4: Calculate Contextual Embeddings

**Goal**: Create Y_Money and Y_Bank (contextual embeddings)

**Process**:
1. **Similarity Scores**: Calculate dot products
   - Q_Money · K_Money
   - Q_Money · K_Bank

2. **Scaling**: Divide by √d_k (dimension of key vectors)

3. **Normalization**: Apply Softmax to get attention weights
   - W_Money_Money, W_Money_Bank

4. **Weighted Sum**: Combine Value vectors using attention weights
   - Y_Money = W_Money_Money × V_Money + W_Money_Bank × V_Bank

## Visualizing Embeddings in 2D Space

### Word2Vec Visualization
- Words plotted as points in 2D space (reduced from 200D using PCA)
- **Semantic clustering**: Similar words group together
  - Chemical elements: "Chloride," "Magnesium," "Sodium," "Oxide"
  - Formal words: "Indeed," "Regard," "Consequently"
- Embeddings can be understood as vectors in geometric space

## Geometric Intuition: Step-by-Step

### Setup
Using "Money Bank" with 2D embeddings:
- E_Money = [2, 7]
- E_Bank = [7, 3]
- W_Q, W_K, W_V are 2×2 matrices

### Step 1: Linear Transformation
**What happens**: Embedding vectors undergo linear transformation when multiplied by weight matrices

**Result**: 
- E_Money → Q_Money, K_Money, V_Money (3 new vectors)
- E_Bank → Q_Bank, K_Bank, V_Bank (3 new vectors)
- **Total**: 6 new vectors from 2 original embeddings

Each transformation moves the vector to a new position in the space.

### Step 2: Calculate Y_Bank (Contextual Embedding)

#### 2a. Similarity Scores (Dot Products)
Calculate angular relationships:
- **Q_Bank · K_Money = 10** (larger angular distance = lower similarity)
- **Q_Bank · K_Bank = 32** (smaller angular distance = higher similarity)

#### 2b. Scaling
Divide by √d_k = √2:
- 10 → 7.09
- 32 → 22.6

#### 2c. Normalization (Softmax)
Apply Softmax to [7.09, 22.6]:
- **W_Bank_Money = 0.2** (attention to "Money")
- **W_Bank_Bank = 0.8** (attention to "Bank" itself)

#### 2d. Weighted Sum of Value Vectors
**Q and K vectors are discarded; only V vectors remain**

**Scalar multiplication**:
- Scale V_Money by 0.2
- Scale V_Bank by 0.8

**Vector addition** (parallelogram law):
- Y_Bank = (0.2 × V_Money) + (0.8 × V_Bank)

### Step 3: Geometric Insight

**Key Observation**: Y_Bank is positioned significantly closer to E_Money than E_Bank was.

**Visual representation**:
- Original E_Bank is far from E_Money
- Contextual Y_Bank has moved toward E_Money
- This shift reflects the contextual relationship between "Money" and "Bank"

## The Brilliance of Self-Attention

### Context-Awareness Through "Gravity"
Self-Attention acts like gravitational pull in embedding space:
- Words that appear together in a sentence "pull" each other's embeddings closer
- In "Money Bank": "Money" pulls "Bank" toward its position
- The result: Y_Bank is context-aware

### Context-Dependent Representations

**Example 1**: "Money Bank"
- Y_Bank moves toward E_Money
- Represents financial institution context

**Example 2**: "River Bank" (hypothetical)
- Y_Bank would move toward E_River instead
- Represents riverside context

**Core Principle**: The same word ("Bank") gets different contextual embeddings based on surrounding words.

### Key Takeaway
Self-Attention empowers a word to be represented in terms of other words available in the sentence. The entire process is driven by patterns learned from the training dataset.

## Summary

Self-Attention transforms static word embeddings into dynamic, context-aware representations through:
1. **Linear transformations** (Q, K, V matrices)
2. **Similarity calculations** (dot products)
3. **Attention weighting** (scaled softmax)
4. **Vector composition** (weighted sums)

The geometric view reveals that Self-Attention creates a "pull" between related words, moving their representations closer in embedding space based on context.
