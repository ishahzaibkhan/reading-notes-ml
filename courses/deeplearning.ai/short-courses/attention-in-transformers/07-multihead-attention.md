# Multi-Head Attention

## Why Multi-Head Attention?

Single attention works for simple examples, but for **longer, more complex sentences and paragraphs**, we need to capture relationships from multiple perspectives simultaneously. The solution: apply attention **multiple times in parallel**.

---

## What is a Head?

Each **attention head** is an independent attention unit with its **own set of weights** for computing Q, K, and V matrices. Each head can learn to focus on different types of relationships between tokens.

---

## Multi-Head Attention: How It Works

### Step 1: Run Attention in Parallel
- Multiple heads compute attention simultaneously on the same encoded input
- Each head produces its own attention output
- The original transformer paper (Vaswani et al.) used **8 attention heads**

### Step 2: Reduce Back to Original Dimensionality
After all heads compute their outputs, the total number of values is multiplied. Two strategies to reduce back to the original size:

#### Method 1: Fully Connected Layer
- Concatenate all head outputs
- Pass through a **fully connected (linear) layer** with the desired number of outputs
- *Example: 3 heads × 2 values each = 6 values → FC layer → 2 outputs*

#### Method 2: Reshape the Value Weight Matrix
- Reduce the **number of columns** in the value weight matrix per head
- Fewer columns → fewer outputs per head → total outputs naturally match original size
- *Example: 2 encoded values → use 2 heads with 1-column value matrices → 2 total outputs*
- Transformers can also be built to handle this **flexibly**

---

## Summary Table

| Concept | Detail |
|---|---|
| **Head** | Single attention unit with its own Q, K, V weights |
| **Multi-Head Attention** | Multiple heads running attention in parallel |
| **Original paper** | Used 8 attention heads |
| **Output reduction** | Fully connected layer **or** reshaped value weight matrix |
| **Goal** | Capture richer, multi-faceted token relationships |


