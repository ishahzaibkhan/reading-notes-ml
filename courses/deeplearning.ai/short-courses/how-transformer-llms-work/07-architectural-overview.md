# Transformer Architecture Deep Dive

## Core Generation Principle

### Token-by-Token Generation
**Critical Intuition:** Transformers generate tokens **one by one**, not all at once

**Process:**
- Input prompt → Model processes
- Output text generated sequentially
- One token at a time generation

---

## Three Major Components of Transformers

### 1. Tokenizer
**Function:** Breaks down text into multiple chunks (tokens)

**Token Embeddings:**
- **Example:** Vocabulary of 50,000 tokens
- Model has 50,000 vectors representing each token
- These vectors are **token embeddings**
- Substituted at the beginning when model processes inputs

### 2. Stack of Transformer Blocks
**Characteristics:**
- Where **majority of computing** happens
- Contains the neural networks
- Does "all the magic" (though understandable, not actually magic)
- Processes the token embeddings

### 3. Language Modeling Head
**Function:** Final neural network that outputs next token

**Process:**
- Takes output from transformer blocks
- Performs **scoring** or **token probability calculation**
- Based on all processing done by transformer stack
- Makes sense of input context and what's requested in prompt
- Determines what next token should be

**Output:** Token probability distribution
- Scores all 50,000 tokens in vocabulary
- Each token gets probability percentage
- **All probabilities must add up to 100%**
- Example: "dear" scored at 40% probability

---

## Decoding Strategies

### Greedy Decoding
**Method:** Always choose highest probability token
- **When Used:** Temperature parameter set to zero
- **Good for:** Many use cases
- **Characteristic:** Deterministic output

### Alternative Strategies

**Top-P Sampling:**
- Incorporates multiple tokens
- Usually generates highest probability token
- Sometimes picks lower probability tokens
- Looks at scoring but doesn't always pick top one

**Purpose:**
- Generate text that sounds natural
- Introduce variety in outputs
- Why same prompt can generate different answers

**Temperature Parameter:**
- **Zero:** Greedy decoding (always top token)
- **Greater than zero:** More randomness, varied outputs

---

## Parallel Processing (Key Advantage)

### Why Transformers Are Better Than RNNs

**Parallel Token Processing:**
- Process **all input tokens simultaneously**
- Not sequential like RNNs
- **Time efficient:** Can compute long context on multiple GPUs in similar time

### Visualization: Multiple Tracks
- **Tracks:** Flow through stack of transformer blocks
- **Number of tracks = Context size** of the model
- Example: Model with 16,000 token context size can process 16,000 tokens at same time

### Context Size
**Definition:** Maximum number of tokens model can process simultaneously
- Example: 16,000 token context = 16,000 parallel tracks
- All tokens processed in parallel through transformer blocks

---

## Generation Loop Process

### First Step (Initial Processing)
**Input:** Original prompt
**Process:**
1. All tokens flow through transformer blocks in parallel
2. Language modeling head generates probabilities
3. **Output:** First generated token (from final token position)

**Note:** All arrows are "red" (representing fresh computation)

### Subsequent Steps (Iterative Generation)
**Input:** Original prompt + previously generated token(s)
**Process:**
1. Feed entire sequence back into transformer
2. **Loop structure:** Continuous iteration
3. Process inputs one by one as tokens are generated

---

## KV Caching (Optimization)

### Problem
Between first and subsequent steps, many calculations are repeated

### Solution: KV Caching
**Key Innovation:** Cache calculations from previous steps
- **K:** Keys
- **V:** Values
- Calculations remain exactly the same for previous tokens
- **Benefit:** Speed up generation significantly
- Major addition making transformers faster

### Why It Works
- Previous token computations don't change
- No need to recalculate
- Only compute new token processing

---

## Performance Metrics

### Time to First Token (TTFT)
**Definition:** How long model takes to process initial input and generate first token
- Includes full parallel processing of input
- Initial computation through all transformer blocks
- First probability calculation and token selection

### Subsequent Token Generation
- Slightly different process (uses KV caching)
- Faster than first token (cached computations)
- Incremental generation

---

## Key Takeaways

1. **Sequential Output:** Despite parallel processing, output is generated one token at a time
2. **Three Components:** Tokenizer → Transformer Blocks → Language Modeling Head
3. **Probability Distribution:** Language modeling head outputs probabilities for all vocabulary tokens
4. **Decoding Strategies:** Different methods for choosing next token (greedy, top-p, temperature-based)
5. **Parallel Processing:** All input tokens processed simultaneously (unlike RNNs)
6. **Context Size:** Determines how many tokens can be processed in parallel
7. **Generation Loop:** Iterative process feeding previous outputs back as inputs
8. **KV Caching:** Critical optimization for fast generation after first token
9. **Determinism vs Randomness:** Temperature parameter controls output variety
10. **Performance Metrics:** TTFT measures initial processing speed, different from subsequent generation

---

## Important Distinctions

### Processing vs Generation
- **Processing:** Parallel (all input tokens at once)
- **Generation:** Sequential (one output token at a time)

### First Token vs Subsequent Tokens
- **First Token:** Full computation, no caching, measured by TTFT
- **Subsequent Tokens:** Use KV caching, faster, incremental updates
