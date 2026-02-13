# Layer Normalization in Transformers

## Introduction to Normalization

### Core Concepts
- **Normalization**: Transforming data to have specific statistical properties (mean, standard deviation)
- **Standardization**: Transforms data to mean = 0 and standard deviation = 1
- **Min-Max Scaling**: Transforms data to a specific range (e.g., 0 to 1)

### What Gets Normalized in Deep Learning
1. **Input Data**: Normalizing features before feeding into the input layer
2. **Hidden Layer Activations**: Normalizing outputs/weighted sums of hidden layers

## Benefits of Normalization

### 1. Improved Training Stability
- Reduces likelihood of extreme values
- Prevents gradient explosion and vanishing
- Keeps activations within stable ranges

### 2. Faster Convergence
- More consistent gradient magnitudes
- More efficient optimization
- Quicker model training

### 3. Mitigation of Internal Covariate Shift (ICS)
- **Covariate Shift**: Distribution change between training and testing data
  - Example: Training on red roses, testing on yellow/white roses
- **Internal Covariate Shift**: Output distributions of hidden layers change during training
  - As weights in earlier layers update via backpropagation, inputs to subsequent layers shift
  - Normalization maintains consistent distribution across layers

### 4. Regularization Effect
- Provides mild regularization (especially Batch Normalization)
- Helps prevent overfitting

## Batch Normalization

### Process
1. **Batch Processing**: Data sent in batches (e.g., batch size 5)
2. **Calculate Weighted Sums**: Compute z1, z2, z3 for each node in hidden layer
3. **Normalization Across Batch** (Column-wise):
   - For each node, calculate mean (μ) and standard deviation (σ) across all data points in the batch
   - Each node has its own statistics
4. **Standardization**: 
```
   z_normalized = (z - μ) / σ
```
5. **Scaling and Shifting**: 
```
   z_final = γ * z_normalized + β
```
   - γ (gamma) and β (beta) are learnable parameters
   - Learned during training

## Why Batch Normalization Fails in Transformers

### Core Issue: Incompatibility with Self-Attention and Sequential Data

### Self-Attention Overview
- Tokenizes sentences into words
- Calculates embedding vectors for each word
- Generates contextual embeddings considering word context
- Output vectors have same dimension as input embeddings

### The Padding Problem

#### Example: Sentiment Analysis
- Batch contains sentences of different lengths:
  - "Hi Nitish"
  - "How are you today?"

#### Padding Requirement
- All sentences in batch must have same length
- Shorter sentences padded with zero embedding vectors

#### The Critical Flaw
1. After Self-Attention, output matrices contain contextual embeddings
2. Padding words produce zero output vectors
3. Batch Normalization calculates statistics **column-wise** (across batch)
4. **Problem**: Varying sentence lengths mean varying amounts of padding zeros
5. **Result**: Mean and standard deviation calculations include many unnecessary zeros from padding
6. **Impact**: Statistics don't represent actual data, compromising normalization effectiveness

## Layer Normalization

### Key Difference
- **Batch Normalization**: Normalizes across the batch (column-wise)
- **Layer Normalization**: Normalizes across features (row-wise)

### Process
1. **For Each Individual Data Point** (row-wise):
   - Calculate mean (μ) and standard deviation (σ) across all features for that specific row
2. **Standardization**:
```
   z_normalized = (z - μ_row) / σ_row
```
3. **Scaling and Shifting**:
```
   z_final = γ * z_normalized + β
```
   - Each feature still has its own independent γ and β parameters

### Layer Normalization in Transformers

#### Application to Self-Attention Output
- Applied row-wise across embedding dimensions for each word's contextual embedding
- Each word's embedding is normalized independently

#### Critical Advantage
- **Padding zeros don't affect other words' statistics**
- Zeros only impact their own padded row (which is already zero)
- Statistical representation of actual data is preserved
- Effective normalization without padding interference

## Summary

### Why Layer Normalization for Transformers?
- Works seamlessly with sequential data of varying lengths
- Immune to padding-induced statistical distortion
- Maintains accurate normalization for actual data
- Essential for effective Self-Attention mechanism

### Key Distinction
- **Batch Normalization**: Statistics computed across batch → fails with variable-length sequences
- **Layer Normalization**: Statistics computed per sample → robust to sequence length variation

