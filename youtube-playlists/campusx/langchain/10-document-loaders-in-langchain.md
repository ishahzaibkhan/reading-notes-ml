# Document Loaders in LangChain

## Overview

This guide covers Document Loaders, a fundamental component for building RAG (Retrieval Augmented Generation) applications using LangChain.

---

## What is RAG?

**RAG (Retrieval Augmented Generation)** solves key limitations of standard LLMs by providing an external knowledge base.

### Limitations of Standard LLMs

1. **Lack of Current Information**
   - LLMs are trained on past data and don't have up-to-date information
   - Cannot answer questions about recent events or current affairs

2. **No Access to Personal/Private Data**
   - Cannot answer questions about personal emails or company documentation
   - Haven't been trained on private data sources

3. **Document Size Limitations**
   - Context length limits prevent processing very large documents (e.g., 1 GB files)

### How RAG Solves These Problems

- Provides an **external knowledge base** to the LLM (company databases, PDFs, personal documents, etc.)
- When a user asks a question, the LLM **retrieves information** from this knowledge base
- Uses retrieved information as **context** to generate accurate and grounded responses

### Benefits of RAG

- ✅ Provides up-to-date information
- ✅ Ensures privacy for confidential data (query personal documents without uploading to public LLMs)
- ✅ No limit on document size (large documents can be divided into chunks)

---

## RAG Components

The four most important components of RAG:

1. **Document Loaders** - Load documents from various sources
2. **Text Splitters** - Break down large texts into smaller chunks
3. **Vector Databases** - Store and retrieve document embeddings
4. **Retrievers** - Fetch relevant documents from the vector database

---

## Document Loaders - Detailed Definition

**Document Loaders** are components in LangChain used to load data from various sources into a standardized format, usually as Document objects, which can then be used for chunking, embedding, retrieval, and generation.

### Purpose

- Data exists in various sources (PDFs, text files, databases, cloud providers)
- Document Loaders ensure data from any source is loaded into a **common, standardized format**

### Standardized Format: Document Object

Each **Document object** has two main parts:

- **`page_content`** - The actual content of the data
- **`metadata`** - Additional information (source, creation date, modification date, author name, etc.)

Document Loaders are utilities that fetch data from different sources and convert it into this standardized Document object format.

---

## 1. Text Loader

The simplest document loader in LangChain.

### Function
Takes text files and loads them as Document objects into LangChain.

### Use Cases
- Log files
- Code snippets
- Transcripts (e.g., YouTube video transcripts)

### Code Example
```python
from langchain_community.document_loaders import TextLoader

# Create loader object
loader = TextLoader('cricket.txt', encoding='utf-8')

# Load document
docs = loader.load()

# Access document content
print(len(docs))  # Number of documents
print(docs[0])  # First document
print(docs[0].page_content)  # Actual content
print(docs[0].metadata)  # Metadata
```

### Key Points

- Returns a **list of Document objects** (even for a single text file, it returns a list with one Document)
- `encoding` parameter is optional but often needed for special characters
- Can be integrated directly with LLM chains for processing

---

## 2. PyPDFLoader

Loads content from PDF files.

### Function
- Returns a list of Document objects
- **Each page** typically becomes a separate Document object
- Each Document has its own `page_content` and `metadata` (including page number and source)

### Internal Mechanism
Uses the **PyPDF** Python library to read PDF files.

### Limitations
- Not great with scanned PDFs or complex layouts
- Best for **simple textual PDFs**

### Code Example

```python
# Pre-requisite: pip install pypdf
from langchain_community.document_loaders import PyPDFLoader

# Create loader object
loader = PyPDFLoader('DL_Curriculum.pdf')

# Load documents
docs = loader.load()

# Access content
print(len(docs))  # Number of pages
print(docs[0].page_content)  # First page content
print(docs[0].metadata)  # Metadata with page info
```

### Metadata Includes
- Producer, creator, creation date, title
- Source (filename)
- Total pages and current page number

---

## Other PDF Loaders

PyPDFLoader is not the only PDF loader in LangChain. For different scenarios:

| Loader | Use Case |
|--------|----------|
| **PDFPlumberLoader** | PDFs with tabular structures (extract table data) |
| **UnstructuredPDFLoader** | Scanned images in PDFs, structure extraction |
| **AmazonTextractPDFLoader** | Scanned images in PDFs |
| **PyMuPDF** | PDFs with complex layouts |

**Recommendation:** Understand the basic concept and refer to LangChain documentation for specific project needs.

---

## 3. Directory Loader

Loads multiple documents from a single folder simultaneously.

### Function
Helps to load multiple documents from within a directory.

### Code Example

```python
from langchain_community.document_loaders import DirectoryLoader, PyPDFLoader

# Create loader object
loader = DirectoryLoader(
    path='./books',
    glob='*.pdf',
    loader_cls=PyPDFLoader
)

# Load documents
docs = loader.load()
```

### Parameters

- **`path`** - The path to the directory (e.g., `./books`)
- **`glob`** - A pattern to specify which files to load
  - `*.pdf` - Load all PDF files
  - `*.txt` - Load all text files
  - `**/*.pdf` - Load PDFs from subfolders
- **`loader_cls`** - The specific document loader class to use (e.g., `PyPDFLoader` for PDFs)

### Key Points

- Can load multiple documents simultaneously
- Works with other document loaders, not just PyPDFLoader
- Example: 3 PDFs with 326 + 392 + 468 pages = 1186 Document objects
- Metadata includes filename (source) and page number

---

## Load vs Lazy Load

### Problem with `load()` (Eager Loading)

- Loading multiple large documents takes significant time
- **All documents loaded into memory (RAM) simultaneously**
- Problematic for hundreds or thousands of large files

### Solution: Lazy Loading

| Feature | `load()` (Eager Loading) | `lazy_load()` (Lazy Loading) |
|---------|--------------------------|------------------------------|
| **Loading Method** | Loads everything at once | Loads documents on demand (one at a time) |
| **Return Type** | List of Document objects | Generator of Document objects |
| **Memory Usage** | High - all in memory | Low - one at a time |
| **Best For** | Small number of documents | Large number or very large files |
| **Processing** | All loaded first, then processed | Load → Process → Remove → Next |

### Code Example

```python
# Eager Loading
docs = loader.load()
for doc in docs:
    print(doc.metadata)  # Delay at start, then fast

# Lazy Loading
docs_generator = loader.lazy_load()
for doc in docs_generator:
    print(doc.metadata)  # Immediate processing, one by one
```

**Use lazy_load() for:**
- Large number of documents
- Very large files
- Stream processing without consuming too much memory

---

## 4. WebBaseLoader

Loads content from web pages into LangChain.

### Function
Loads and extracts text content from web pages.

### Use Cases
- Static websites
- Blogs
- News articles
- Public websites

### Internal Mechanism

Uses two Python libraries:
- **Requests** - Making HTTP requests to the web page
- **BeautifulSoup** - Parsing HTML structure and extracting text content

### Limitation
Works better with **static pages**.

---

## 5. CSVLoader

Loads data from CSV files. Each row in the CSV typically becomes a separate document.

---

## Other Document Loaders

LangChain has hundreds of document loaders for various data sources:

- **Databases** - SQL, MongoDB
- **Cloud Storage** - S3, Google Drive
- **Productivity Tools** - Notion
- **Media** - YouTube
- And many more...

**The principle remains the same for all loaders.**

---

## Custom Document Loader

If no existing loader fits your specific data source or format, you can create **Custom Document Loaders** by:

1. Implementing the necessary logic to read your data
2. Converting it into LangChain's Document object format
   - `page_content` - Your data content
   - `metadata` - Relevant metadata

---

## Summary

- **Document Loaders** standardize data from various sources into Document objects
- Each Document has `page_content` and `metadata`
- Choose the appropriate loader based on your data source
- Use `lazy_load()` for large datasets to optimize memory
- Hundreds of pre-built loaders available in LangChain
- Custom loaders can be created for unique requirements

---

## Resources

- LangChain documentation for complete list of loaders
- Tutorials on extracting structured data from PDFs
- Specific use cases for different PDF loaders
