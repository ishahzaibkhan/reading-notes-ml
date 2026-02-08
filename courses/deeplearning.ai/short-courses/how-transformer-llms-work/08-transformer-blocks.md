# Transformer Blocks: Internal Architecture

## Overview of Transformer Block Flow

### Basic Example: "the Shawshank"

**Tokenization:**
- "the" → Token 1
- "Shawshank" → Token 2

**Embedding Substitution:**
- Replace tokens with their associated embedding vectors
- **Language converted to numbers** → Now can apply calculus to predict next word

### Flow Through Transformer Stack

**Sequential Processing:**
1. **Input:** Token embeddings (vectors)
2. **First Transformer Block:**
   - Processes embeddings in parallel across tracks
   - Generates vector of same size as output
   - Internal processing happens (explored below)
3. **Second Transformer Block:**
   - Operates on outputs of first block
   - Parallel processing across tracks
4. **Continue:** Process repeats through all transformer blocks
5. **Final Layer:** Vector from final token presented to language modeling head
6. **Output:** Predicted next token

**Direction:** One-way flow
- Tokenizer → Transformer Blocks (one by one in sequence) → Language Modeling Head

---

## Two Major Components of Transformer Block

### 1. Feed-Forward Neural Network Layer

#### High-Level Intuition
**Function:** Statistical pattern storage

**Example:** "the Shawshank" → "redemption"
- Most probable token to follow "Shawshank" is "redemption"
- Reference to famous film
- Training data (internet, Wikipedia) shows "redemption" statistically appears after "Shawshank"

**Key Capability:**
- Storage of information and statistics
- Predicts next word based on input token
- Models statistical patterns from training data

#### Architecture
**Structure:** Classic neural network
1. **Input Layer:** Receives embedding
2. **Hidden Layer:** Expands (larger)
3. **Output Layer:** Shrinks back down

**Where Knowledge Lives:**
- **Dense layer connections** store all information
- Models encode:
  - Code generation capabilities
  - World knowledge
  - Language fluency
  - Coherent generation abilities

**Note:** Does more complicated things, but this is reasonable first approximation

### 2. Self-Attention Layer

#### Purpose
Allows model to attend to previous tokens and incorporate context

#### Coreference Resolution Example
**Prompt:** "the dog chased the llama because it"

**Question:** What does "it" refer to?
- The dog?
- The llama?

**Self-Attention Function:**
- Model needs to understand what "it" refers to
- Incorporates representation of relevant token (e.g., "llama")
- Bakes context into current token representation
- While processing "it", model has "understanding" it refers to llama

**NLP Task:** Coreference resolution
- Difficult to determine from words alone
- Context from previous tokens provides answer

---

## Self-Attention: High-Level Formulation

### Components

**Input:**
1. **Other Positions:** Previous tokens already processed
2. **Current Position:** Token being processed now
3. **Vector Representation:** 
   - Embedding (if in first transformer block)
   - Output from previous block (if in later blocks)

**Output:**
- Vector representation that bakes in relevant information from other positions

### Visualization: Five Tracks
- Processing multiple positions in parallel
- Pulling in right information useful to represent current token
- Selective incorporation based on relevance

---

## How Self-Attention Works

### Two-Step Process

#### Step 1: Relevance Scoring
**Function:** Assigns score to how relevant each input token is to current token

**Process:**
- Evaluate all previous tokens
- Determine which are most relevant to current processing
- Generate relevance scores

#### Step 2: Combining Information
**Function:** Incorporate relevant information into representation

**Process:**
- Based on relevance scores
- Combine information from relevant tokens
- Update current vector representation
- Create contextualized embedding

---

## Feed-Forward vs Self-Attention

### Feed-Forward Neural Network
- **Looks at:** Individual token in isolation
- **Function:** Statistical pattern matching
- **Capability:** Predict likely next word based on training data
- **Storage:** Encodes factual knowledge and patterns

### Self-Attention
- **Looks at:** Current token + context (previous tokens)
- **Function:** Contextual understanding
- **Capability:** Resolve references, incorporate context
- **Processing:** Relevance scoring + information combination

---

## Key Concepts

### Parallel Processing Within Block
- Multiple tracks (tokens) processed simultaneously
- Each track flows through same operations
- Maintains parallelization advantage

### Vector Transformation
- Input vector → Transformer block processing → Output vector
- **Same size maintained** through processing
- Information enriched, not expanded

### Information Flow
**Within Each Block:**
1. Self-attention adds context
2. Feed-forward adds statistical knowledge
3. Combined output passes to next block

**Across Blocks:**
- Each block refines representation
- Stacking amplifies capabilities
- Progressive refinement of understanding

---

## Key Takeaways

1. **Two Components:** Feed-forward network + Self-attention layer
2. **Feed-Forward Role:** Statistical pattern storage and prediction
3. **Self-Attention Role:** Contextual understanding and reference resolution
4. **Sequential Blocks:** Information flows one direction through stack
5. **Parallel Tracks:** Multiple tokens processed simultaneously within each block
6. **Relevance Scoring:** Critical first step in self-attention
7. **Information Combination:** Selective incorporation based on relevance
8. **Vector Preservation:** Same size maintained, but information enriched
9. **Complementary Functions:** Feed-forward and attention serve different purposes
10. **Knowledge Storage:** Dense connections in feed-forward layers store model's knowledge

---

## Next Steps
Detailed examination of:
- Specific mechanisms of self-attention
- Mathematical operations involved
- How relevance scores are calculated
- How information combination works
