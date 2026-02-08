# Self-Attention: Detailed Mechanics

## Two-Step Process Overview

### Step 1: Relevance Scoring
Assign scores to determine how relevant each input token is to current token

### Step 2: Combining Information
Incorporate relevant information into current representation

---

## Attention Head Mechanics

### Core Components
Self-attention happens within an **attention head**

**Inputs:**
- **Current token:** Vector being processed
- **Other positions:** Vector representations of preceding tokens

### Three Projection Matrices

**Critical Concept:** Queries, Keys, and Values (Q, K, V)

1. **Query Projection Matrix (Q)**
   - Represents current position
   - Creates query vector for token being processed

2. **Key Projection Matrix (K)**
   - Represents other tokens in sequence
   - Each row = vector representing one position
   - Used for matching with queries

3. **Value Projection Matrix (V)**
   - Represents other tokens in sequence
   - Each row = vector representing one position
   - Used for information extraction

**Weight Matrices:**
- Used to calculate query, key, and value matrices
- Enable relevance scoring and information combination

---

## Step 1: Relevance Scoring - Detailed

### Goal
Assign relevance score to each token

**Example Output:**
- "the dog" → highest relevance score
- Other tokens → lower scores
- **All scores add up to 100%**

### Calculation Method
**Matrix Multiplication:**
1. Multiply **query vector** (current token)
2. By **key vectors** (previous tokens)
3. Result: Relevance scores for each token

**Result:** Tokens with higher scores will have more representation baked into enriched vector

**Note:** For detailed attention calculation and implementation, see DeepLearning.AI course on attention (by Joshua Starmer/StatQuest)

---

## Step 2: Combining Information - Detailed

### Process

**Step 1: Weight Values**
- Each token has associated **value vector**
- Multiply each token's score by its value vector
- Result: **Weighted values**

**Example:**
- "the dog" (highest score) → highest weighted value
- Lower-scored tokens → values closer to zero

**Step 2: Sum Weighted Values**
- Add all weighted values together
- Result: **Output of information combination step**

---

## Multi-Head Attention

### Parallel Processing
Same operation happens **in parallel** across multiple attention heads

### Each Attention Head Has:
- Own set of **query projection matrix**
- Own set of **key projection matrix**
- Own set of **value projection matrix**
- **Different attention assignments** to various vectors

### Multi-Head Attention Process

**Two Main Steps:**

1. **Splitting:**
   - Information split into multiple attention heads
   - Before self-attention calculation

2. **Combining:**
   - Information from all attention heads merged back together
   - Forms output of self-attention layer

**Benefit:** Different heads can learn different types of relationships

---

## Efficient Attention Mechanisms

### Problem
Self-attention is usually:
- One of the longest steps
- Requires most computational time in transformers

### Solution 1: Multi-Query Attention (MQA)

**Traditional Approach:**
- Each attention head has own keys and values

**Multi-Query Attention:**
- All attention heads **share same** keys matrix and values matrix
- Only **one set** of K and V projection matrices for entire layer
- Each head still has own queries

**Benefits:**
- Compression of parameters
- Smaller number of parameters to calculate
- Faster self-attention calculation

### Solution 2: Grouped Query Attention (GQA)

**Approach:**
- Use multiple keys and values (not just one set)
- **Fewer than number of attention heads**
- Number of K/V sets = "number of groups"

**Benefits:**
- Better results than single shared K/V (MQA)
- More efficient than full multi-head (each head has own K/V)
- Important for larger models with extensive training data

**In Papers:**
- Models mention: number of attention heads
- Also mention: number of groups (key-value heads)

---

## Sparse Attention

### Motivation
In larger models, full attention at every layer becomes too expensive

### Full Attention vs Sparse Attention

**Full Attention:**
- Every token can attend to every previous token
- Token 7 can attend to tokens 1-6
- Happens at all layers

**Sparse Attention:**
- **Not in all layers** - typically interleaved
- Example: Layers 2, 4, 6 use sparse attention
- Tokens can only attend to **limited history**
  - Last 4, 16, or 32 tokens
  - Not entire context

### Visualization Example

**Token Processing:**
- Token 1 ("the"): No previous tokens to attend to
- Token 2 ("dog"): Attends to "the" and "dog"
- Token 3 ("chased"): Attends to all three tokens

**Full Attention Pattern:**
- Each row = processing step
- Every token attends to all previous tokens
- Complete triangular pattern

### Sparse Attention Patterns

**1. Strided Attention:**
- Look back at last 3-4 tokens
- Also look at specific fixed positions (e.g., position 1, 4)

**2. Fixed Attention:**
- Fixed positions in context sequence
- After token 4: only attend to tokens from 4 to current position
- Predetermined attention pattern

**Reference:** "Generating Long Sequences with Sparse Transformers" paper

---

## Advanced: Ring Attention

### Purpose
Enable models to process extremely long contexts:
- 100,000 tokens
- 1 million tokens

### Resource
Blog post: "Ring Attention Explained"
- Highly visual and animated
- Detailed explanation of mechanism

---

## Reading Model Architecture Papers

### Example: Llama 3.1 (Meta)

**When reading architecture sections, understand these parameters:**

#### Llama 3.1 8B Model Parameters

**Number of Layers: 32**
- 32 transformer blocks stacked
- Visual: Stack of 32 identical blocks

**Model Dimension: 4,096**
- Length of vector flowing through transformer
- Size of representations at each layer

**Feed-Forward Dimension: 14,336**
- Number of units in hidden layer of FFN
- Expansion size in middle of feed-forward network

**Attention Heads: 32**
- Number of parallel attention heads
- Each learns different patterns

**Grouped Query Attention: 8 key-value heads**
- Uses GQA (not full multi-head or MQA)
- 8 groups sharing keys and values
- More efficient than 32 separate K/V sets

**Vocabulary Size: 128,000**
- Number of tokens in tokenizer
- Size of embedding matrix

**Positional Embeddings: RoPE**
- Rotary Position Embeddings
- Method for encoding position information

---

## Key Takeaways

1. **Q, K, V Matrices:** Fundamental to attention mechanism
2. **Relevance Scoring:** Query-Key multiplication determines attention weights
3. **Information Combination:** Value vectors weighted by scores and summed
4. **Multi-Head Attention:** Multiple parallel attention operations capture different patterns
5. **Efficiency Matters:** Computational cost drives architectural innovations
6. **Multi-Query Attention:** Share K/V across heads for efficiency
7. **Grouped Query Attention:** Balance between efficiency and performance
8. **Sparse Attention:** Limit attention scope to reduce computation
9. **Attention Patterns:** Different strategies (strided, fixed) for different use cases
10. **Reading Papers:** Understanding these concepts enables comprehension of model architectures

---

## Efficiency Hierarchy

**Most Expensive (Full Attention):**
- Each head has own Q, K, V
- Every token attends to all previous tokens
- All layers use full attention

**Medium Efficiency:**
- Grouped Query Attention
- Sparse Attention (selective layers)

**Most Efficient:**
- Multi-Query Attention (shared K, V)
- Sparse Attention (limited context)
- Combination of strategies

---

## Practical Implications

### For Model Selection
- Larger vocabulary → more memory but better token efficiency
- More layers → better performance but slower
- GQA → good balance of speed and quality
- Sparse attention → enables longer contexts

### For Understanding Behavior
- Multi-head attention → captures different relationships
- Attention patterns → determines what context is used
- Efficiency mechanisms → trade-offs between speed and quality
