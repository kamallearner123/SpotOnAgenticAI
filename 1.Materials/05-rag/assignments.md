# Module 05 — Assignment

## 🏆 Mini Project 01: PDF Chatbot

**Due:** June 6, 2026  
**This is a graded mini-project (not just an assignment)**  
**Submission:** GitHub repo with working code + demo video (2 min screen recording)

---

## Project Goal

Build a production-ready PDF chatbot that can answer questions about any PDF document.

---

## Requirements

### Core Features (60 points)
- [ ] Accept any PDF file as input
- [ ] Chunk and embed the document into ChromaDB
- [ ] Answer questions based on document content only
- [ ] Show which chunk the answer came from (source attribution)
- [ ] Handle "I don't know" gracefully when answer isn't in the document

### Enhanced Features (30 points — choose any 3)
- [ ] Support multiple PDF files at once
- [ ] Implement persistent storage (ChromaDB persists across runs)
- [ ] Show confidence/relevance score with each answer
- [ ] Support conversation history (remember previous questions)
- [ ] Add a simple web interface using Streamlit or Gradio

### Code Quality (10 points)
- [ ] Clean, readable code with comments
- [ ] Requirements.txt included
- [ ] README with setup and usage instructions

---

## Architecture

Your solution should follow this structure:

```
pdf_chatbot/
├── README.md
├── requirements.txt
├── .env.example
├── chatbot.py          ← main entry point
├── ingestion.py        ← PDF loading and chunking
├── retriever.py        ← vector DB operations
├── generator.py        ← LLM call and response
└── sample_docs/
    └── sample.pdf      ← include at least one sample PDF
```

---

## Evaluation Criteria

```
Core functionality works          40%
Code is well-organized            20%
Source attribution implemented    15%
"I don't know" handled correctly  15%
README is clear                   10%
```

---

## Demo Script

Your demo video should show:
1. Loading a PDF (show the chunking process)
2. Asking 3 questions that ARE in the document (with correct answers)
3. Asking 1 question that is NOT in the document (shows "I don't know")
4. The source chunk displayed alongside the answer

---

## Starter Code

```python
# chatbot.py — starter
import os
import chromadb
from pypdf import PdfReader
from openai import OpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter

client = OpenAI()
chroma = chromadb.PersistentClient(path="./chroma_db")  # persists across runs

def load_and_index_pdf(pdf_path: str, collection_name: str = "pdf_docs"):
    # 1. Check if already indexed
    try:
        collection = chroma.get_collection(collection_name)
        print(f"Using existing index: {collection.count()} chunks")
        return collection
    except:
        pass

    # 2. Load PDF
    reader = PdfReader(pdf_path)
    full_text = "\n".join(page.extract_text() for page in reader.pages)
    print(f"Loaded PDF: {len(reader.pages)} pages")

    # 3. Chunk
    splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
    chunks = splitter.split_text(full_text)
    print(f"Created {len(chunks)} chunks")

    # 4. Embed and store
    collection = chroma.create_collection(collection_name)
    collection.add(
        documents=chunks,
        ids=[f"chunk_{i}" for i in range(len(chunks))],
        metadatas=[{"page_range": f"chunk_{i}", "source": pdf_path} for i in range(len(chunks))]
    )
    print("Indexed successfully!")
    return collection

def answer_question(collection, question: str) -> dict:
    # Retrieve
    results = collection.query(query_texts=[question], n_results=3)
    chunks = results["documents"][0]
    sources = results["metadatas"][0]

    # Generate
    context = "\n\n---\n\n".join(chunks)
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Answer using ONLY the provided context. If the answer isn't there, say 'I don't know based on the provided document.'"},
            {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {question}"}
        ]
    )

    return {
        "answer": response.choices[0].message.content,
        "sources": chunks[:1],  # show top source chunk
        "source_meta": sources[:1]
    }

if __name__ == "__main__":
    import sys
    pdf_path = sys.argv[1] if len(sys.argv) > 1 else "sample_docs/sample.pdf"
    collection = load_and_index_pdf(pdf_path)

    print(f"\nPDF Chatbot ready! Asking about: {pdf_path}")
    print("Type 'quit' to exit\n")

    while True:
        q = input("You: ").strip()
        if q.lower() == "quit":
            break
        result = answer_question(collection, q)
        print(f"\nAI: {result['answer']}")
        print(f"\n[Source: {result['sources'][0][:200]}...]\n")
```
