# Module 04 — GenAI Ecosystem & Tools

**Duration:** 2 Hours | **Session:** Weekend 2, Sunday (May 25, 2026)

---

## 🎯 Learning Objectives

- Navigate the GenAI tool landscape without getting overwhelmed
- Run a local LLM using Ollama — no API keys, no cost
- Understand when to use which framework (LangChain vs CrewAI vs LangGraph)
- Know where to find open-source models on HuggingFace

---

## The Ecosystem Map

```
┌─────────────────────────────────────────────────────────────┐
│                        LLM PROVIDERS                         │
│  OpenAI (GPT-4)  │  Anthropic (Claude)  │  Google (Gemini)  │
│  Mistral  │  Cohere  │  Together AI  │  Groq                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      LOCAL / OPEN SOURCE                     │
│  Ollama (run locally)  │  LM Studio (GUI)  │  HuggingFace   │
│  Models: Llama3, DeepSeek, Mistral, Phi-3, Qwen             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     AI FRAMEWORKS                            │
│  LangChain  │  LlamaIndex  │  CrewAI  │  AutoGen  │ LangGraph│
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     VECTOR DATABASES                         │
│  ChromaDB (local)  │  FAISS  │  Pinecone  │  Weaviate       │
└─────────────────────────────────────────────────────────────┘
```

---

## Tool Comparison

### LLM Providers

| Provider | Best For | Pricing | Notes |
|----------|----------|---------|-------|
| **OpenAI** | Production, reliability | Pay-per-token | GPT-4o is state-of-art |
| **Anthropic** | Long documents, safety | Pay-per-token | Claude 3.5 for coding |
| **Groq** | Speed (fastest inference) | Free tier available | Great for prototyping |
| **Ollama** | Privacy, offline, no cost | Free (local) | Must have GPU or M-chip |
| **HuggingFace** | Research, custom models | Free/paid | Huge model library |

---

### Frameworks

| Framework | Best For | Complexity | When to Use |
|-----------|----------|------------|-------------|
| **LangChain** | Chains, RAG, agents | Medium | General purpose |
| **LlamaIndex** | Document Q&A, RAG | Medium | Heavy document work |
| **CrewAI** | Multi-agent teams | Low-Medium | Role-based agents |
| **AutoGen** | Conversational agents | Medium | Agent-to-agent chat |
| **LangGraph** | Complex agent workflows | High | State machines, loops |

---

## Ollama — Run LLMs Locally

### Install & Run
```bash
# Install Ollama (Mac)
brew install ollama

# Or download from ollama.com

# Pull and run Llama 3
ollama pull llama3

# Run interactively
ollama run llama3

# List downloaded models
ollama list
```

### Call Ollama from Python
```python
from openai import OpenAI  # Ollama has OpenAI-compatible API

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # required but ignored
)

response = client.chat.completions.create(
    model="llama3",
    messages=[{"role": "user", "content": "Explain RAG in 3 sentences."}]
)
print(response.choices[0].message.content)
```

### Popular Local Models

| Model | Size | Best For |
|-------|------|----------|
| `llama3` | 4.7GB | General purpose, strong reasoning |
| `deepseek-coder` | 6.7GB | Code generation |
| `mistral` | 4.1GB | Fast, good instruction following |
| `phi3:mini` | 2.3GB | Lightweight, runs on CPU |
| `llava` | 4.5GB | Multimodal (text + images) |

---

## LangChain Quick Start

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage

model = ChatOpenAI(model="gpt-4o-mini")

messages = [
    SystemMessage(content="You are a Python expert."),
    HumanMessage(content="What is a decorator?")
]

response = model.invoke(messages)
print(response.content)
```

### LangChain Chain (LCEL)
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}."),
    ("user", "{question}")
])

model = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

# Build a chain using the pipe operator
chain = prompt | model | parser

result = chain.invoke({
    "role": "Python expert",
    "question": "What is the GIL?"
})
print(result)
```

---

## HuggingFace — Finding Models

```python
from transformers import pipeline

# Sentiment analysis (no API key needed)
classifier = pipeline("sentiment-analysis")
result = classifier("I love building AI systems!")
print(result)  # [{'label': 'POSITIVE', 'score': 0.999}]

# Text generation
generator = pipeline("text-generation", model="gpt2")
result = generator("Python is great for AI because", max_length=50)
```

**HuggingFace Hub:** [huggingface.co/models](https://huggingface.co/models)
- Filter by task: text-generation, sentence-similarity, question-answering
- Filter by size: for local use, pick models under 10GB

---

## 🧪 Demo: Run DeepSeek Locally

```bash
# Pull DeepSeek Coder (good for coding tasks)
ollama pull deepseek-coder:6.7b

# Test it
ollama run deepseek-coder:6.7b "Write a Python function to reverse a linked list"
```

```python
# Use from Python
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

response = client.chat.completions.create(
    model="deepseek-coder:6.7b",
    messages=[
        {"role": "system", "content": "You are an expert programmer."},
        {"role": "user", "content": "Write a binary search in Python with comments."}
    ]
)
print(response.choices[0].message.content)
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md)
