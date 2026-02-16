# Self-Attention: Matrix Math Explained

## Core Equation Variables

**Q, K, V Terminology** - derived from database concepts:
- **Q (Query)**: The search term used to query the database
- **K (Key)**: The actual entries in the database being searched against
- **V (Value)**: The result returned from the database

### Database Analogy
- Guest checks in with last name "Starmer"
- Receptionist types "Stammer" (query)
- Computer compares query to all last names in database (keys)
- Finds closest match and returns room number 537 (value)

## Self-Attention in Transformers

### Purpose
Self-attention calculates similarity between:
- Each word and itself
- Each word and all other words
- Performs this for every word in the sentence

### Requirements
- One query per word
- One key per word
- One value per word (returned by each key)

## Step-by-Step Process

### 1. Input Encoding
**Example prompt**: "Write a poem"

1. Convert each word to word embeddings
2. Add positional encoding to embeddings
3. Result: numerical encodings representing each word
   - Simple example: 2 numbers per word
   - Real-world: typically 512+ numbers per word

### 2. Creating Queries
- Stack encodings into a matrix
- Multiply by a 2×2 matrix of query weights (W^Q^T)
- Result: 2 query numbers per word

**Scaling rule**: 
- If starting with N encoded values per word → use N×N weight matrix to get N query numbers per word
- Only constraint: matrix multiplication must be mathematically valid

**Note**: Weights are transposed (indicated by ^T) because PyTorch outputs require transposition for correct matrix math

### 3. Creating Keys and Values
- **Keys**: Multiply encoded values by 2×2 key weights matrix (W^K^T)
- **Values**: Multiply encoded values by 2×2 value weights matrix (W^V^T)

### 4. Calculate Unscaled Similarities
**Operation**: Q × K^T (multiply query matrix by transposed key matrix)

**Why transpose K?**
1. **Technical reason**: Matrix dimensions must align for multiplication
2. **Mathematical reason**: Computes dot products between all query-key pairs

**Dot Product Process**:
- Multiply corresponding number pairs
- Sum the products
- Example: Query for "write" × Key for "write" = -0.09

**Dot products** = unscaled similarity measure (related to cosine similarity, but not scaled between -1 and 1)

**Result**: Matrix containing unscaled dot product similarities between all possible query-key combinations

### 5. Scale Similarities
**Formula**: Divide by √(d_k)

- d_k = dimension of key matrix (number of values per token)
- In example: d_k = 2, so divide by √2

**Purpose**: Improves performance (per original transformer paper), though scaling isn't systematic

### 6. Apply Softmax
**Operation**: Take softmax of each row in scaled similarity matrix

**Result**: Each row sums to 1, creating percentage relationships
- Example: Word "write" is 36% similar to itself, 40% similar to "A", 24% similar to "poem"

### 7. Calculate Final Self-Attention Scores
**Operation**: Multiply softmax percentages by value matrix V

**Process for each word**:
- Multiply row of percentages by columns in V
- Example for "write":
  - First score = 36% × (first value for "write") + 40% × (first value for "A") + 24% × (first value for "poem")
  - Repeat for second column to get second score

**Interpretation**: Percentages determine how much influence each word has on the final encoding of any given word

## Summary

The self-attention equation:
1. Calculates scaled dot product similarities among all words
2. Converts similarities to percentages using softmax
3. Uses percentages to scale values into final self-attention scores

**Key insight**: Despite appearing complex, self-attention is fundamentally about measuring and weighting relationships between words.
