# Mixture of Experts (MoE) - Detailed Architecture

## Decoder Block Recap

### Standard Decoder Flow

**Step 1: Layer Normalization**
- Input vectors normalized first

**Step 2: Masked Self-Attention**
- Weight tokens based on relative importance
- Context of all other tokens considered

**Step 3: Residual Connection**
- Output aggregated with unprocessed inputs
- Creates both direct and indirect path
- **Purpose:** Attention mechanism prepares input with more contextual information

**Step 4: Layer Normalization**
- Normalize before feed-forward processing

**Step 5: Feed-Forward Neural Network**
- Typically **largest component** of LLM
- Finds complex relationships in attention-processed information
- Processes through one or more hidden layers

**Step 6: Residual Connection**
- Final aggregation

---

## Dense vs Sparse Networks

### Dense Network (Traditional FFN)
**Characteristic:** All parameters activated and used (at least to some degree)
- Single feedforward neural network
- Every parameter always active
- No selectivity in parameter use

### Sparse Network (MoE)
**Characteristic:** Only subset of parameters activated at given time
- Multiple feedforward neural networks (experts)
- Selective activation based on input
- **MoE Layer:** The sparse model implementation

---

## Mixture of Experts Architecture

### Core Concept
**Replacement:** Single FFN → Multiple expert FFNs + Router

**Example Structure:**
- Instead of 1 feedforward network
- Now have 4 networks (experts)
- When input flows through: one or more experts selected
- Other experts remain **unactivated**

### Two Main Components

#### 1. Experts
**Definition:** Each expert is a feedforward neural network

**Key Points:**
- **Not domain-specialized:** Not psychology expert, biology expert, etc.
- **Token-level specialization:** Learn syntactic information
  - Punctuation patterns
  - Verb processing
  - Conjunctions
  - Other linguistic features
- **Multiple per layer:** Each MoE layer has its own set of experts

#### 2. Router
**Definition:** Feedforward neural network (smaller than experts)

**Function:** Choose which inputs go to which expert

**Process:**
1. **Probability Scoring:** Creates probability score for each expert
   - Indicates how likely expert is suited for particular input
   - One score per expert
2. **Expert Selection:** Selects experts based on scores

**Selection Strategies:**
- **Highest Probability:** Select expert with top score
- **Creative Strategies:** Other methods that introduce more variability

---

## MoE Layer Operation

### Input Processing Flow

**Step 1: Input Arrives**
- Token vectors enter MoE layer

**Step 2: Router Analysis**
- Router evaluates input
- Generates probability for each expert

**Step 3: Expert Selection**
- Based on router probabilities
- Can select single or multiple experts

**Step 4: Expert Processing**
- Selected expert(s) process input
- Unselected experts remain inactive

**Step 5: Output Generation**

**Single Expert:**
- Use that expert's output directly

**Multiple Experts:**
- **Aggregation:** Weighted mean
- Higher router probability → bigger influence on final output
- Combine multiple expert outputs

---

## Computational Requirements

### Parameter Distribution (Five Locations)

1. **Input Embeddings**
2. **Masked Self-Attention**
3. **Router**
4. **Experts** (largest component)
5. **Output Embeddings**

### Sparse vs Active Parameters

#### Sparse Parameters
**Definition:** All parameters that need to be loaded
- Must load entire model
- Includes **all experts** even if unused
- **High memory requirement** for loading

#### Active Parameters
**Definition:** Parameters actually used during inference
- Only subset used
- **One or more experts** (not all)
- **Lower memory during inference**

### Resource Trade-off
- **Loading:** High memory (all parameters)
- **Inference:** Comparatively low (subset of parameters)
- **Benefit:** Efficient production deployment

---

## Example: Mixtral 8x7B Analysis

### Model Specification
- **Name suggests:** 8 experts × 7B parameters each
- **Reality:** More nuanced

### Parameter Breakdown

#### Shared Parameters (Always Used)
**Attention Mechanism:** >1 billion parameters
- Largest shared component
- Used in both loading and inference

**Router:** 32,000 parameters
- Relatively small
- Lightweight routing decision

#### Expert Parameters
**Per Expert:** 5.6 billion parameters (not 7B!)
- **Total Experts:** 8
- **Combined:** 45 billion parameters
- **Majority** of model parameters

**Naming Discrepancy:**
- Authors likely added shared parameters to expert count
- 5.6B (expert) + shared = ~7B per expert
- **Misleading:** Shared parameters shouldn't count toward expert-specific params

### Total Model Size

**Sparse Parameters (Loading):** 46 billion
- Everything must be loaded into memory

**Active Parameters (Inference):** Much fewer
- Shared parameters (~1B+ for attention)
- Router (32K)
- **Only 2 experts selected at a time** (Mixtral's strategy)
- Greatly reduces parameter count during inference

---

## Advantages and Disadvantages

### Pros

**1. Efficient Inference**
- Fewer active parameters during generation
- Less VRAM/GPU memory required
- **Excellent for production deployment**

**2. Higher Performance**
- Better than traditional dense models
- Experts remove redundancy in computations
- Specialized processing paths

**3. Flexible Architecture**
- Choice in which experts to use
- Adaptable routing strategies
- Can tune expert selection

### Cons

**1. High Memory Requirements (Loading)**
- Need to load all experts
- Large model size on disk/memory

**2. Overfitting Risk**
- Model might overfit on single expert
- Requires **careful balancing**
- Need to ensure expert diversity

**3. Complex Architecture**
- Requires careful training
- More components to manage
- Router optimization crucial

---

## Beyond Transformers: MoE Versatility

### Key Insight
MoE layer only affects **feedforward neural network**, not attention mechanism

### Applicability
**Can be used in non-transformer models:**

**State Space Models:**
- **Mamba:** Has MoE variants
- **Jamba:** Alternative to transformers with MoE

**Benefit:** MoE is architecture-agnostic technique
- Useful across entire LLM space
- Not limited to transformers
- Applicable to various neural architectures

---

## Technical Implementation Challenges

### Load Balancing
**Problem:** Ensuring experts are used relatively evenly
**Solutions:**
- Auxiliary loss functions
- Encourage router to balance expert usage
- Prevent single-expert dominance

### Router Training
**Challenge:** Learning good routing decisions
**Considerations:**
- Joint training with experts
- Router must learn meaningful distinctions
- Balance exploration vs exploitation

### Expert Specialization
**Goal:** Different experts learn different patterns
**Challenge:** Preventing expert collapse (all learning same thing)
**Methods:**
- Diversity encouragement
- Regularization techniques
- Careful initialization

---

## Key Takeaways

1. **MoE Replaces FFN:** Multiple expert networks instead of single dense network
2. **Router is Critical:** Small network making big decisions about expert selection
3. **Sparse Activation:** Only subset of experts active at any time
4. **Not Domain Experts:** Specialize in token-level patterns, not knowledge domains
5. **Memory Trade-off:** High loading requirements, efficient inference
6. **Weighted Aggregation:** Multiple expert outputs combined by router probabilities
7. **Mixtral Example:** Real-world implementation shows naming complexity
8. **Production Benefits:** Lower inference requirements make MoE ideal for deployment
9. **Overfitting Risk:** Careful balancing required to maintain expert diversity
10. **Beyond Transformers:** Applicable to various architectures, not just transformers

---

## Practical Implications

### When to Use MoE
- **Production deployment** where inference efficiency matters
- Need higher quality than dense model of similar active parameter count
- Have resources for large model storage
- Can handle training complexity

### When to Avoid MoE
- Limited storage/loading memory
- Simple deployment requirements
- Don't need extra performance boost
- Want simpler training process

### Design Considerations
- **Number of experts:** Balance between capacity and complexity
- **Routing strategy:** Top-k selection, probabilistic sampling
- **Expert size:** Trade-off with number of experts
- **Load balancing:** Prevent expert underutilization
