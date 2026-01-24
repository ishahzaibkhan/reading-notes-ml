# Retrievers in LangChain

## Overview
Retrievers are crucial components for building Retrieval Augmented Generation (RAG) applications in LangChain. They fetch relevant documents from data sources in response to user queries.

## Core Components of RAG (In Order)
1. Document Loaders
2. Text Splitters
3. Vector Stores
4. **Retrievers** (Current Topic)

## What are Retrievers?

A retriever is a component that fetches relevant documents from a data source based on a user's query.

### How Retrievers Work
1. Data source (Vector Store, API, etc.) holds all data
2. User sends a query to the system
3. Retriever receives query and searches the data source
4. Retriever identifies most relevant documents
5. Returns relevant documents as output

### Function Analogy
- **Input**: User's query
- **Output**: Multiple LangChain Document objects
- **Internal Process**: Acts like a search engine to find relevant results

### Key Characteristics
- Multiple retrievers available in LangChain for different use cases
- All LangChain retrievers are **runnables**
  - Can be used to form chains
  - Can be plugged into existing chains
  - Enhances system flexibility for RAG applications

## Types of Retrievers

Retrievers are categorized based on two main criteria:

### 1. Based on Data Source
Different retrievers work with different types of data sources:

- **Wikipedia Retriever**: Searches and retrieves articles from Wikipedia API
- **Vector Store Based Retrievers**: Searches documents within vector stores using semantic search
- **ArXiv Retriever**: Scans research papers on ArXiv website

### 2. Based on Search Strategy/Retrieval Mechanism
Different retrievers use different mechanisms to search for documents:

- **Maximum Marginal Relevance (MMR)**
- **Multi Query Retriever**
- **Contextual Compression Retriever**

## Wikipedia Retriever

### Functionality
- Queries Wikipedia API to fetch relevant content for a given query
- Uses keyword matching internally to determine most relevant articles

### Implementation
```python
# Installation
!pip install langchain wikipedia

# Import
from langchain_community.retrievers import WikipediaRetriever

# Initialization
retriever = WikipediaRetriever(
    top_k_results=2,  # Number of documents to retrieve
    lang="en"         # Language specification
)

# Usage
docs = retriever.invoke("The geopolitical history of India and Pakistan from the perspective of a Chinese")

# Output: LangChain Document objects with page content and metadata
for doc in docs:
    print(doc.page_content)
```

### Key Distinction
Wikipedia Retriever is a **retriever**, not just a document loader, because it:
- Performs internal search logic (keyword matching)
- Finds relevant documents based on query
- Rather than just loading all articles

## Vector Store Retriever

### Functionality
The most common type of retriever that searches documents from a vector store based on semantic similarity using vector embeddings.

### How It Works
1. Documents are stored in a vector store (FAISS, Chroma, Weaviate, etc.)
2. Each document is converted into a dense vector (embedding) using an embedding model
3. User query is also converted into a vector
4. Query vector is compared with all document vectors (semantic search)
5. Returns the most similar (top-k) results

### Implementation
```python
# Imports
from langchain_community.vectorstores import Chroma, FAISS
from langchain_openai import OpenAIEmbeddings
from langchain.docstore.document import Document

# Sample Documents
docs = [
    Document(page_content="LangChain information..."),
    Document(page_content="Chroma information..."),
    Document(page_content="Embeddings information...")
]

# Create embeddings
embeddings = OpenAIEmbeddings()

# Create Vector Store
vectorstore = Chroma.from_documents(
    documents=docs,
    embedding=embeddings
)

# Create Retriever
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 2}  # Number of results to return
)

# Query
results = retriever.invoke("What is Chroma used for")

# Display results
for result in results:
    print(result.page_content)
```

### Why Use `as_retriever()` Instead of `similarity_search()`?

While `similarity_search()` can perform semantic search directly, `as_retriever()` offers:
- **Standardized interface** (runnable)
- **Advanced search strategies** not available in `similarity_search()`
- **Essential for chaining** and complex RAG applications

## Maximum Marginal Relevance (MMR) Retriever

### Problem MMR Solves
Normal similarity search often returns redundant or very similar documents, lacking diversity.

**Example**: Query "What are the adverse effects of climate change?" might return multiple documents about melting glaciers, ignoring other effects like deforestation or wildfires.

### Core Philosophy
Pick results that are:
- Relevant to the query
- Different from each other
- Reduces redundancy while maintaining high relevance

### How MMR Works
1. First, picks the most relevant document
2. For subsequent picks, selects documents that are:
   - Relevant to the query
   - Dissimilar to already selected documents
3. Ensures diverse perspectives in results

### Implementation
```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# Create sample documents (including some redundant ones)
docs = [
    Document(page_content="LangChain information..."),
    Document(page_content="Chroma information..."),
    Document(page_content="Embedding information..."),
    Document(page_content="MMR information..."),
    Document(page_content="Another LangChain statement...")  # Redundant
]

# Create FAISS vector store
vectorstore = FAISS.from_documents(
    documents=docs,
    embedding=embeddings
)

# Create MMR Retriever
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,
        "lambda_mult": 1  # Controls relevance vs. diversity balance
    }
)

# Query
results = retriever.invoke("What is LangChain")
```

### Lambda_mult Parameter
Controls the balance between relevance and diversity:

- **lambda_mult = 1**: 
  - Behaves like normal similarity search
  - High relevance, low diversity
  
- **lambda_mult = 0**: 
  - Provides very diverse results
  - High diversity, potentially lower relevance
  
- **lambda_mult = 0.5**: 
  - Balanced approach
  - Good mix of relevance and diversity

**Example Results**:
- With `lambda_mult = 1`: Query "What is LangChain" returns three similar LangChain documents
- With `lambda_mult = 0.5`: Same query returns diverse results including LangChain and embeddings documents

## Multi Query Retriever

### Problem Multi Query Retriever Solves
User queries can be ambiguous or too broad, leading to poor quality search results.

**Example**: "How can I stay healthy?" could mean diet, exercise, or stress management.

### Core Philosophy
Reduce ambiguity by generating multiple diverse (but related) queries from a single ambiguous query.

### How It Works
1. User sends an ambiguous query (e.g., "How can I stay healthy?")
2. Query is sent to an LLM
3. LLM generates multiple sub-queries exploring different interpretations:
   - "What are the best foods for health?"
   - "How often should I exercise?"
   - "What lifestyle habits improve wellbeing?"
4. Each sub-query is sent to a normal retriever (e.g., similarity search)
5. Results from all retrievers are merged
6. Duplicates are removed
7. Top relevant and diverse results are presented

### Benefits
Provides comprehensive results by covering different facets of the initial ambiguous query.

### Implementation
```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.docstore.document import Document
from langchain.retrievers.multi_query import MultiQueryRetriever

# Create sample documents (health-related and some random)
docs = [
    Document(page_content="Health and nutrition information..."),
    Document(page_content="Exercise guidelines..."),
    Document(page_content="The Solar System in Modern Homes..."),
    Document(page_content="Python programming..."),
    Document(page_content="Photosynthesis process..."),
    Document(page_content="FIFA World Cup history...")
]

# Create vector store
vectorstore = FAISS.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings()
)

# Create standard retriever (baseline)
vanilla_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 5}
)

# Create LLM for generating sub-queries
llm = ChatOpenAI(temperature=0)

# Create Multi Query Retriever
multi_query_retriever = MultiQueryRetriever.from_llm(
    retriever=vanilla_retriever,
    llm=llm
)

# Query
query = "How to improve energy levels and maintain balance"

# Compare results
vanilla_results = vanilla_retriever.invoke(query)
multi_query_results = multi_query_retriever.invoke(query)
```

### Performance Comparison

**Vanilla Retriever Results**:
- Retrieves mostly health-related documents
- May include irrelevant documents like "The Solar System in Modern Homes"
- Confused by keyword matches ("energy system," "balance")

**Multi Query Retriever Results**:
- All results are strictly health and nutrition-related
- LLM generates precise sub-queries
- Internal retriever fetches only truly relevant documents
- Successfully overcomes query ambiguity

## Contextual Compression Retriever

### Definition
An advanced retriever that improves retrieval quality by compressing documents after retrieval, keeping only relevant content based on the user's query.

### Problem It Solves
Retrieved documents often contain both relevant and irrelevant information.

**Example**: 
- Document discusses both Grand Canyon and photosynthesis
- Query: "What is photosynthesis?"
- Standard retriever: Returns entire document (including Grand Canyon info)
- Contextual Compression: Returns only photosynthesis-related content

### How It Works
- Analyzes retrieved documents
- Identifies relevant portions based on query
- Returns only the relevant parts
- Ignores irrelevant content
- Provides concise and focused answers

### Benefits
- Enhances user experience
- Provides focused, relevant information
- Reduces noise in retrieved results
- Improves answer quality

## Summary

| Retriever Type | Primary Purpose | Key Benefit |
|---------------|-----------------|-------------|
| **Wikipedia** | Search Wikipedia API | Access to encyclopedic knowledge |
| **Vector Store** | Semantic similarity search | Most common, flexible retrieval |
| **MMR** | Diverse yet relevant results | Reduces redundancy |
| **Multi Query** | Handle ambiguous queries | Comprehensive coverage |
| **Contextual Compression** | Extract relevant portions | Concise, focused answers |

## Best Practices

1. **Choose the right retriever** based on your data source and use case
2. **Use `as_retriever()`** instead of direct similarity search for:
   - Standardized interface
   - Advanced search strategies
   - Chain compatibility
3. **Adjust parameters** (k, lambda_mult) based on your needs
4. **Combine retrievers** for complex RAG applications
5. **Test different strategies** to find optimal configuration for your use case
