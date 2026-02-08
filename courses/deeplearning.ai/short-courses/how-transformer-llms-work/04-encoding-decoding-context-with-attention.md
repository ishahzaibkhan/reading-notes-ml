# Contextual Embeddings and Attention Mechanism

## Limitations of Static Embeddings

### The Context Problem
**Example:** The word "bank"
- Can refer to a **financial bank**
- Can refer to the **bank of a river**
- **Problem:** Word2Vec generates the same embedding regardless of context
- **Solution Needed:** Embedding should change based on context

### Why Context Matters
Essential for language tasks such as **translation**

---

## Recurrent Neural Networks (RNNs)

### Advancement
Step toward encoding text context through sequence modeling

### RNN Capabilities
- Variant of neural networks
- Can model sequences as additional input
- Used for two tasks:
  1. **Encoding:** Representing an input sentence
  2. **Decoding:** Generating an output sentence

### Example: Translation Process
**Input:** "I love llamas" (English)
**Output:** "Ik hou van lama's" (Dutch)

**Process:**
1. Text passed to **encoder**
2. Encoder represents entire sequence through embeddings
3. **Decoder** uses those embeddings to generate language

---

## Autoregressive Generation

### Definition
Architecture generates text one token at a time, consuming all previously generated words

### Step-by-Step Process

**Step 1:**
- Input: "I love llamas"
- Output: "Ik"

**Step 2:**
- Input: "I love llamas Ik" (previous output appended)
- Output: "hou"

**Step 3:**
- Input: "I love llamas Ik hou"
- Output: "van"

**Continue:** Process repeats, continuously updating inputs with previously generated tokens until entire output is complete

**Note:** Most models are autoregressive and generate a single token each time

---

## Encoder-Decoder Architecture Details

### Encoding Process

**Step 1: Tokenization**
- Input sentence: "I love llamas" → tokens

**Step 2: Create Embeddings**
- Use Word2Vec to create embeddings as inputs
- Although embeddings are static individually...

**Step 3: Contextual Processing**
- Encoder processes entire sequence in one go
- Takes into account the context of embeddings
- **Goal:** Represent input as well as possible
- **Output:** Context in the form of an embedding

### Decoding Process
- Decoder generates language
- Leverages previously generated context embedding
- Generates output tokens autoregressively (one at a time)

---

## Problem with Context Embeddings

### The Bottleneck Issue
- Context embedding = single embedding representing entire input
- **Challenge:** Difficult to deal with longer sentences
- **Failure Point:** Single embedding might fail to capture entire context of long, complex sequences

---

## Attention Mechanism (2014)

### Revolutionary Solution
Highly improved upon original RNN architecture

### Core Concept
**Attention allows the model to:**
- Focus on parts of the input sequence that are relevant to one another
- "Attend" to each other
- Amplify their signal
- Selectively determine which words are most important in a sentence

### How Attention Works

**Attention Weights:**
- **High attention weights:** Words with similar meaning or strong relationships
  - Example: "I" and "Ik" (synonyms) have higher attention weights
- **Low attention weights:** Words that don't relate much to each other
  - Example: "I" and "llamas" in this sentence

### Architecture Enhancement

**Traditional RNN Decoder:**
- Only passes context embedding to decoder

**RNN with Attention:**
1. Input represented using Word2Vec embeddings
2. Passed to encoder
3. **Key Difference:** Hidden states of ALL input words passed to decoder (not just context embedding)

**Hidden State Definition:**
- Internal vector from a hidden layer of an RNN
- Contains information about previous words

**Decoder with Attention:**
- Uses attention mechanism to look at entire sequence
- Generates language based on full context

### Benefits
- **Better output quality:** Looks at entire sequence
- Uses embeddings for each token/word
- Replaces limited single context embedding

### Attention During Generation

**Example:** After generating "Ik hou van" from "I love llamas"
- Attention mechanism highlights most relevant input words
- Model attends to most relevant inputs for next token

---

## Limitation of RNN Architecture

### Sequential Processing Problem
- **Issue:** Sequential nature precludes parallelization during training
- Cannot process tokens simultaneously
- Limits training efficiency

**Preview:** Next lesson covers how transformers use attention to overcome this limitation

---

## Key Takeaways

1. **Context is Critical:** Same word needs different embeddings in different contexts
2. **RNNs Added Context:** First step toward contextual understanding
3. **Autoregressive Generation:** Standard approach for text generation (one token at a time)
4. **Context Bottleneck:** Single context embedding fails for long sequences
5. **Attention Mechanism:** Revolutionary solution that allows selective focus on relevant parts
6. **Hidden States:** Carry information about sequence through processing
7. **Parallelization Challenge:** Sequential nature limits RNN efficiency
8. **Attention > Context Embedding:** Using full sequence attention produces better results than single context vector
