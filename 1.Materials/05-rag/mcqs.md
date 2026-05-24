# Module 05 — MCQ Quiz

---

**Q1.** RAG stands for:

- A) Random Access Generation
- B) Retrieval Augmented Generation
- C) Real-time AI Generation
- D) Recursive Agent Generation

---

**Q2.** The primary problem RAG solves is:

- A) Making LLMs faster
- B) Reducing API costs
- C) Allowing LLMs to answer questions about data outside their training set
- D) Making models safer

---

**Q3.** What is "chunking" in the context of RAG?

- A) Compressing embeddings to save storage
- B) Splitting documents into smaller pieces before embedding
- C) Breaking the LLM's context into parallel processes
- D) A technique to reduce hallucination

---

**Q4.** Why is chunk overlap important when splitting documents?

- A) It reduces the total number of chunks
- B) It prevents losing context at chunk boundaries
- C) It improves the speed of embedding generation
- D) It reduces vector database storage costs

---

**Q5.** Cosine similarity is used in vector search to:

- A) Measure the angle between two vectors (semantic similarity)
- B) Calculate the Euclidean distance between embeddings
- C) Find exact keyword matches
- D) Compress embeddings for storage

---

**Q6.** ChromaDB is best described as:

- A) A relational database for storing documents
- B) A local, open-source vector database
- C) A cloud-only vector storage service
- D) An embedding model

---

**Q7.** In the RAG pipeline, embeddings are generated during:

- A) The retrieval step
- B) The generation step
- C) Both indexing (for documents) and retrieval (for queries)
- D) Only the indexing step

---

**Q8.** What does "k" represent in `collection.query(n_results=k)`?

- A) The number of documents to index
- B) The number of most similar chunks to retrieve
- C) The embedding dimension
- D) The maximum token count per chunk

---

**Q9.** When would RAG NOT be the right solution?

- A) When the document is longer than the context window
- B) When you need answers from a specific knowledge base
- C) When the answer is based on general world knowledge already in the LLM
- D) When you have multiple PDFs to search

---

**Q10.** The correct RAG system prompt tells the model to:

- A) Answer from its training data only
- B) Search the internet for answers
- C) Answer based on provided context only, and say "I don't know" if not found
- D) Generate the most creative answer

---

**Score:** ___/10

---

## Answer Key

| Question | Correct Answer |
| :---: | :---: |
| **Q1** | B |
| **Q2** | C |
| **Q3** | B |
| **Q4** | B |
| **Q5** | A |
| **Q6** | B |
| **Q7** | C |
| **Q8** | B |
| **Q9** | C |
| **Q10** | C |
