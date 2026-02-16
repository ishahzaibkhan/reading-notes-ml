# Attention in Transformers: Concepts and Code

**Course by:** Josh Starmer (CEO of StatQuest)  
**With:** Andrew Ng

---

## Course Overview

This course covers the attention mechanism - a key breakthrough that led to Transformers - including its historical development, implementation, and role in modern large language models (LLMs).

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

- **Contextual Embeddings:** Vector representations where meaning depends on both the word and its context
- **Attention/Attending:** The mechanism of weighting or focusing on specific input elements
- **Self-Attention:** Attention mechanism applied within a single sequence
- **Masked Self-Attention:** Self-attention with certain positions masked/hidden
- **Cross Attention:** Attention between two different sequences (e.g., encoder and decoder)
- **Multi-head Attention:** Multiple attention mechanisms operating in parallel

---

## Course Credits

**Instructors:** Josh Starmer, Andrew Ng  
**Contributors:** Geoff Ladwig, Esmaeil Gargari, Hawraa Salami
