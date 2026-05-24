# Module 05 — RAG: Retrieval Augmented Generation

**Duration:** 3 Hours | **Sessions:** Weekend 3, Sat (May 31, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Master the RAG Architecture:** Design end-to-end Retrieval Augmented Generation pipelines from first principles.
* **Optimize Chunking Pipelines:** Compare recursive, token-based, and semantic chunking strategies, adjusting sizes and overlaps for optimal performance.
* **Navigate Vector Databases:** Understand vector indexing structures (HNSW, IVF, Flat) and select appropriate distance metrics (Cosine, Dot Product, $L_2$).
* **Implement Advanced Retrieval:** Integrate Hybrid Search (Sparse BM25 + Dense Vector) and Cross-Encoder Reranking to maximize precision.
* **Quantitatively Evaluate Pipelines:** Set up RAGAS-inspired evaluation frameworks to monitor Faithfulness, Answer Relevance, and Context Recall.

---

## 🗺️ Topics Covered

1. [The Architectural Paradigm: Grounding vs. Fine-Tuning](#1-the-architectural-paradigm-grounding-vs-fine-tuning)
2. [Document Ingestion and Advanced Chunking Strategies](#2-document-ingestion-and-advanced-chunking-strategies)
3. [Vector Databases: Indexing Mechanics and Math](#3-vector-databases-indexing-mechanics-and-math)
4. [Advanced Retrieval: Hybrid Search and Cross-Encoder Reranking](#4-advanced-retrieval-hybrid-search-and-cross-encoder-reranking)
5. [The Generation Phase: Proven Prompt Grounding Patterns](#5-the-generation-phase-proven-prompt-grounding-patterns)
6. [RAG Evaluation: Faithfulness, Relevance, and Recall](#6-rag-evaluation-faithfulness-relevance-and-recall)

---

## 1. The Architectural Paradigm: Grounding vs. Fine-Tuning

Large Language Models excel at reasoning, but their knowledge is frozen at their training cutoff date. When asked about custom company files, new public developments, or highly specific database keys, standard models either fail or hallucinate plausible-sounding lies.

To solve this, developers use two primary strategies:
* **Fine-Tuning:** Backpropagating new knowledge directly into the neural network's parameter weights. 
  * *Trade-off:* Slow, expensive, complex, prone to catastrophic forgetting, and cannot enforce strict document-level security.
* **Retrieval Augmented Generation (RAG):** Retrieving highly relevant factual sections from external documents dynamically at runtime, injecting them directly into the context window, and instructing the model to synthesize the final answer based *only* on the retrieved context.

### Complete RAG Pipeline Architecture

```
1. INGESTION PIPELINE (Offline / Event-Driven):
Raw Files (PDFs/Web) ──► Load & Parse ──► Chunking ──► Embedding Model ──► Vector Database
                                                                           (HNSW Index)

2. RETRIEVAL & GENERATION PIPELINE (Runtime):
User Query ─────────────────► [Embedding Model] ───────────────────┐
                                   │                               ▼
                                   │                           Vector Search (Top K)
                                   ▼                               ▼
User Query ─────────────────► [Reranker] ◄──────────────────── Retrieved Chunks
                                   │
                                   ▼ (Top P Reranked Chunks)
Prompt Assembler (System Instructions + Grounded Context + Query) ──► LLM ──► Answer
```

---

## 2. Document Ingestion and Advanced Chunking Strategies

A RAG pipeline is only as good as the context it retrieves. If your chunking strategy cuts a crucial sentence or formula in half, the meaning is lost.

### Chunking Strategies Analysis

```
Recursive Chunking:
"This is a long sentence. We split it on separators dynamically."
├── Chunk 1: "This is a long sentence."
└── Chunk 2: "We split it on separators dynamically."

Semantic Chunking:
"Sentence A. Sentence B. [Similarity Gap] Sentence C."
├── Chunk 1: "Sentence A. Sentence B." (Semantically aligned)
└── Chunk 2: "Sentence C." (Focus shifts to a new topic)
```

#### 1. Character-Based / Fixed-Size Chunking
* **Method:** Splits text into a set number of characters (e.g., 500 characters) regardless of word boundaries.
* **Pros:** Extremely fast and simple.
* **Cons:** Brittle. Frequently cuts words, code blocks, or sentences in half, causing significant semantic fragmentation.

#### 2. Recursive Character Chunking
* **Method:** Iteratively splits text based on a hierarchical list of separators (typically `["\n\n", "\n", ".", " ", ""]`). It attempts to keep paragraphs, then sentences, and finally words together in a single chunk.
* **Pros:** Highly reliable and the default standard for general text documents.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=800,      # Target character count per chunk
    chunk_overlap=150,   # Overlap characters to prevent loss of context at boundaries
    separators=["\n\n", "\n", ". ", "? ", "! ", " ", ""]
)
chunks = splitter.split_text(raw_document_content)
```

#### 3. Semantic Chunking
* **Method:** Rather than using arbitrary character counts, semantic chunking splits the document into sentences. It embeds each sentence and calculates the cosine distance between consecutive sentences. If the distance exceeds a specific threshold, it triggers a chunk boundary, ensuring each chunk focuses on a single, coherent topic.
* **Pros:** Exceptional retrieval quality. Excellent for complex technical articles.
* **Cons:** Slower. Requires calling the embedding model for every sentence during ingestion.

---

## 3. Vector Databases: Indexing Mechanics and Math

A **Vector Database** is a database designed to index, store, and query high-dimensional vector representations fast. 

### Core Indexing Mechanisms
Linear search (comparing a query vector against millions of document vectors one-by-one) is far too slow for production systems. Vector databases use approximate nearest neighbor (ANN) indexes to trade a tiny amount of accuracy for orders-of-magnitude speedups.

```
       HNSW Graph Index:
       (Layer 2 - Sparsest)   O ───────────── O
                               │             │
       (Layer 1 - Moderate)   O ─── O ─────── O ─── O
                               │     │       │     │
       (Layer 0 - Dense)      O ─ O ─ O ─ O ─ O ─ O ─ O
```

* **Flat Index (Exact Search):** No index structure is built. Performs a raw linear scan.
  * *Use Case:* High-accuracy requirements on small datasets ($< 10,000$ items).
* **IVF (Inverted File Index):** Groups vectors into clusters using k-means. Queries are compared only against the centroids of the nearest clusters.
  * *Use Case:* Large datasets with tight memory limits.
* **HNSW (Hierarchical Navigable Small World):** Builds a multi-layered graph index. High layers have sparse connections for fast traversal across clusters; lower layers contain dense connections for precise local searches.
  * *Use Case:* The industry standard for high-performance production vector search.

### Math of Distance Metrics
Ensure your database index distance metric matches the training objective of your embedding model:

* **Cosine Similarity:** Focuses purely on direction rather than magnitude. Excellent for varying document lengths.
* **Inner Product (Dot Product):** Extremely fast if vectors are pre-normalized to unit length.
* **Euclidean ($L_2$) Distance:** Measures the straight-line distance between vector coordinates. Highly sensitive to vector scale.

---

## 4. Advanced Retrieval: Hybrid Search and Cross-Encoder Reranking

Standard semantic vector search is excellent at capturing abstract concepts, but it can fail on exact terms, product codes, SKU numbers, or specific function names (e.g., searching for `CVE-2026-1011` might retrieve general security articles rather than the specific vulnerability report).

### Hybrid Search (Dense + Sparse)
To build a highly resilient search engine, we combine:
* **Dense Retrieval (Semantic Search):** Captures high-level concepts and synonyms using vector embeddings.
* **Sparse Retrieval (Keyword Search / BM25):** Matches exact strings, product IDs, and proper nouns using term frequency metrics.

We combine these results using algorithms like **Reciprocal Rank Fusion (RRF)** to generate a unified, high-precision ranking list.

```
User Query: "How to fix CVE-2026-4412"
  │
  ├──► Sparse Search (BM25) ──► [doc_A: Rank 1, doc_B: Rank 2] ────┐
  │                                                                 ▼
  └──► Dense Search (Vector) ─► [doc_C: Rank 1, doc_A: Rank 2] ──► Reciprocal Rank Fusion (RRF)
                                                                    │
                                                                    ▼
                                                             Final: [doc_A, doc_C, doc_B]
```

### Cross-Encoder Reranking
Embedding models are **Bi-Encoders**—they embed the query and documents independently, losing fine-grained contextual interactions. 

A **Reranker (Cross-Encoder)** takes the query and a retrieved chunk together and runs them through a unified attention layer. This calculates a highly accurate similarity score based on the direct grammatical relationship between the query and context.

```
User Query ──┐
             ├──► Cross-Encoder Model ──► Semantic Relevance Score (0.0 to 1.0)
Doc Chunk  ──┘
```

Because Cross-Encoders are computationally expensive, we use a two-stage retrieval pipeline:
1. Retrieve the top 20 candidate chunks using fast Hybrid Search (Stage 1).
2. Rerank those 20 candidates down to the top 5 chunks using a Cross-Encoder (Stage 2).

---

## 5. The Generation Phase: Proven Prompt Grounding Patterns

Once we have retrieved the top relevant chunks, we must format them into a highly resilient prompt context that prevents the LLM from hallucinating or ignoring instructions.

### Production Grounding Prompt Template

```python
GROUNDING_PROMPT = """
You are a highly analytical technical support assistant. Your task is to resolve the user query using ONLY the provided verified context.

STRICT OPERATIONAL DIRECTIVES:
1. Ground your answer entirely in the facts provided under the "VERIFIED CONTEXT" section.
2. If the answer cannot be confidently derived from the provided context, state: "I don't have enough verified information to answer this question." Do not attempt to synthesize outside knowledge.
3. For every claim or step you describe, append an in-text citation linking back to its source document ID (e.g., [doc_2]).
4. Keep your formatting technical, clean, and concise.

VERIFIED CONTEXT:
$context_data

USER QUERY:
$user_query

Provide your grounded, cited response below:
"""
```

---

## 6. RAG Evaluation: Faithfulness, Relevance, and Recall

To verify the quality of your RAG pipeline, you must move beyond subjective manual testing. The **RAGAS** framework provides a quantitative methodology to evaluate RAG components:

```
            ┌──────────────────────────────────────────────┐
            │               RAGAS EVALUATION               │
            ├──────────────────────┬───────────────────────┤
            │ Ingestion/Retrieval  │ Generation Quality    │
            │ • Context Recall     │ • Faithfulness        │
            │                      │ • Answer Relevance    │
            └──────────────────────┴───────────────────────┘
```

* **Faithfulness (Hallucination Metric):** Evaluates if the generated answer is derived *only* from the retrieved context. An LLM parses the generated answer into individual statements, and then checks if each statement is explicitly backed by the retrieved context chunks.
* **Answer Relevance:** Evaluates if the generated answer directly addresses the user's question. This is calculated by prompting an LLM to generate hypothetical questions from the generated answer, and then computing the vector cosine similarity between those generated questions and the user's actual query.
* **Context Recall:** Evaluates if the retrieval engine fetched all the necessary facts required to answer the question. This compares the retrieved chunks against a human-verified ground-truth answer.

---

## 🔨 Hands-On Production Labs

In this module's labs, you will build and run a production-grade RAG pipeline:

1. **The PDF Semantic Database Loader:** Parse technical PDFs, split the text using a recursive character splitter with dynamic page markers, generate embeddings using the OpenAI API, and store them in a local ChromaDB collection.
2. **Developing the Fully Grounded PDF Chatbot:** Build a stateful, interactive terminal application that retrieves relevant PDF chunks, formats them into a strict grounding prompt, and stream-renders responses with verified document citations.
3. **Implementing the Reranking Engine:** Integrate a local reranker (such as the Hugging Face `cohere-rerank` or `bge-reranker` models) into the retrieval loop to benchmark retrieval precision before and after reranking.

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Consolidate your understanding of document loading, splitting, vector indices, distance metrics, and RAG evaluation with 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md) | 🏆 Mini Project 01: PDF Chatbot
* **Objective:** Complete your first major graded project: the **Enterprise PDF Chatbot**. You must write a production-grade Python app that takes any uploaded technical PDF, processes it through a recursive character chunker, indexes it into a persistent ChromaDB database, performs hybrid retrieval with custom metadata filtering, and streams a grounded, cited response back to the user interface.
