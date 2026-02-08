# Tokens and Tokenization

## Tokenization Overview

### Basic Process

**Definition:** Breaking down text into smaller pieces called **tokens**

**Complete Pipeline:**
1. **Input Text:** Raw sentence (e.g., "have the bards who")
2. **Tokenization:** Break text into tokens
3. **Token Embeddings:** Convert to numerical representation (vectors representing semantic nature)
   - **Static embeddings:** Created independent from all other embeddings/tokens
4. **LLM Processing:** Converts to contextualized embeddings
   - **Contextualized embeddings:** One per input token, processed considering all other tokens
5. **Output:** Can be model output or used to create outputs (e.g., next token in generative models)

---

## What Are Tokens?

### Token Types
- **Entire words:** Complete word as single token
- **Word pieces:** Parts of words that combine to form original words

### Why Tokenization Needed
- Tokenizers have **limited vocabulary**
- When encountering unknown words, can represent them using **sub-tokens**

### Token IDs
- Each token has an **associated fixed ID**
- Used to easily encode and decode tokens
- Fed to language model that creates token embeddings
- Generative model output = token ID → decoded to actual token/word

---

## Tokenization Levels

**Example Input:** "Have the bards who precede..." (with special symbol)

### 1. Word Tokens
- Entire sequence represented by complete words
- Each word = one token

### 2. Subword Tokens (Most Common)
- **Flexibility:** Can be entire word OR pieces of word
- **Example:** If "bards" not in vocabulary → split into "b" + "ards"
- **Most LLMs use this level**
- Vocabulary is flexible, represents most words either entirely or using subtokens

### 3. Character Tokens
- One token per character
- Entire input = individual characters

### 4. Byte Tokens (Smallest)
- Used to encode single character of text in computer
- Every character encoded at byte level
- **Note:** Complex symbols need additional bytes (e.g., special characters more complex than single-character)

---

## Practical Implementation

### Setup
```python
pip install transformers  # Package for tokenizers and LLMs
from transformers import AutoTokenizer  # Import for any tokenizer
```

### Basic Tokenization Example

**Input:** "hello world"

**Load Tokenizer:**
```python
tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")
```

**Process Input:**
```python
token_ids = tokenizer(sentence)  # Get token IDs
# Output: numerical values representing tokens
```

**Decode Tokens:**
```python
for token_id in token_ids:
    decoded_token = tokenizer.decode(token_id)
    print(decoded_token)
```

**Output Tokens:**
1. `[CLS]` - Classification token (represents entire input)
2. `hello`
3. `world`
4. `!`
5. `[SEP]` - Separation token (signifies end of sentence)

---

## Special Tokens

### [CLS] Token
- **Classification token**
- Represents entire input
- Used in encoder models like BERT

### [SEP] Token
- **Separation token**
- Signifies end of sentence
- Used in encoder models

### [UNK] Token
- **Unknown token**
- Used when tokenizer doesn't know how to represent a token

### Hashtag Notation (##)
- Indicates token belongs to previous token
- Together they represent a single word
- Example: "capital" + "##ization" = "capitalization"

---

## Comparing Tokenizers

### BERT Tokenizer (bert-base-cased)

**Vocabulary Length:** ~30,000 tokens

**Characteristics:**
- Includes special tokens ([CLS], [SEP])
- Struggles with complex words (many subtokens needed)
- Example: "capitalization" → broken into many pieces
- Uses [UNK] for unknown tokens
- Uses ## to indicate subword continuation

### GPT-4 Tokenizer (Tiktoken)

**Vocabulary Length:** ~100,000 tokens (3x larger than BERT)

**Characteristics:**
- **More efficient:** Needs fewer tokens to represent same input
- "capitalization" → only 2 tokens
- **Generative focused:** No [CLS] or [SEP] tokens
- Better handles uncommon characters (e.g., tabs)
- Represents complex inputs more efficiently

---

## Vocabulary Size Trade-offs

### Larger Vocabulary
**Advantages:**
- Easier to represent uncommon tokens
- Fewer tokens needed per input
- More efficient encoding

**Disadvantages:**
- More embeddings need to be calculated
- Each token requires learning separate representation
- Computational cost increases

### Balance
Trade-off between:
- Having large vocabulary (e.g., million tokens)
- Actually learning quality representations for each token

---

## Exploring Tokenizers

### Visualization Function
```python
# Create color-coded token display
colors = [RGB_values]  # Highlight different tokens

def show_tokens(sentence, tokenizer_name):
    # Load tokenizer
    tokenizer = AutoTokenizer.from_pretrained(tokenizer_name)
    
    # Create token IDs
    token_ids = tokenizer(sentence)
    
    # Extract vocabulary length
    vocab_length = len(tokenizer)
    
    # Decode and display each token with color
    for token_id in token_ids:
        token = tokenizer.decode(token_id)
        # Display with highlighting
```

### HuggingFace Platform
- Find many models and tokenizers online
- Tokenizers relatively small → easy to try out
- Compare different tokenizers:
  - Western vs Eastern languages
  - Different model architectures
  - Various vocabulary sizes

---

## Key Takeaways

1. **Tokenization is Essential:** Converts text to format LLMs can process
2. **Subword Level Optimal:** Most LLMs use subword tokenization for flexibility
3. **Fixed IDs:** Each token mapped to unique identifier
4. **Special Tokens:** Serve specific purposes ([CLS], [SEP], [UNK])
5. **Static → Contextual:** Embeddings start static, become contextual through LLM processing
6. **Vocabulary Size Matters:** Larger = more efficient encoding but higher computational cost
7. **Tokenizer Choice:** Different tokenizers optimized for different purposes (encoding vs generation)
8. **Decoding Required:** Token IDs meaningless until decoded to actual tokens
9. **Subword Benefits:** Handle unknown words by breaking into known pieces
10. **Model-Specific:** Different models (BERT vs GPT) use different tokenization strategies
