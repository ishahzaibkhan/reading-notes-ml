# Text Splitters in LangChain

## Overview
Text Splitters are a crucial component in building Retrieval-Augmented Generation (RAG) applications. They break down large texts (articles, PDFs, HTML pages, books) into smaller, manageable chunks that Large Language Models can process effectively.

## Why Text Splitting is Important

### 1. Overcoming Model Limitations
- LLMs have context length limits - they can only process a certain amount of input text at once
- Example: An LLM with a 50,000 token limit cannot process a 100,000-word PDF
- Text splitting allows documents exceeding these limits to be processed

### 2. Improved Downstream Task Performance

**Embedding**
- Converts text into numerical vectors
- Embedding large text at once results in lower quality vectors that don't effectively capture semantic meaning
- Smaller chunks capture semantic meaning more effectively

**Semantic Search**
- Searching for relevant documents based on meaning
- Chunking results in more precise and improved search quality compared to searching on whole documents

**Summarization**
- LLMs often struggle with summarizing very large documents
- They may "drift" off-topic or "hallucinate" information not present in the document
- Text splitting is empirically proven to yield better summarization results

### 3. Optimizing Computational Resources
- Processing smaller chunks is more memory-efficient
- Allows for better parallelization of processing tasks
- Reduces computational requirements

## Types of Text Splitters

### 1. Length-Based Text Splitting (CharacterTextSplitter)

**How It Works**
- Simplest and fastest method
- Pre-define the chunk size (in characters or tokens)
- Text is traversed and chunks are created when the defined limit is reached

**Advantages**
- Very simple and easy to implement
- Fast processing

**Disadvantages**
- Does not consider linguistic structure, grammar, or semantic meaning
- Chunks can cut in the middle of words, sentences, or paragraphs
- Leads to incomplete semantic meaning
- Not widely used due to these limitations

**Implementation**
```python
from langchain.text_splitter import CharacterTextSplitter

# Create splitter object
splitter = CharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=0,
    separator=""
)

# For plain text
chunks = splitter.split_text(your_text)

# For Document objects
chunks = splitter.split_documents(your_documents)
```

### 2. Chunk Overlap

**Purpose**
- Defines how many characters/tokens two adjacent chunks will share
- Ensures context is not lost when chunks are cut mid-statement
- Provides continuity across chunks

**Trade-offs**
- Higher overlap = more chunks = increased computational requirements

**Recommended Values**
- For RAG-based applications: 10% to 20% of the chunk size

### 3. Text-Structure Based Text Splitting (RecursiveCharacterTextSplitter)

**Overview**
- One of the most used text splitting techniques
- Considers the inherent linguistic structure of text (paragraphs, sentences, words, characters)

**How It Works**
- Breaks text using a predefined list of separators in a recursive manner
- Prioritizes breaking at higher-level structures (paragraphs) before moving to lower-level ones (lines, words, characters)
- Default separators: `["\n\n", "\n", " ", ""]` (paragraphs, lines, words, characters)
- Tries to avoid abrupt breaks in text
- Optimizes by merging smaller chunks if they fit within the allowed chunk_size

**Advantages**
- Much better than CharacterTextSplitter
- Preserves semantic meaning by splitting at natural breaks

**Implementation**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Create splitter object
splitter = RecursiveCharacterTextSplitter(
    chunk_size=your_size,
    chunk_overlap=your_overlap
)

# Split text
chunks = splitter.split_text(your_text)
```

**Behavior**
- Smaller chunk_size leads to word-level splitting
- Larger chunk_size leads to paragraph-level splitting
- Most likely to be used in future applications

### 4. Document-Structure Based Text Splitting (Specialized RecursiveCharacterTextSplitter)

**Overview**
- Extension of Text-Structure Based Text Splitting
- Designed for non-plain text documents with specific structured formats

**Use Cases**

**Code (e.g., Python)**
- Organized by keywords like `class`, `def`, loops
- Uses specialized separators: `class`, `def`, etc., followed by normal separators (paragraph, line, word, character)

**Markdown**
- Markup language with headings, lists, etc.
- Aims to create chunks based on Markdown headings and sections

**Goal**
- Respects the document's inherent structural hierarchy
- Uses specialized separators with RecursiveCharacterTextSplitter

### 5. Semantic Meaning Based Text Splitting

**Overview**
- Most advanced text splitting technique
- Uses embeddings and semantic similarity to determine optimal split points
- Focuses on preserving coherent semantic units rather than relying on structural markers

**How It Works**
- Generates embeddings for sentences or text segments
- Calculates semantic similarity between adjacent segments
- Splits text where semantic similarity drops significantly (indicating topic changes)
- Groups semantically similar content together into chunks

**Advantages**
- Creates chunks with maximum semantic coherence
- Better preserves context and meaning
- Particularly effective for dense, technical, or nuanced content
- Improves retrieval accuracy in RAG applications

**Disadvantages**
- Computationally more expensive (requires embedding generation)
- Slower than structure-based methods
- Requires an embedding model

**Implementation**
```python
from langchain.text_splitter import SemanticChunker
from langchain.embeddings import OpenAIEmbeddings

# Create embeddings model
embeddings = OpenAIEmbeddings()

# Create semantic splitter
splitter = SemanticChunker(
    embeddings=embeddings,
    breakpoint_threshold_type="percentile"
)

# Split text
chunks = splitter.split_text(your_text)
```

**Use Cases**
- High-stakes applications where semantic accuracy is critical
- Technical documentation with complex interconnected concepts
- Legal or medical documents requiring precise context preservation
- Research papers and academic content

## Best Practices

1. **Choose the right splitter**: 
   - RecursiveCharacterTextSplitter for most general applications
   - SemanticChunker for applications requiring maximum semantic accuracy
   - Specialized splitters for code and markup documents

2. **Set appropriate chunk_overlap**: 10-20% of chunk size for RAG applications

3. **Consider performance trade-offs**:
   - Length-based: Fastest but lowest quality
   - Structure-based: Good balance of speed and quality
   - Semantic-based: Highest quality but most resource-intensive

4. **Match splitter to content type**:
   - Plain text: RecursiveCharacterTextSplitter
   - Code: Document-structure based with specialized separators
   - Complex semantic content: SemanticChunker

5. **Balance chunk size with use case**: Consider computational resources, model context limits, and retrieval precision requirements
