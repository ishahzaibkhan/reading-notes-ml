# Transformer Architecture and Modern LLMs

## The Transformer Revolution

### "Attention is All You Need" Paper
- Introduced the **Transformer architecture**
- Based **solely on attention** without recurrent neural networks (RNNs)
- **Key Advantage:** Allows parallel training
- **Performance:** Significantly speeds up calculation compared to RNN-based models

---

## Transformer Architecture

### Overall Structure
- Consists of **stacked encoder and decoder blocks**
- Each block uses the same attention mechanism
- **Stacking effect:** Amplifies the strength of encoders and decoders

### Encoder Architecture

**Input Processing:**
1. Input "I love llamas" converted to embeddings
2. **Difference from Word2Vec:** Start with random values (not pre-trained)
3. **Self-Attention:** Attention focused only on the input
   - Processes embeddings and updates them
   - Updated embeddings contain more contextualized information
4. **Feedforward Neural Network:** Further processes the embeddings
5. **Output:** Contextualized token/word embeddings

**Purpose:** Encoder is meant for representing text and generating high-quality embeddings

#### Self-Attention Mechanism
- Attention mechanism that processes only **one sequence** (the inputs)
- Compares sequence to itself
- Different from standard attention (which processes two separate sequences)

### Decoder Architecture

**Processing Flow:**
1. Takes previously generated words
2. Passes to **Masked Self-Attention**
   - Similar to encoder's self-attention
   - Processes embeddings
3. Generates intermediate embeddings
4. Passes to another **Attention Network** together with encoder embeddings
   - Processes both: what has been generated AND what you already have
5. Output passed to neural network
6. **Final Output:** Generates next word in sequence

#### Masked Self-Attention
- Similar to self-attention
- **Key Difference:** Removes all values from upper diagonal
- **Purpose:** Masks future positions
- **Result:** Any given token can only attend to tokens that came before it
- **Benefit:** Prevents information leakage when generating output

---

## BERT: Encoder-Only Architecture (2018)

### Full Name
**B**idirectional **E**ncoder **R**epresentations from **T**ransformers

### Purpose
- Addresses limitation of original transformer (not easily used for tasks like text classification)
- Can be leveraged for wide variety of tasks

### Architecture
- **Encoder-only** architecture
- **Focus:** Representing language and generating contextual word embeddings
- Uses same encoder blocks: self-attention followed by neural networks

### CLS Token
- **Additional input token:** Classification token
- **Purpose:** Representation for entire input
- **Usage:** Often used as input embedding for fine-tuning on specific tasks (e.g., classification)

### Training: Masked Language Modeling

**Process:**
1. Randomly mask words from input sequence
2. Model predicts these masked words
3. **Learning:** Model learns to represent language by deconstructing masked words

**Two-Step Training Approach:**
1. **Pre-training:** Apply masked language modeling on large amounts of data
2. **Fine-tuning:** Fine-tune pre-trained model on downstream tasks (including classification)

---

## GPT: Decoder-Only Architecture

### Full Name
**G**enerative **P**re-**T**rained Transformer

### Architecture
- **Decoder-only:** Only stacks decoders (no encoders)
- Uses **masked self-attention**
- Followed by neural network
- Generates next word as output

### Process
1. Input sequence with randomly initialized embeddings
2. Passed to decoder only
3. Decoder block processes with masked self-attention
4. Passed to neural network
5. Next word generated

---

## Two Main Model Types

### 1. Generative Models
- Example: ChatGPT
- Generate text
- Decoder-based

### 2. Representation Models
- Example: Embedding models
- Encode and represent text
- Encoder-based

---

## Context Length

### Definition
The amount of tokens currently being processed

### Example
**Input:** "Tell me something about llamas"
**Previously Generated Tokens:** Some text already generated
**Current Context Length:** Original query + previously generated tokens

### Maximum Context Length
- Models have a maximum capacity (e.g., 512 tokens)
- Model can only process up to this limit at a given time
- **Important:** Includes tokens being generated (they update current context length)

---

## Scale of Large Language Models

### Parameter Growth

**GPT-1:**
- 100+ million parameters

**GPT-2:**
- 1+ billion parameters

**GPT-3:**
- 175 billion parameters

**Capability Correlation:**
As number of parameters grew, so did capabilities → drives development of large models

---

## The Rise of Modern LLMs

### The Generative AI Breakthrough Year

**Starting Point:**
- **ChatGPT** (more accurately: GPT-3.5)
- Launched the mainstream generative AI era

**Following ChatGPT's Success:**
1. Many proprietary models emerged
2. **Open-source models** followed quickly
   - Weights publicly available
   - Some free for commercial use

---

## Key Takeaways

1. **Transformer Innovation:** Parallel training without RNNs revolutionized the field
2. **Attention is Sufficient:** Entire architecture built on attention mechanism alone
3. **Encoder vs Decoder:** Different architectures for different purposes
4. **BERT (Encoder-Only):** Excellent for representation and understanding tasks
5. **GPT (Decoder-Only):** Designed for generation tasks
6. **Masked Attention:** Critical for preventing information leakage during generation
7. **Self-Attention:** Processes single sequence by comparing to itself
8. **Context Length:** Fundamental constraint on how much text can be processed
9. **Scale Matters:** Larger parameter counts correlate with improved capabilities
10. **Open Source Movement:** Democratization of powerful language models
