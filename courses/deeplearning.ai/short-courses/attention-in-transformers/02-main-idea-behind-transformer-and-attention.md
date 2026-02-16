# Attention in Transformers: Concepts and Code

**Course by:** Josh Starmer (CEO of StatQuest)  
**With:** Andrew Ng

---

## Course Overview

This course covers the attention mechanism - a key breakthrough that led to Transformers - including its historical development, implementation, and role in modern large language models (LLMs).

---

## What is a Transformer?

**Real-World Application:** ChatGPT and similar LLMs are fundamentally based on Transformers.

**Example Interaction:**
- Input: "Tell me about pizza"
- Output: "Pizza is awesome"

---

## Three Fundamental Building Blocks of Transformers

### 1. Word Embedding

**Purpose:** Converts text into numbers for neural network processing

**What it processes:**
- Words
- Bits of words
- Symbols
- Collectively called: **tokens**

**Why needed:** Transformers are neural networks, which only accept numerical input values

**Example:**
- Input text: "tell me about pizza"
- After word embedding: Converted to numerical vectors

---

### 2. Positional Encoding

**Purpose:** Keeps track of word order in sentences

**Why it matters:** Word order dramatically changes meaning

**Examples demonstrating importance:**

| Sentence | Meaning |
|----------|---------|
| "Squatch eats pizza" | Squatch is happy → "Yum!" |
| "Pizza eats Squatch" | Squatch is in danger → "Yikes!" |

**Key Point:** Same words, completely different meanings based on order

**Implementation Note:** Multiple methods exist for positional encoding (details beyond scope of this lesson)

---

### 3. Attention

**Purpose:** Establishes relationships among words in a sentence

#### The Word Association Problem

**Example Sentence:**  
"The pizza came out of the oven and it tasted good"

**Challenge:** What does "it" refer to?
- Option 1: Pizza ✓ (correct - pizzas taste good)
- Option 2: Oven ✗ (incorrect - ovens don't taste good)

**Solution:** Attention mechanism correctly associates "it" with "pizza"

---

## Understanding Self-Attention

**Definition:** The most basic type of attention mechanism

### How Self-Attention Works

**Core Concept:** Calculates how similar each word is to all other words in the sentence (including itself)

**Process:**
1. For each word, calculate similarity with every other word
2. Example with "the pizza came out of the oven and it tasted good":
   - Calculate similarity between "the" and all other words (the, pizza, came, out, of, the, oven, and, it, tasted, good)
   - Repeat for every word in the sentence

**Application of Similarities:**
- Similarity scores determine how each word is encoded by the transformer
- Words with higher similarity scores have larger impact on encoding

**Example in Practice:**
- Across many sentences about pizza:
  - "it" more commonly associated with "pizza" than "oven"
  - Result: Higher similarity score between "it" and "pizza"
  - Outcome: "pizza" has larger impact on how "it" is encoded

---

## Summary: Three Fundamental Components

| Component | Function |
|-----------|----------|
| **Word Embedding** | Converts input text into numbers |
| **Positional Encoding** | Keeps track of word order |
| **Attention** | Establishes relationships among words |

**Note:** Multiple types of attention exist - self-attention is the foundational form

---

## Historical Development of Attention

### Early Machine Translation (Pre-2014)

**Basic Approach:**
- Direct word-by-word translation (e.g., English → French)
- Simply looked up corresponding words in target language

**Limitations:**
1. **Word Order Differences:** English and French have different word orders
   - Example: "the European Economic Area was..." has different word order in French
2. **Length Variations:** Sentences can have different lengths across languages
   - Example: "They arrived late" (3 words in English) = 5 words in French

### Breakthrough: Attention Mechanism (2014)

**Key Research Groups:**
- Yoshua Bengio's group (University of Montreal)
- Chris Manning's group (Stanford University)
- Both independently developed similar approaches

**Innovation: Encoder-Decoder Architecture**

#### The Encoder
- Reads input one word at a time
- Produces output vectors (one per word)
- **Key difference from earlier approaches:** Instead of a single dense vector for entire sentence, vectors for each individual word are preserved
- Creates **contextual embeddings** - word representations that depend on:
  - The word itself
  - Surrounding words (context)

#### The Decoder
- Uses encoder vectors as inputs
- Generates output words one at a time
- **Critical Feature:** Can **weight** or **attend to** each input word embedding independently based on:
  - Position in input sequence
  - Position in output generation

**Example of Attention in Action:**
- When generating 1st French word → attends most heavily to 1st English word
- When generating 2nd French word → might attend to 4th input vector ("area") due to word order changes in French
- Model dynamically focuses on most relevant words for each translation step

---

## The Transformer Revolution (2017)

### "Attention is All You Need" Paper

**Authors:** Ashish Vaswani, Noam Shazeer, Nikki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, Illia Polosukhin  
**Team:** Google Brain

**Key Innovations:**
1. More general form of attention
2. Designed specifically for GPU scalability
   - Primary design criterion: "Can we scale this on a GPU?"
   - This proved to be crucial for future success

### Transformer Architecture

**Components:**
1. **Encoder:** Creates contextual embeddings for input sentence in single pass
2. **Decoder:** Produces output one word at a time
   - Each output feeds back as input for next step
   - Provides context of previously generated words

**Original Scale:** 6 layers of attention  
**Modern Scale Example:** Llama 3.2-405B has 126 layers (same basic architecture)

---

## Legacy and Impact

### BERT (From Encoder)
- **Full Name:** Bidirectional Encoder Representations from Transformers
- **Applications:** 
  - Basis for embedding models
  - Used in RAG (Retrieval-Augmented Generation)
  - Recommender systems

### GPT (From Decoder)
- **Full Name:** Generative Pre-trained Transformer
- **Developed by:** OpenAI
- **Applications:** ChatGPT and similar LLMs
- **Also basis for:**
  - Anthropic models (Claude)
  - Google models
  - Mistral models
  - Meta models (Llama)

---

## Course Structure

1. **Main Ideas** - Core concepts behind Transformers and Attention
2. **Matrix Math and Coding** - Mathematical foundations and implementation
3. **Attention Variants:**
   - Self-attention
   - Masked self-attention
   - PyTorch implementation
4. **Advanced Topics:**
   - Encoder-decoder architecture details
   - Multi-head attention
   - Cross attention

---

## Key Terminology

- **Tokens:** Words, bits of words, and symbols used as input
- **Contextual Embeddings:** Vector representations where meaning depends on both the word and its context
- **Attention/Attending:** The mechanism of weighting or focusing on specific input elements
- **Self-Attention:** Attention mechanism that calculates similarity between all words in a sequence
- **Masked Self-Attention:** Self-attention with certain positions masked/hidden
- **Cross Attention:** Attention between two different sequences (e.g., encoder and decoder)
- **Multi-head Attention:** Multiple attention mechanisms operating in parallel
- **Similarity Scores:** Numerical values representing how related two words are in context

