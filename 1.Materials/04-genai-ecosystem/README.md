# Module 04 — GenAI Ecosystem & Tools

**Duration:** 2 Hours | **Session:** Weekend 2, Sunday (May 25, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Navigate the Provider Landscape:** Understand the comparative advantages, latency/cost profiles, and architecture trade-offs of OpenAI, Anthropic, Google, and Meta models.
* **Master Local Inference Architectures:** Install, run, and benchmark local open-weight models using Ollama and understand hardware quantization mechanics.
* **Deconstruct Frameworks vs. Raw SDKs:** Make pragmatic architectural choices between calling raw APIs, using lightweight templates, or building on heavy orchestrators like LangChain.
* **Explore Hugging Face:** Leverage the Hugging Face Hub to locate open weights and load models directly using the Python `transformers` library.

---

## 🗺️ Topics Covered

1. [LLM Provider Landscape: Enterprise API & Architecture Comparison](#1-llm-provider-landscape-enterprise-api--architecture-comparison)
2. [Local LLM Infrastructure: Ollama, Quantization & Offline Execution](#2-local-llm-infrastructure-ollama-quantization--offline-execution)
3. [AI Orchestrators: Raw SDKs vs. LangChain (LCEL) & LlamaIndex](#3-ai-orchestrators-raw-sdks-vs-langchain-lcel--llamaindex)
4. [Hugging Face Hub and the Transformers Library](#4-hugging-face-hub-and-the-transformers-library)

---

## 1. LLM Provider Landscape: Enterprise API & Architecture Comparison

Choosing the right foundation model involves evaluating several key factors: latency, reliability, licensing, context requirements, and API costs.

```
                  ┌──────────────────────────────────────────────┐
                  │          ENTERPRISE LLM SELECTION            │
                  ├──────────────────────┬───────────────────────┤
                  │ CLOUD PROVIDERS      │ LOCAL OPEN-WEIGHTS    │
                  │ (OpenAI, Anthropic,  │ (Llama 3, Qwen,       │
                  │  Gemini)             │  Mistral)             │
                  └──────────┬───────────┴───────────┬───────────┘
                             │                       │
                             ▼                       ▼
                     Pros: State-of-art      Pros: Zero cost, privacy,
                           No local hardware       Custom fine-tuning,
                           Simple APIs             100% offline control
```

### Comprehensive Provider Comparison

| Provider / Model Line | Best Performance Dimension | In-Context Reasoning | Function Calling Reliability | API Latency Profile | Best Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **OpenAI** (GPT-4o / Mini) | Tool Integration | Very High | Excellent | Low / Consistent | General production backends, complex JSON pipelines |
| **Anthropic** (Claude 3.5 Sonnet) | Software Engineering, Logic | Outstanding | Excellent | Moderate | Code parsing, deep logical planning, agent orchestration |
| **Google** (Gemini 1.5 Pro) | Massive Context Window (2M) | High | Very Good | Variable | Analyzing massive documents, multi-modal ingestion (video/audio) |
| **Meta / Open Weights** (Llama 3 / 3.1) | Cost & Privacy Control | High (70B model) | Good | Dependent on local hardware | Offline execution, high-compliance health/fintech data processing |

---

## 2. Local LLM Infrastructure: Ollama, Quantization & Offline Execution

Running open-weight models locally eliminates API costs, guarantees data privacy, and enables fully offline development. The primary tool of choice for local runtime execution is **Ollama**.

### The Mathematics of Quantization
To fit a high-parameter model (like Llama 3 with 8 Billion parameters) onto consumer hardware (such as an Apple M-series chip or a consumer Nvidia GPU), we must compress the model's weights. This process is called **Quantization**.

* **FP16 (Uncompressed):** Each parameter is stored as a 16-bit float.
  
  $$\text{VRAM Required} = 8,000,000,000 \times 2 \text{ bytes} \approx 16 \text{ GB}$$
  
* **INT4 / Q4_K_M (Compressed):** Compresses each float to 4 bits of precision.
  
  $$\text{VRAM Required} = 8,000,000,000 \times 0.5 \text{ bytes} \approx 4.7 \text{ GB}$$

While INT4 compression slightly degrades the model's language nuance, it reduces VRAM requirements by over $70\%$. This makes it possible to run highly capable models on standard laptops at excellent tokens-per-second rates.

### Initializing Ollama on Your System

```bash
# 1. Download and run the background runtime daemon from ollama.com (or use homebrew)
brew install ollama

# 2. Pull Llama 3 in background
ollama pull llama3:8b

# 3. List installed local models
ollama list

# 4. Spin up an interactive command-line terminal session with the model
ollama run llama3:8b
```

### OpenAI Compatibility Layer
Ollama runs an internal web server on port `11434`. It exposes an API that matches the OpenAI API spec, making it incredibly easy to switch your code from cloud APIs to local models.

```python
from openai import OpenAI

# 1. Point the client to the local Ollama runtime port
local_client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama" # Must be populated, but the value is ignored
)

# 2. Complete prompt exactly as you would with OpenAI's API
response = local_client.chat.completions.create(
    model="llama3:8b",
    messages=[
        {"role": "system", "content": "You are a local developer companion."},
        {"role": "user", "content": "Briefly explain quantization."}
    ],
    temperature=0.4
)

print(response.choices[0].message.content)
```

---

## 3. AI Orchestrators: Raw SDKs vs. LangChain (LCEL) & LlamaIndex

When structuring complex agent architectures, developers face a critical architectural decision: build directly on raw vendor SDKs or adopt an orchestration framework like LangChain.

```
RAW SDK APPROACH (OpenAI/Anthropic SDKs):
[App Logic] ───────► Direct HTTP/JSON Payloads ───────► Model APIs
  * Pros: Total control, zero hidden abstractions, fast boot, simple debugging.
  * Cons: Must write custom state, parser, and session management logic.

FRAMEWORK APPROACH (LangChain / LCEL):
[App Logic] ──► ChatPromptTemplate ──► ChatOpenAI ──► OutputParser ──► Target Output
  * Pros: Built-in memory, standard integrations, rapid prototyping.
  * Cons: High complexity, difficult to debug, brittle APIs, extensive custom abstractions.
```

### LangChain Expression Language (LCEL)
LangChain uses **LCEL** to declare processing pipelines using the Unix pipe operator (`|`). Under the hood, this leverages Python's `__or__` operator overrides to construct a runnable sequence graph.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from dotenv import load_dotenv

load_dotenv()

# 1. Set up the structural prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an automated refactoring bot. Provide only clean code, no conversations."),
    ("user", "Refactor this function to be more pythonic:\n```python\n{raw_code}\n```")
])

# 2. Initialize model instance
model = ChatOpenAI(model="gpt-4o-mini", temperature=0.1)

# 3. Initialize output parser (extracts raw text from response object)
parser = StrOutputParser()

# 4. Chain the components using the pipe operator
refactoring_chain = prompt | model | parser

# 5. Execute the chain synchronously
python_code = """
def process_numbers(nums):
    result = []
    for x in nums:
        if x % 2 == 0:
            result.append(x * 2)
    return result
"""

completed_output = refactoring_chain.invoke({"raw_code": python_code})
print(completed_output)
```

---

## 4. Hugging Face Hub and the Transformers Library

**Hugging Face** is the central repository for open-source AI. It hosts models, datasets, and interactive spaces across a wide range of modalities (NLP, Vision, Audio).

### Programmatic Ingestion with the `transformers` Pipeline
For specialized local tasks (like sentiment analysis, text classification, or feature extraction), you can run models directly in Python using the `transformers` library without needing a separate runtime server.

```python
import sys
from transformers import pipeline

print("[*] Downloading/loading sentiment classification model...")
# 1. Instantiate the high-level pipeline. This downloads the default model automatically on first run.
sentiment_analyzer = pipeline(
    task="sentiment-analysis", 
    model="distilbert-base-uncased-finetuned-sst-2-english"
)

# 2. Analyze sentiment directly on local hardware
review = "This tutorial provides incredibly crisp and actionable insights!"
result = sentiment_analyzer(review)

print(f"\nReview: '{review}'")
print(f"Outcome: {result[0]['label']} (Confidence Score: {result[0]['score']:.4f})")
```

---

## 🧪 Interactive Ecosystem Labs

In this module's labs, you will build and test three core integrations:

1. **The Model Performance Benchmark Suite:** Write a Python script that fires identical reasoning queries to both cloud APIs (`gpt-4o-mini`, `claude-3-5-sonnet`) and a local model (`llama3:8b`) to record, plot, and analyze latency, tokens-per-second, and total cost.
2. **LCEL Logging & Tracing Pipeline:** Build a classic LangChain LCEL chain with custom callbacks to intercept and print intermediate payloads, gaining visibility into exactly what is being sent to the model under the hood.
3. **Local Text Embedding Ingestion:** Use Ollama's embedded embedding API (`nomic-embed-text`) programmatically to convert a raw list of technical strings into local semantic vectors, saving them to a JSON database.

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Consolidate your understanding of providers, local quantization mathematics, LCEL pipelines, and Hugging Face pipelines with 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md)
* **Objective:** Implement a fallback orchestration system. Using pure Python or LangChain, build a query runner that first attempts to call a local Ollama model (`llama3:8b`). If local execution fails due to a timeout or missing daemon, the runner must gracefully fail over to the cloud API (`gpt-4o-mini`), logging latency and cost changes.
