# Transformer Encoder Architecture

## Introduction and Prerequisites

### Video Purpose
- Detailed explanation of Transformer's encoder architecture
- Focus on practical examples and the 'why' behind each component
- Integration of previously explained components into full architecture

### Teaching Approach
- Individual components covered first (self-attention, multi-head attention, positional encoding, layer normalization)
- Then integrated into complete Transformer architecture
- Prevents overwhelming complexity

### Architecture Overview
- Transformer consists of two main parts: **Encoder** and **Decoder**
- This focuses specifically on the Encoder

### Required Prerequisites
- Introduction to Transformers
- Self-Attention (query, key, value vectors, main goals)
- Multi-Head Attention
- Positional Encoding
- Layer Normalization

## Overall Transformer Architecture

### Structural Composition

#### Multiple Encoder Blocks
- Not a single block, but **6 identical blocks stacked together** (original paper)
- Number chosen through experimentation (not arbitrary)
- All encoder blocks share identical internal architecture
- Understanding one block = understanding all six

#### Single Encoder Block Components
Each encoder block contains two main sub-blocks:
1. **Multi-Head Attention Block**
2. **Feed-Forward Neural Network**

Additional components:
- Add & Norm layers
- Residual Connections (Skip Connections)

### Information Flow
1. Input enters first Encoder block
2. Output of first block → input of second Encoder block
3. Process continues through all six blocks
4. Output of sixth Encoder block → passed to Decoder

## Detailed Encoder Architecture

### Example Sentence: "How are you"

## Step 1: Input Processing Block

### 1.1 Tokenization
- Sentence split into individual words (tokens)
- Example: "How", "are", "you"

### 1.2 Text Vectorization/Embedding
- Each token converted to numerical vector via embedding layer
- Each word represented by **512-dimensional vector**
- Examples: how_embedding, are_embedding, you_embedding
- **Context-independent** at this stage

### 1.3 Positional Encoding
- **Why needed**: Embeddings don't capture word order
- For each word position (1, 2, 3), generate unique 512-dimensional positional vector
- Positional vector **added** to word's embedding vector
- Result: **Positional Encoding Embedded Vectors** (x1, x2, x3)
- Each vector remains 512-dimensional, now contains positional information

## Step 2: Multi-Head Attention Block

### Input
- Three positional encoding embedded vectors (x1, x2, x3)
- Each 512-dimensional

### Purpose
- Initial embeddings are **not context-aware**
- Example: "bank" in "river bank" vs "money bank" has same embedding initially
- Multi-Head Attention enables understanding context by word interactions

### Output
- New set of **contextually aware vectors** (z1, z2, z3)
- Each still 512-dimensional
- Refined versions incorporating contextual information

## Step 3: Add & Norm (First Instance)

### 3.1 Residual Connection (Skip Connection)
- Original input vectors (x1, x2, x3) **added directly** to Multi-Head Attention output (z1, z2, z3)
- Results in new vectors (z1', z2', z3')
- Each remains 512-dimensional

#### Why Use Residual Connections?

**Reason 1: Stable Training**
- Mitigates vanishing gradient problem
- Provides alternate path for gradient flow
- Prevents gradients from shrinking excessively in deep networks

**Reason 2: Preserving Original Features**
- Allows model to retain original features alongside transformed features
- If transformation doesn't perform well, model can rely on original features
- Acts as "identity operation" preventing performance degradation

### 3.2 Layer Normalization
- Applied to each resulting vector (z1', z2', z3')
- Normalizes values within each vector to smaller, consistent range

#### Why Normalize?
- Multi-Head Attention outputs can be in any range
- Normalization stabilizes neural network training
- Keeps values within defined range

### Output
- Normalized vectors (z1_norm, z2_norm, z3_norm)
- Still 512-dimensional

## Step 4: Feed-Forward Neural Network

### Architecture

**Two-Layer Network:**

1. **First Hidden Layer**
   - Input: 512-dimensional
   - Neurons: **2048**
   - Activation: **ReLU**
   - Weights (W1): 512×2048
   - Biases (b1): 2048
   - Dimensionality: 512 → 2048

2. **Second Hidden Layer**
   - Neurons: **512**
   - Activation: **Linear**
   - Weights (W2): 2048×512
   - Biases (b2): 512
   - Dimensionality: 2048 → 512

### Data Flow and Matrix Operations

1. Three input vectors (z1_norm, z2_norm, z3_norm) combined into **3×512 matrix**
2. Matrix multiplication: (3×512) × (512×2048) + b1 → **3×2048 matrix**
3. Apply ReLU activation
4. Matrix multiplication: (3×2048) × (2048×512) + b2 → **3×512 matrix**

### Why Dimensionality Expansion Then Reduction?
- Primary benefit: **Introduces non-linearities via ReLU**
- Allows network to learn more complex patterns
- Linear operations alone insufficient for complex relationships

## Step 5: Add & Norm (Second Instance)

### Process
1. **Residual Connection**: Feed-Forward Network output combined with its input (normalized vectors from previous step)
2. **Layer Normalization**: Applied to result

### Important Note on Parameters
- While each Encoder block has **identical architecture**
- **Parameters (weights and biases) are different** for each block
- Updated independently during backpropagation

### Final Output of Encoder Block
- Set of vectors (e.g., 3×512 for example)
- Retain original dimensionality
- Progressively richer in contextual information and learned representations
- Output from sixth Encoder block → passed to Decoder

## Key Questions and Research Insights

### 1. Why Use Residual Connections?

**Empirical Evidence:**
- Original Transformer paper doesn't explicitly state reason
- Critical for performance (Kaggle notebook experience)
- Model without them performed poorly; adding them back significantly improved performance

**Speculated Reasons:**
- **Stable Training**: Combat vanishing gradient problem in deep networks
- **Preserve Original Features**: Allow bypass of potentially degrading transformations

### 2. Why Feed-Forward Network After Multi-Head Attention?

**Active Research Area:**

**Common Speculation: Non-Linearities**
- Multi-Head Attention primarily performs linear operations
- Feed-Forward Network (especially ReLU in first layer) introduces crucial non-linearities
- Enables capturing complex, non-linear relationships

**Recent Research Perspective: Key-Value Memories**
- Research paper: "Transformer Feed-Forward Layers Are Key-Value Memories"
- Feed-forward layers constitute **two-thirds of Transformer parameters**
- Function as "key-value memories"
- Each "key" correlates with textual patterns in training examples
- Each "value" induces distribution over output vocabulary
- Store and retrieve information in memory-like fashion

### 3. Why Multiple Stacked Encoder Blocks?

**Enhanced Representation Power:**
- Language is highly complex
- Single layer insufficient to capture all intricate patterns and nuances
- Multiple layers learn progressively deeper, more abstract representations

**Deep Learning Philosophy:**
- Stacking layers is fundamental principle
- Allows learning hidden representations essential for complex phenomena
- Number of blocks (e.g., six) determined empirically for best results
- Can vary based on specific application

## Summary

### Complete Encoder Flow
1. **Input Processing**: Tokenization → Embedding → Positional Encoding
2. **Multi-Head Attention**: Context-aware representations
3. **Add & Norm**: Residual connection + Layer normalization
4. **Feed-Forward Network**: Non-linear transformations with dimensionality expansion/reduction
5. **Add & Norm**: Second residual connection + Layer normalization
6. **Repeat**: Process through six identical blocks
7. **Output**: Rich contextual representations → passed to Decoder

### Key Architectural Principles
- **Residual connections** enable stable training and preserve features
- **Layer normalization** maintains numerical stability
- **Feed-forward networks** introduce non-linearities and act as key-value memories
- **Stacking blocks** progressively enhances representation power
- **Consistent dimensionality** (512) maintained throughout for our example
