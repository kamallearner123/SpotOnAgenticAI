# Module 02 — Python for AI Builders

**Duration:** 2 Hours | **Session:** Weekend 1, Sunday (May 18, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Construct Production Environments:** Set up state-of-the-art Python virtual environments using high-performance package managers like `uv`.
* **Master the OpenAI Client Lifecycle:** Deeply understand SDK architecture, secure API key configurations, and proper HTTP connection pooling practices.
* **Guarantee Type-Safe JSON Outputs:** Move beyond legacy instructions to leverage modern Structured Outputs using Pydantic models to guarantee valid JSON parsability.
* **Harness Concurrency:** Write high-throughput, non-blocking asynchronous Python code using `asyncio` to execute multiple parallel LLM operations.
* **Build Real-World Tools:** Construct a robust, error-resistant PDF text summarizer and a stateful command-line chatbot from scratch.

---

## 🗺️ Topics Covered

1. [Modern Python Environment Architecture: venv, uv, and env](#1-modern-python-environment-architecture-venv-uv-and-env)
2. [The OpenAI Python SDK Lifecycle & Internals](#2-the-openai-python-sdk-lifecycle--internals)
3. [Guaranteed Structured Outputs using Pydantic](#3-guaranteed-structured-outputs-using-pydantic)
4. [Asynchronous AI & Concurrency with asyncio](#4-asynchronous-ai--concurrency-with-asyncio)
5. [Production-Grade Hands-on Implementations](#5-production-grade-hands-on-implementations)

---

## 1. Modern Python Environment Architecture: venv, uv, and env

Setting up a clean, reproducible development environment is essential when integrating multi-dependency AI projects. Historically, packages were managed using standard `venv` and slow `pip` installations. Modern AI system engineering has shifted towards high-speed rust-based tools.

### Python Environment Managers Comparison

| Tool | Language | Install Speed | Dependency Resolution | Key Benefit |
| :--- | :--- | :--- | :--- | :--- |
| **venv + pip** | Python | Slow | Basic | Built directly into Python |
| **Conda** | C Python | Slow | Heavy / Strict | Excellent for raw C/CUDA dependencies |
| **uv** (by Astral) | Rust | **Ultra-Fast ($10-100\times$)** | Advanced & Cached | Single-binary, lightning-fast virtual environments |

### Project Setup Guide using `uv`

To initialize your project environment and secure your API credentials:

```bash
# 1. Install uv globally
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. Create a clean virtual environment using the local python version
uv venv --python 3.11

# 3. Activate the virtual environment
# Mac/Linux:
source .venv/bin/activate
# Windows:
.venv\Scripts\activate

# 4. Install dependencies fast with caching
uv pip install openai pydantic python-dotenv pypdf asyncio

# 5. Save the locked dependency snapshot
uv pip freeze > requirements.txt
```

### Safe Environment Variable Ingestion
Never hardcode API secrets in your repositories. Use a `.env` file to manage environment configurations:

```ini
# File: .env
OPENAI_API_KEY=sk-proj-abc123XYZ...your-private-key...
OPENAI_API_BASE=https://api.openai.com/v1
```

Load these configurations into Python using `python-dotenv`:

```python
import os
from dotenv import load_dotenv

# Search for and load the .env file dynamically
load_dotenv()

# Ingest variable safely
api_key = os.getenv("OPENAI_API_KEY")
if not api_key:
    raise ValueError("CRITICAL: OPENAI_API_KEY environment variable is missing.")
```

---

## 2. The OpenAI Python SDK Lifecycle & Internals

When you initialize `client = OpenAI()`, the SDK establishes an instance of an HTTP client with specific defaults optimized for REST interaction.

```
+--------------------------------------------------------+
|                   OpenAI Client Instance               |
├────────────────────────────────────────────────────────┤
| • HTTP Connection Pool (urllib3)                      |
| • Automatic Retry Logic (Max Retries: 2, Exp Backoff)  |
| • Custom Request Headers & Authentication Details     |
+--------------------------------------------------------+
```

### Standard Synchronous Invocations
Here is the production anatomy of a standard chat completion call:

```python
import os
from openai import OpenAI, APIConnectionError, RateLimitError, APIStatusError
from dotenv import load_dotenv

load_dotenv()

# Initialize the client. This handles thread-safe HTTP connection pooling under the hood.
# It automatically loads OPENAI_API_KEY from environment variables by default.
client = OpenAI()

try:
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "You are a pragmatic, direct engineering assistant."},
            {"role": "user", "content": "Briefly explain connection pooling."}
        ],
        temperature=0.3,
        max_tokens=150,
        timeout=10.0 # Prevent system hangs by specifying a strict timeout (in seconds)
    )
    
    # Extraction path
    reply = response.choices[0].message.content
    print(f"Response:\n{reply}")
    
    # Metadata extraction
    prompt_tokens = response.usage.prompt_tokens
    completion_tokens = response.usage.completion_tokens
    print(f"\n[Tokens Used] Prompt: {prompt_tokens} | Completion: {completion_tokens}")

except APIConnectionError as e:
    print(f"The server could not be reached: {e.__cause__}")
except RateLimitError as e:
    print(f"A 429 status code was received. Back off and retry: {e}")
except APIStatusError as e:
    print(f"Non-200 status code returned. Status: {e.status_code} | Details: {e.response.json()}")
```

---

## 3. Guaranteed Structured Outputs using Pydantic

Older prompting strategies relied heavily on raw string parsing and manual regex matching to extract structured data (like JSON or keys) from LLMs. This approach was highly unstable.

### Structured Outputs API
Modern LLMs support native **Structured Outputs**. When you pass a **Pydantic schema** to the OpenAI client, the API parses the schema, builds a strict JSON schema template, and instructs the LLM to run its autocomplete token distribution through a constrained decoder. This guarantees that the output will match the exact shape of your data model.

```
                    ┌─────────────────────────┐
                    │  Pydantic Python Model   │
                    └────────────┬────────────┘
                                 │
                                 ▼ (Translates to JSON Schema)
┌─────────────────────────────────────────────────────────────┐
│                      OpenAI Constrained Decoder             │
├─────────────────────────────────────────────────────────────┤
│ • Validates output token syntax dynamically against schema. │
│ • Prevents syntax errors, invalid keys, or structural slips. │
└─────────────────────────────────────────────────────────────┘
```

### Production Pydantic Validation Implementation
Here is how to enforce a type-safe parser in production:

```python
from typing import List, Optional
from pydantic import BaseModel, Field
from openai import OpenAI

client = OpenAI()

# 1. Define the desired schema using Pydantic fields and validation descriptions
class SkillExtraction(BaseModel):
    name: str = Field(description="The formal name of the programming skill or tool.")
    years_of_experience: float = Field(description="Decimal value denoting years of usage.")
    seniority: str = Field(description="Estimated level: 'junior', 'mid', or 'senior'.")

class ResumeEvaluation(BaseModel):
    candidate_name: str = Field(description="Name of the applicant.")
    core_skills: List[SkillExtraction] = Field(description="List of extracted programming skills.")
    fit_score: int = Field(description="Confidence integer score matching job specs from 0 to 100.")
    gaps: Optional[List[str]] = Field(default=None, description="Identified requirement mismatches.")

# 2. Invoke chat completion using beta helpers for structured outputs
response = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a rigid resume parser. Extract candidate details accurately."},
        {"role": "user", "content": "Kamal Mukiri has used Python for 8 years, FastAPI for 3.5 years, and lacks Kubernetes experience."}
    ],
    response_format=ResumeEvaluation # Enforce strict parsing
)

# 3. Access the parsed object with native type checking
extracted_data: ResumeEvaluation = response.choices[0].message.parsed

print(f"Candidate: {extracted_data.candidate_name}")
print(f"Fit Score: {extracted_data.fit_score}/100")
for skill in extracted_data.core_skills:
    print(f" - {skill.name}: {skill.years_of_experience} yrs ({skill.seniority})")
print(f"Identified Gaps: {extracted_data.gaps}")
```

---

## 4. Asynchronous AI & Concurrency with asyncio

When building AI backends, making external network requests to model providers represents a significant block on performance. Running three requests sequentially forces your app to wait for the network I/O of each call.

By leveraging **Asynchronous Python (`asyncio`)**, the thread yields control of the process during the network wait time, allowing multiple API queries to run concurrently.

```
SEQUENTIAL PATTERN:
Query 1 [====== Network Latency Wait ======] -> Response 1
                                             Query 2 [====== Network Latency Wait ======] -> Response 2

ASYNC CONCURRENT PATTERN:
Query 1 [====== Network Latency Wait ======] (Yields CPU control)
Query 2 [====== Network Latency Wait ======] (Runs in parallel background)
        [=== Combined concurrency wait ===] -> Responses 1 & 2 returned together
```

### Async OpenAI Execution Pattern

```python
import asyncio
import time
from openai import AsyncOpenAI

# Use the Async client instead of the standard client
async_client = AsyncOpenAI()

async def fetch_semantic_summary(topic: str) -> str:
    print(f"[*] Starting request for: {topic}...")
    start_time = time.time()
    
    # Non-blocking async network invocation
    response = await async_client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Provide a precise 1-sentence engineering definition."},
            {"role": "user", "content": f"Define {topic}"}
        ],
        max_tokens=80
    )
    
    elapsed = time.time() - start_time
    print(f"[✓] Completed {topic} in {elapsed:.2f}s")
    return response.choices[0].message.content

async def main():
    topics = ["Vector Databases", "Asynchronous Event Loops", "Semantic Reranking"]
    
    start_all = time.time()
    
    # Run all tasks concurrently
    tasks = [fetch_semantic_summary(t) for t in topics]
    results = await asyncio.gather(*tasks)
    
    total_elapsed = time.time() - start_all
    print(f"\nAll tasks resolved concurrently in {total_elapsed:.2f} seconds!")
    
    for topic, result in zip(topics, results):
        print(f"\n* {topic}:\n  {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 5. Production-Grade Hands-on Implementations

### Implementation A: Robust PDF Text Summarizer

This utility reads a PDF, handles potential file extraction exceptions, splits the text into clean semantic prompts, and summarizes the content concurrently.

```python
import os
import sys
from pypdf import PdfReader
from openai import OpenAI, APIError
from dotenv import load_dotenv

load_dotenv()

class PDFSummarizer:
    def __init__(self, model_name: str = "gpt-4o-mini"):
        self.client = OpenAI()
        self.model = model_name

    def extract_text_from_pdf(self, file_path: str) -> str:
        """Reads PDF pages and merges text, catching file system exceptions."""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Source PDF file not found at: {file_path}")
            
        try:
            reader = PdfReader(file_path)
            full_text = []
            for i, page in enumerate(reader.pages):
                page_text = page.extract_text()
                if page_text:
                    full_text.append(page_text)
            
            extracted = "\n".join(full_text).strip()
            if not extracted:
                raise ValueError("The PDF contains no legible text characters (it might be scanned).")
            return extracted
            
        except Exception as e:
            print(f"[CRITICAL] Failed to parse PDF file: {e}")
            raise

    def generate_summary(self, pdf_text: str, max_chunk_chars: int = 4000) -> str:
        """Chunks PDF contents and requests summary synthesis."""
        # Simple character-based slicing for safety
        sliced_text = pdf_text[:max_chunk_chars]
        
        print(f"[*] Dispatching {len(sliced_text)} characters of context to {self.model}...")
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": "You are a technical document annotator. Synthesize key engineering requirements in bullet points."},
                    {"role": "user", "content": f"Document Snippet:\n\"\"\"{sliced_text}\"\"\""}
                ],
                temperature=0.2
            )
            return response.choices[0].message.content
        except APIError as e:
            print(f"[API ERROR] OpenAI interaction failed: {e}")
            raise

# Execution entry point
if __name__ == "__main__":
    summarizer = PDFSummarizer()
    try:
        # Example using a sample pdf file
        raw_text = summarizer.extract_text_from_pdf("sample.pdf")
        summary_result = summarizer.generate_summary(raw_text)
        print(f"\n--- Executive PDF Summary ---\n{summary_result}")
    except Exception as error:
        print(f"[Execution Halted] {error}")
```

### Implementation B: Stateful Terminal Chatbot with System Prompts

This chatbot maintains a system prompt and dynamically updates its conversation history, preventing history explosion by keeping token footprints minimal.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

class TerminalChatEngine:
    def __init__(self, system_persona: str, model: str = "gpt-4o-mini"):
        self.client = OpenAI()
        self.model = model
        # Initialize memory list with system context
        self.conversation_history = [
            {"role": "system", "content": system_persona}
        ]

    def chat_loop(self):
        print("==================================================")
        print(f"AI Assistant Active (Model: {self.model})")
        print("Type 'exit', 'quit', or 'clear' to manage state.")
        print("==================================================\n")

        while True:
            try:
                user_msg = input("\033[94mYou:\033[0m ").strip()
                
                # Check for control commands
                if not user_msg:
                    continue
                if user_msg.lower() in ["exit", "quit"]:
                    print("[*] Terminating conversation. Memory cleared.")
                    break
                if user_msg.lower() == "clear":
                    # Keep system persona but wipe transient chat history
                    self.conversation_history = [self.conversation_history[0]]
                    print("[*] History wiped successfully.\n")
                    continue

                # Append user prompt to state history
                self.conversation_history.append({"role": "user", "content": user_msg})

                # Stream response from model
                print("\033[92mAI:\033[0m ", end="", flush=True)
                
                response_stream = self.client.chat.completions.create(
                    model=self.model,
                    messages=self.conversation_history,
                    stream=True # Stream tokens as they are generated for a premium experience
                )

                collected_chunks = []
                for chunk in response_stream:
                    token = chunk.choices[0].delta.content
                    if token:
                        print(token, end="", flush=True)
                        collected_chunks.append(token)
                print("\n")

                # Append full assistant response back to conversation memory
                full_reply = "".join(collected_chunks)
                self.conversation_history.append({"role": "assistant", "content": full_reply})

            except KeyboardInterrupt:
                print("\n[*] Exiting chat thread.")
                break
            except Exception as e:
                print(f"\n[Error Encountered] {e}\n")

# Execution Entry Point
if __name__ == "__main__":
    persona = "You are a concise Unix system administrator. Respond with raw commands or highly technical explanations."
    engine = TerminalChatEngine(system_persona=persona)
    engine.chat_loop()
```

---

## 📝 MCQ Checklist → [mcqs.md](./mcqs.md)
* Consolidate syntax understanding of environment setup and API payloads by answering the 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md)
* **Objective:** Modify the asynchronous loop implementation to run comparative latency benchmarks between models (`gpt-4o-mini` vs. `gpt-4o`) and calculate the exact API call cost dynamically based on returned token values.
