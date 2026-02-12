# Multi-head Attention in Transformers

## Overview
Multi-head Attention is an extension of Self-Attention designed to enhance the model's ability to capture diverse patterns and relationships in data by applying multiple attention mechanisms in parallel.

## Self-Attention Fundamentals

### The Problem with Static Embeddings
- Traditional embeddings (like Word2Vec) are **static** - the same word always has the same embedding regardless of context
- Example: "bank" has identical representation in both "money bank" and "river bank"
- This creates ambiguity in word meaning

### Self-Attention Solution
Self-Attention generates **contextual embeddings** that change based on surrounding words, resolving the ambiguity of static embeddings.

### Self-Attention Mechanism

**Process Flow:**
1. Input sentence (e.g., "Money Bank")
2. Generate static embeddings (E_money, E_bank)
3. Pass through Self-Attention block
4. Output contextual embeddings (Y_money, Y_bank)

**Detailed Steps:**
1. **Generate Q, K, V vectors**: For each word embedding, three weight matrices (Wq, Wk, Wv) produce Query, Key, and Value vectors
2. **Calculate similarity scores**: Dot product of Query vector with all Key vectors
3. **Scale**: Normalize the scores
4. **Softmax**: Convert scaled scores to attention weights
5. **Weighted sum**: Multiply weights with Value vectors to produce final contextual embedding
6. **Repeat**: Process performed for all words in the sentence

## The Limitation of Single Self-Attention

### Single Perspective Problem
Self-Attention can only capture **one perspective** or interpretation of ambiguous text.

**Example:** "The man saw the astronomer with a telescope"
- **Interpretation 1**: The man used a telescope to see the astronomer (high similarity between "man" and "telescope")
- **Interpretation 2**: The astronomer had a telescope (high similarity between "astronomer" and "telescope")

Standard Self-Attention typically captures only one interpretation, creating a single table of word similarities.

### Need for Multiple Perspectives
Many NLP tasks (like document summarization) require understanding content from multiple angles simultaneously.

## Multi-head Attention Architecture

### Core Concept
Instead of one Self-Attention module, Multi-head Attention uses **multiple Self-Attention modules in parallel**, each called a "head."

### Mechanism

**Independent Processing:**
- Each head has its own independent set of weight matrices (Wq, Wk, Wv)
- This produces multiple sets of Q, K, V vectors for each word
  - Example with 2 heads: Q_money_1, Q_money_2; K_money_1, K_money_2; V_money_1, V_money_2
- Each head performs complete Self-Attention independently
- Results in multiple contextual embeddings per word (e.g., Y_money_1, Y_money_2)

**Outcome:**
Multiple heads enable the model to capture different perspectives or relationships within the input sequence simultaneously.

## Matrix Form Implementation

### Step-by-Step Process

**1. Input Embeddings**
- Sentence converted to embedding matrix
- Example: "Money Bank" → 2×4 matrix (2 words, 4-dimensional embeddings)

**2. Multiple Weight Matrices**
- Each head has separate Wq, Wk, Wv matrices
- Example: 4×4 matrices for 4-dimensional embeddings

**3. Generate Q, K, V Matrices**
- Input matrix multiplied with each head's weight matrices
- Produces Q, K, V matrices for each head (Q1, K1, V1, Q2, K2, V2)
- Each matrix contains vectors for all input words

**4. Parallel Self-Attention**
- Full Self-Attention calculation performed independently for each head
- Produces output matrices (Z1, Z2, etc.)
- Example: Z1 contains Y_money_1 and Y_bank_1

**5. Concatenation**
- All head outputs concatenated into single matrix (Z')
- Example: Two 2×4 matrices → one 2×8 matrix

**6. Final Linear Transformation**
- Output weight matrix (W_output) projects concatenated result back to original dimension
- Shape: (concatenated_dim × original_dim)
- Example: 8×4 matrix transforms 2×8 back to 2×4
- **Purpose**: Learns to combine different perspectives, balancing their importance for optimal representation

## Original Transformer Paper Implementation

### Specifications
- **Embedding dimension**: 512
- **Number of heads**: 8
- **Input matrix**: 2×512 (for 2 words)

### Linear Projection and Dimension Reduction
- Each Wq, Wk, Wv matrix: 512×64
- Multiplication produces Q, K, V matrices of 2×64 per head
- Reduces dimensionality from 512 to 64 for each head

### Processing
- 8 parallel Self-Attention computations on 64-dimensional vectors
- Produces 8 output matrices (2×64 each)
- Concatenation creates 2×512 matrix (8 × 64 = 512)

### Final Transformation
- W_output matrix: 512×512
- Projects concatenated output back to original 2×512 dimension

### Computational Efficiency
Reducing dimensionality to 1/8th per head (512÷8=64) and running 8 parallel heads means:
- Overall computation ≈ single Self-Attention on full 512-dimensional vector
- Benefit: Multiple perspectives without additional computational cost

## Visualization Example

Using "The man saw the astronomer with a telescope":

**Head 0:**
- Strong similarity between "man" and "telescope"
- Captures: "The man used a telescope to see the astronomer"

**Head 1:**
- Strong similarity between "man" and "astronomer"
- Strong similarity between "astronomer" and "telescope"
- Captures: "The man saw an astronomer who had a telescope"

This demonstrates how multiple heads simultaneously capture and consider different interpretations of the same input.
