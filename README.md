# 🤖 Spot on Agentic AI: Build Systems That Think, Remember & Act

> *"AI is not magic. LLMs are programmable. Agents are just systems with memory + tools + reasoning. You can build this yourself."*

Welcome to the **Agentic AI** developer masterclass. This course is an immersive, hands-on, and production-first curriculum designed specifically for software developers, system engineers, and backend programmers. 

Rather than treating AI as a "black box" or a collection of high-level abstractions, this course walks you through the core principles of building robust, autonomous, and self-correcting AI systems from first principles.

---

## 📋 Course Overview

| Parameter | Masterclass Details |
| :--- | :--- |
| **Course Title** | Agentic AI: Build Systems That Think, Remember & Act |
| **Trainer** | Kamal Kumar Mukiri |
| **Duration** | 24 Hours of Intensive, Hands-on Sessions |
| **Schedule** | Saturday & Sunday, 3:00 PM – 6:00 PM IST |
| **Mode** | Fully Interactive, Project-driven, Developer-first Learning |
| **Target Audience** | Software Developers, Backend/System Engineers, Technical Architects |

---

## 🎯 Production Systems You Will Build

By the end of this course, you won't just write prompts; you will build, evaluate, and deploy five production-grade AI systems:

```
┌─────────────────────────────────────────────────────────────────┐
│                     AGENTIC AI ECOSYSTEM                        │
├─────────────────┬─────────────────┬──────────────┬──────────────┤
│ 🗂️ PDF Chatbot  │ 🔍 Search Agent │ 💻 Code Gen  │ 📊 Researcher│
│  (Advanced RAG) │  (ReAct Loop)   │ (Self-Debug) │ (Multi-Agent)│
└─────────────────┴─────────────────┴──────────────┴──────────────┘
```

1. **🗂️ Enterprise PDF Chatbot (Advanced RAG)**
   * *Architecture:* Chunking pipeline, dense semantic embeddings, hybrid vector-sparse search, reranking (using Cross-Encoders), and citation/source grounding.
2. **🔍 Autonomous Web Search Agent**
   * *Architecture:* Continuous reasoning loop (using the ReAct pattern), dynamic tool call routing, page scraping, context assembly, and automatic token truncation.
3. **💻 Self-Correcting Code Assistant**
   * *Architecture:* Local code execution sandbox, runtime output redirection, traceback extraction, compiler-guided prompt correction loops, and self-healing code generation.
4. **📊 Multi-Agent Collaborative Research Team**
   * *Architecture:* Hierarchical multi-agent crew (Planner, Researcher, Editor), task assignment, state consolidation, and collaborative output generation.
5. **🛡️ Production Personal AI Assistant**
   * *Architecture:* State persistence, short-term conversational buffers, long-term episodic retrieval, security guardrails, rate limiting, and observability.

---

## 🧠 Prerequisites & Ideal Student Profile

This course bypasses heavy mathematical formulas, gradient descent matrices, and statistics theories. We focus on **AI System Engineering**.

* **You should know:**
  * Solid fundamentals in **Python** (loops, lists, dicts, basic asynchronous syntax).
  * Fundamentals of **REST APIs** (HTTP methods, headers, payload formats, JSON manipulation).
  * Comfortable navigating the **Command Line** (environment variables, system paths, pip).
* **You do NOT need:**
  * Prior Machine Learning or deep learning background.
  * Complex math, calculus, or statistics.
  * Previous experience with generative models or prompt tuning.

---

## 🗺️ Curriculum Learning Journey

Below is the conceptual journey you will follow, moving systematically from raw language models to stateful, multi-agent frameworks:

```mermaid
graph TD
    A[Module 1: AI Foundations] --> B[Module 2: Python for AI Builders]
    B --> C[Module 3: Prompt Engineering Deep Dive]
    C --> D[Module 4: GenAI Ecosystem & Tools]
    D --> E[Module 5: Advanced RAG]
    E --> F[Module 6: Agentic AI Fundamentals]
    F --> G[Module 7: Building Agents (LangGraph/CrewAI)]
    G --> H[Module 8: MCP Server Architectures]
    H --> I[Module 9: Production AI System Design]
    I --> J[Module 10: Capstone Showcases]
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style E fill:#bbf,stroke:#333,stroke-width:2px
    style G fill:#bfb,stroke:#333,stroke-width:2px
    style I fill:#fbf,stroke:#333,stroke-width:2px
```

---

## 📚 Detailed Course Modules

### [Module 01 — AI Foundations for Developers](./1.Materials/01-ai-foundations/README.md)
* **Focus:** Under-the-hood mechanics of large language models.
* **Topics:** Classic programming vs. ML vs. DL vs. GenAI; Byte-Pair Encoding (BPE) tokenizers; semantic vector embeddings and high-dimensional spaces; context windows and attention limits; self-attention mechanics in Transformers; Supervised Fine-Tuning (SFT) and Reinforcement Learning from Human Feedback (RLHF).
* **Key Outcome:** Understand LLMs, tokens, and semantic similarity inside and out.

### [Module 02 — Python for AI Builders](./1.Materials/02-python-for-ai/README.md)
* **Focus:** Programmatic control of LLM calls using modern Python.
* **Topics:** Environment setup with `venv` and `uv`; secure environment management via dotenv; OpenAI SDK client structures and response payloads; structured JSON parsing and robust error handling; concurrent execution with Python `asyncio`.
* **Key Outcome:** Build a fully asynchronous terminal chatbot and document summarizer with robust error recovery.

### [Module 03 — Prompt Engineering Deep Dive](./1.Materials/03-prompt-engineering/README.md)
* **Focus:** Programmatic output control and security.
* **Topics:** Zero-shot and Few-shot paradigms; Chain-of-Thought (CoT) and Self-Consistency; Role Prompting activation states; Structured Prompting with Pydantic; Prompt Injection attacks (jailbreaks, prompt leaking) and advanced defense engineering.
* **Key Outcome:** Build structured, secure prompts that reliably feed downstream software systems.

### [Module 04 — GenAI Ecosystem & Tools](./1.Materials/04-genai-ecosystem/README.md)
* **Focus:** Navigating commercial and open-weight ecosystems.
* **Topics:** Cloud providers (OpenAI, Anthropic, Google Gemini, Meta Llama) vs. local model execution; local inference setups with Ollama and quantization profiles; raw SDKs vs. orchestrators (LangChain, LlamaIndex); Hugging Face Hub APIs.
* **Key Outcome:** Deploy and call both cloud APIs and high-performance local open-weight models.

### [Module 05 — RAG — Retrieval Augmented Generation](./1.Materials/05-rag/README.md)
* **Focus:** Building search systems over proprietary documentation.
* **Topics:** Ingestion pipelines; document loaders and recursive/token/semantic chunking; vector databases (Pinecone, Chroma, FAISS, `pgvector`); dense semantic retrieval vs. sparse BM25 keyword retrieval; Reranking (Cross-Encoder models); RAG evaluation metrics (Faithfulness, Answer Relevance).
* **Key Outcome:** Build a production-grade PDF chatbot with verified citations and zero hallucinations.

### [Module 06 — Agentic AI Fundamentals](./1.Materials/06-agentic-ai-fundamentals/README.md)
* **Focus:** Transitioning from static RAG pipelines to autonomous execution loops.
* **Topics:** Deterministic code paths vs. autonomous agent loops; the ReAct (Reasoning + Acting) pattern; short-term conversational buffers and long-term vector-based episodic memory; function calling mechanics and tool schemas; planning, decomposition, and self-reflection loops.
* **Key Outcome:** Implement a custom ReAct agent loop from scratch in pure Python.

### [Module 07 — Building AI Agents](./1.Materials/07-building-agents/README.md)
* **Focus:** Production agent orchestration frameworks.
* **Topics:** CrewAI declarative multi-agent crews (roles, backstories, collaboration, delegation); LangGraph stateful multi-agent execution graphs (nodes, edges, conditional routing, state compilation); cyclic execution loops; human-in-the-loop state intervention.
* **Key Outcome:** Build cooperative multi-agent teams and resilient, complex self-correction loops.

### [Module 08 — MCP & AI Tool Integration](./1.Materials/08-mcp-tool-integration/README.md)
* **Focus:** The Model Context Protocol (MCP) by Anthropic.
* **Topics:** Standardizing client-server tool interactions; Host/Client/Server MCP architecture; standardizing Resources, Prompts, and Tools; building custom Python MCP servers for database, browser, and filesystem access.
* **Key Outcome:** Build a custom MCP server to securely connect local tools and files to any LLM.

### [Module 09 — AI System Design & Production](./1.Materials/09-system-design-production/README.md)
* **Focus:** Scaling AI systems, cost reduction, resilience, and security.
* **Topics:** Caching strategies (Redis exact match vs. GPTCache semantic caching); resilience patterns (Exponential backoff, jitter, rate-limit back-offs, model fallbacks); cost and token optimization strategies; input/output guardrails and PII redaction; LLM observability, latency tracking, and multi-step tracing (LangSmith, Phoenix).
* **Key Outcome:** Build resilient, cost-optimized, and observable enterprise AI pipelines.

### [Module 10 — Capstone Projects](./1.Materials/10-capstone-projects/README.md)
* **Focus:** Final systems showcase.
* **Topics:** Real-world problem statement matching; project architecture patterns; robust test execution frameworks; visual user interface construction; final cohort presentations and peer evaluations.
* **Key Outcome:** Complete and showcase an end-to-end, production-grade Capstone Project.

---

## 🗓️ Weekend Schedule

See the full [Detailed Schedule →](./SCHEDULE.md) for daily breakdowns.

| Week | Dates | Modules Covered | Milestones & Graded Assignments |
| :---: | :--- | :--- | :--- |
| **Week 1** | May 17–18 | Modules 01, 02, 03 (part 1) | Environment Setup check, Assignment 01: Token-Cost Counter |
| **Week 2** | May 24–25 | Modules 03 (part 2), 04, 05 | Assignment 02: Pydantic Structured Output, **Mini Project 1: PDF Chatbot** |
| **Week 3** | May 31 – Jun 1 | Modules 06, 07 (part 1) | Assignment 03: Pure Python ReAct Agent, Capstone Project Pitch |
| **Week 4** | Jun 7–8 | Modules 07 (part 2), 08, 09 | Assignment 04: Custom MCP Database Server, **Mini Project 2: Research Agent** |
| **Week 5** | Jun 14–15 | Module 10 | **Capstone Project Showcase & Graduation** |

---

## 📂 Repository Structure

```
SpotOnAgenticAI/
├── README.md                     ← You are here
├── SCHEDULE.md                   ← Master schedule & daily learning objectives
├── requirements.txt              ← Global project dependencies
├── 1.Materials/                  ← Course slide notes, assignments, and quizzes
│   ├── 01-ai-foundations/        ← Module 1: Tokenizers, embeddings, self-attention
│   ├── 02-python-for-ai/         ← Module 2: Client setup, parsing, async basics
│   ├── 03-prompt-engineering/    ← Module 3: CoT, structured outputs, prompt security
│   ├── 04-genai-ecosystem/       ← Module 4: Ollama, Hugging Face, provider landscape
│   ├── 05-rag/                   ← Module 5: Chunking, vectors, hybrid search, RAGAS
│   ├── 06-agentic-ai-fundamentals/← Module 6: Custom ReAct loop, tool binding, state
│   ├── 07-building-agents/       ← Module 7: Multi-agent crews (CrewAI), stateful graphs (LangGraph)
│   ├── 08-mcp-tool-integration/  ← Module 8: Model Context Protocol servers
│   ├── 09-system-design-production/← Module 9: Caching, fallback, guardrails, LangSmith
│   └── 10-capstone-projects/     ← Module 10: Capstone guidelines & boilerplates
├── 2.Projects/                   ← Workspace for assignments & graded mini-projects
├── 3.Resources/                  ← Extra setup guides, API cheatsheets, reference papers
└── 4.DailyPractices/             ← Daily lab scratchpads & coding warmups
```

---

## 🛠️ Environment Initialization

To get started, please follow the [Environment Setup Guide →](./3.Resources/setup-guide.md). 

Ensure you have your tools initialized:
```bash
# Verify Python version
python3 --version  # Output should be >= 3.10

# Initialize a clean virtual environment using the ultra-fast uv tool (or standard venv)
uv venv .venv
source .venv/bin/activate

# Install global requirements
uv pip install -r requirements.txt
```

---

## 👨‍🏫 About the Trainer

**Kamal Kumar Mukiri**  
*AI Systems Architect & Software Engineer*  

Kamal is a seasoned backend engineer and system architect specializing in large-scale event-driven systems and intelligent automation. He has built and scaled multi-agent AI platforms in enterprise fintech and healthcare spaces, focusing on practical, developer-first patterns to bridge the gap between AI prototype toys and reliable, production-grade distributed architectures.

---

## 📬 Contact & Inquiries

For syllabus clarifications, custom training inquiries, or general registration details, reach out to **Kamal Kumar Mukiri**.

---

> *This course is structured so that by the end, you don't just understand Agentic AI — you've built it.*
