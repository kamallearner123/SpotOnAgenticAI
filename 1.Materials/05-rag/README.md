# Module 05 — RAG: Retrieval Augmented Generation

**Duration:** 3 Hours | **Sessions:** Weekend 3, Sat (May 31, 2026)

---

## 🎯 Learning Objectives

- Understand why LLMs need external memory and how RAG solves it
- Build a complete RAG pipeline from scratch
- Create a working PDF chatbot using ChromaDB
- Understand chunking strategies and their impact on quality

---

## The Core Problem RAG Solves

```
❌ Without RAG:
User: "What does our company policy say about remote work?"
LLM: "I don't have access to your company's documents..." (or worse — hallucinates)

✅ With RAG:
1. Search your documents for "remote work policy"
2. Find the relevant section
3. Send it to the LLM as context
4. LLM answers based on REAL data
```

---

## The RAG Pipeline

```
INDEXING (one-time):
Documents → Chunks → Embeddings → Vector DB

RETRIEVAL (every query):
User Question → Embed → Search Vector DB → Top K Chunks

GENERATION:
Prompt = System + Retrieved Chunks + User Question → LLM → Answer
```

---

## Topics Covered

1. [Embeddings Deep Dive](#1-embeddings)
2. [Chunking Strategies](#2-chunking)
3. [Vector Databases](#3-vector-databases)
4. [Building the Retrieval Pipeline](#4-retrieval-pipeline)
5. [Hands-on: PDF Chatbot](#5-pdf-chatbot)
6. [Advanced: Hybrid Search](#6-advanced)

---

## 1. Embeddings

Embeddings convert text into vectors (lists of numbers) where **similar text has similar vectors**.

```python
from openai import OpenAI

client = OpenAI()

def embed(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

v1 = embed("Python programming language")
v2 = embed("Software development with Python")
v3 = embed("Cooking Italian pasta")

# v1 and v2 are close; v3 is far away
print(f"Embedding dimensions: {len(v1)}")  # 1536
```

### Cosine Similarity
```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

print(cosine_similarity(v1, v2))  # ~0.93 (very similar)
print(cosine_similarity(v1, v3))  # ~0.15 (very different)
```

---

## 2. Chunking

LLMs have token limits — you can't embed an entire book at once. You must chunk it.

### Fixed-size Chunking
```python
def fixed_chunks(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start = end - overlap  # overlap prevents cutting mid-sentence
    return chunks
```

### Sentence-aware Chunking (better)
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,      # characters per chunk
    chunk_overlap=200,    # overlap between chunks
    separators=["\n\n", "\n", ".", " "]  # try these in order
)

chunks = splitter.split_text(document_text)
print(f"Created {len(chunks)} chunks")
```

### Chunking Strategy Guide

| Strategy | Chunk Size | Best For |
|----------|-----------|----------|
| Small chunks | 200-500 chars | Precise retrieval, Q&A |
| Medium chunks | 500-1000 chars | General documents |
| Large chunks | 1000-2000 chars | Summaries, context |
| Semantic chunks | Variable | Best quality, more complex |

---

## 3. Vector Databases

### ChromaDB (Local, Free, Easy)
```python
import chromadb

# Create client and collection
client = chromadb.Client()
collection = client.create_collection("my_documents")

# Add documents
collection.add(
    documents=["Python is great for AI", "JavaScript is used for web"],
    ids=["doc1", "doc2"],
    metadatas=[{"source": "notes.txt"}, {"source": "web.txt"}]
)

# Query
results = collection.query(
    query_texts=["best language for machine learning"],
    n_results=2
)
print(results["documents"])
```

### FAISS (Facebook AI Similarity Search — Fast)
```python
import faiss
import numpy as np

dimension = 1536  # OpenAI embedding size
index = faiss.IndexFlatL2(dimension)

# Add vectors
vectors = np.array([embed(text) for text in documents], dtype="float32")
index.add(vectors)

# Search
query_vector = np.array([embed("your question")], dtype="float32")
distances, indices = index.search(query_vector, k=5)
```

---

## 4. Retrieval Pipeline

```python
from openai import OpenAI
import chromadb

client = OpenAI()
chroma = chromadb.Client()

def build_index(documents: list[str], collection_name: str):
    """Index documents into ChromaDB"""
    collection = chroma.create_collection(collection_name)

    embeddings = [
        client.embeddings.create(
            model="text-embedding-3-small",
            input=doc
        ).data[0].embedding
        for doc in documents
    ]

    collection.add(
        documents=documents,
        embeddings=embeddings,
        ids=[f"doc_{i}" for i in range(len(documents))]
    )
    return collection

def retrieve(collection, query: str, k: int = 3) -> list[str]:
    """Find most relevant chunks for a query"""
    query_embedding = client.embeddings.create(
        model="text-embedding-3-small",
        input=query
    ).data[0].embedding

    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=k
    )
    return results["documents"][0]

def generate_answer(query: str, context_chunks: list[str]) -> str:
    """Generate answer using retrieved context"""
    context = "\n\n---\n\n".join(context_chunks)

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": "Answer questions using ONLY the provided context. If the answer isn't in the context, say 'I don't know.'"
            },
            {
                "role": "user",
                "content": f"Context:\n{context}\n\nQuestion: {query}"
            }
        ]
    )
    return response.choices[0].message.content
```

---

## 5. PDF Chatbot

**Complete working example:**

```python
from pypdf import PdfReader
from langchain.text_splitter import RecursiveCharacterTextSplitter
import chromadb
from openai import OpenAI
import os

client = OpenAI()
chroma_client = chromadb.Client()

def load_pdf(path: str) -> str:
    reader = PdfReader(path)
    return "\n".join(page.extract_text() for page in reader.pages)

def build_pdf_index(pdf_path: str) -> chromadb.Collection:
    # 1. Load PDF
    text = load_pdf(pdf_path)

    # 2. Chunk
    splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
    chunks = splitter.split_text(text)
    print(f"Loaded {len(chunks)} chunks from PDF")

    # 3. Embed & store
    collection = chroma_client.create_collection("pdf_docs")
    collection.add(
        documents=chunks,
        ids=[f"chunk_{i}" for i in range(len(chunks))]
    )
    return collection

def chat_with_pdf(collection: chromadb.Collection, question: str) -> str:
    # 4. Retrieve
    results = collection.query(query_texts=[question], n_results=3)
    context = "\n\n".join(results["documents"][0])

    # 5. Generate
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Answer based only on the provided context. Be concise."},
            {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {question}"}
        ]
    )
    return response.choices[0].message.content

# --- Main ---
collection = build_pdf_index("company_policy.pdf")

print("PDF loaded! Ask questions (type 'quit' to exit)\n")
while True:
    q = input("You: ").strip()
    if q.lower() == "quit":
        break
    print(f"AI: {chat_with_pdf(collection, q)}\n")
```

---

## 6. Advanced: Hybrid Search

Combine **semantic search** (embeddings) with **keyword search** for better results:

```python
# Semantic: finds similar meaning even with different words
# Keyword: finds exact terms, great for proper nouns, codes, IDs

# ChromaDB supports metadata filtering
results = collection.query(
    query_texts=["refund policy"],
    n_results=5,
    where={"department": "finance"}  # filter by metadata
)
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md) | 🏆 Mini Project 01: PDF Chatbot
