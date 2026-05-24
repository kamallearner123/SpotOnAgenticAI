# Module 09 — AI System Design & Production

**Duration:** 2 Hours | **Session:** Weekend 4, Sunday (June 8, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Design Enterprise AI Topologies:** Architect highly resilient, scalable, and observable production-grade AI system backends.
* **Implement Advanced Caching:** Compare Exact Match Caching (Redis) with Semantic Caching (GPTCache) to lower latency and API costs.
* **Master Error Resilience:** Code robust retry architectures incorporating exponential backoff, structural jitter, rate-limit back-offs, and multi-provider model fallbacks.
* **Enforce Security & Input Guardrails:** Implement user validation layers, input sanitization, PII redaction, and output guardrails.
* **Establish Centralized Observability:** Instrument metric collectors tracking Latency, Cost, Time-to-First-Token (TTFT), and trace execution graphs.

---

## 🗺️ Topics Covered

1. [Enterprise Production AI System Topologies](#1-enterprise-production-ai-system-topologies)
2. [AI Caching Architectures: Exact vs. Semantic Caching](#2-ai-caching-architectures-exact-vs-semantic-caching)
3. [Production Error Resilience: Retries, Jitter, and Fallbacks](#3-production-error-resilience-retries-jitter-and-fallbacks)
4. [Enterprise Security: Input Sanitization, PII Redaction, and Guardrails](#4-enterprise-security-input-sanitization-pii-redaction-and-guardrails)
5. [Observability, Latency Metrics & Execution Tracing](#5-observability-latency-metrics--execution-tracing)

---

## 1. Enterprise Production AI System Topologies

Moving an AI prototype to a production environment requires wrapping the core model inside standard enterprise infrastructure. An AI application must handle authentications, manage costs, survive external API outages, enforce rate limits, and provide complete traceability.

```
                  ┌──────────────────────────────────────────────┐
                  │                 CLIENT LAYER                 │
                  │        (Web, Mobile, Slack, CLI API)         │
                  └──────────────────────┬───────────────────────┘
                                         │
                                         ▼
                  ┌──────────────────────────────────────────────┐
                  │              GATEWAY / INGRESS               │
                  │  (Rate Limiter, Auth, HTTPS, Load Balancer)  │
                  └──────────────────────┬───────────────────────┘
                                         │
                                         ▼
                  ┌──────────────────────────────────────────────┐
                  │             APPLICATION LOGIC                │
                  │   ┌─────────────┐ ┌─────────────┐ ┌───────┐  │
                  │   │ Guardrails  │ │Prompt Engine│ │Caching│  │
                  │   └──────┬──────┘ └──────┬──────┘ └───┬───┘  │
                  │          │               │            │      │
                  └──────────┼───────────────┼────────────┼──────┘
                             │               │            │
                             ▼               ▼            ▼
                  ┌──────────────────────────────────────────────┐
                  │              FOUNDATION LLMs                 │
                  │   (Primary Cloud, Fallback Cloud, Local)     │
                  └──────────────────────────────────────────────┘
```

---

## 2. AI Caching Architectures: Exact vs. Semantic Caching

Model API latency (often 1–5 seconds) represents a major bottleneck for user experiences. To lower latency and save API costs, production backends intercept incoming prompts using a caching layer.

```
Exact Match Caching (Redis):
Prompt: "Explain RAG" ──► MD5 Hash ──► Key: "llm:hash" ──► Found? YES ──► Return response

Semantic Caching (GPTCache):
Prompt: "Define RAG"  ──► Embed ────► Cosine Search ──► Similarity > 0.95? ──► Return response
                                                          (Finds "Explain RAG")
```

### Exact Match Caching (Redis)
* **How it works:** The prompt string is hashed (e.g., using MD5). The hash is used as a lookup key in an in-memory key-value database like Redis.
* **Pros:** Extremely fast ($< 2 \text{ ms}$ lookup), highly deterministic, and has zero overhead.
* **Cons:** Brittle. A tiny change in punctuation or a single whitespace change (e.g., `"Explain RAG."` vs `"Explain RAG"`) results in a cache miss.

### Semantic Caching (GPTCache)
* **How it works:** The incoming prompt is passed through an embedding model to generate its semantic vector. This vector is compared against a database of previously embedded prompts using cosine similarity. If the similarity score exceeds a strict threshold (e.g., $> 0.96$), the system returns the cached completion.
* **Pros:** Highly resilient. Matches queries with similar meanings even if the wording is different (e.g., `"What is RAG?"` matches `"Explain Retrieval Augmented Generation"`).
* **Cons:** Slower than exact matching (requires calling the embedding model for the lookup) and can return slightly off-target responses if the threshold is set too low.

---

## 3. Production Error Resilience: Retries, Jitter, and Fallbacks

API providers frequently experience transient errors, rate limit spikes (HTTP 429), or complete regional outages. Your backend must handle these gracefully to prevent user-facing crashes.

```python
# PRODUCTION RESILIENCE PIPELINE:
# 1. Catch Rate Limit Error (HTTP 429)
# 2. Back off exponentially: wait_time = 2 ^ attempt + random_jitter
# 3. If primary provider fails completely, route call to Backup Fallback Model
```

### Resilient Call Implementation with Jitter and Fallback

```python
import os
import time
import random
from openai import OpenAI, RateLimitError, APIError
from dotenv import load_dotenv

load_dotenv()
client = OpenAI()

def resilient_llm_call(
    messages: list,
    primary_model: str = "gpt-4o",
    fallback_model: str = "gpt-4o-mini",
    max_retries: int = 3,
    base_backoff_sec: float = 1.5
) -> str:
    """
    Executes an LLM call with exponential backoff, random jitter, 
    and failover to a backup fallback model.
    """
    for attempt in range(1, max_retries + 1):
        try:
            print(f"[*] Dispatching request to {primary_model} (Attempt {attempt}/{max_retries})...")
            response = client.chat.completions.create(
                model=primary_model,
                messages=messages,
                timeout=15.0 # Enforce strict network timeout
            )
            return response.choices[0].message.content

        except RateLimitError as e:
            # Exponential Backoff with Jitter
            # Formula: Backoff = (Base * 2 ^ attempt) + Random Uniform Jitter
            backoff_duration = (base_backoff_sec * (2 ** attempt)) + random.uniform(0.1, 0.5)
            print(f"[!] Rate Limited (429). Backing off for {backoff_duration:.2f} seconds: {e}")
            if attempt == max_retries:
                # If we've run out of retries on the primary model, fall back to the backup model
                print(f"[✕] Max retries reached on {primary_model}. Attempting failover to {fallback_model}...")
                break
            time.sleep(backoff_duration)

        except APIError as e:
            print(f"[!] Server API Error ({e.status_code}) encountered. Initiating fallback failover...")
            break

    # Failover fallback attempt
    try:
        print(f"[*] Dispatching failover request to {fallback_model}...")
        response = client.chat.completions.create(
            model=fallback_model,
            messages=messages
        )
        return response.choices[0].message.content
    except Exception as fatal_error:
        raise RuntimeError("FATAL: Primary and Fallback model calls both failed.") from fatal_error
```

---

## 4. Enterprise Security: Input Sanitization, PII Redaction, and Guardrails

To deploy AI systems securely, you must validate and sanitize data at both the entry and exit points of your application.

```
User Query ──► [1. Input Validator] ──► [2. PII Redaction] ──► LLM ──► [3. Output Guardrail] ──► User
                 (Blocks Injection)       (Masks emails/keys)           (Verifies safety)
```

### 1. Input Sanitization (Adversarial Detection)
* **Goal:** Detect and block prompt injections, system override attempts, or excessive input lengths.
* **Implementation:** Use length checks, string matching, and automated classifiers to scan incoming user queries before inserting them into your prompt templates.

### 2. PII Redaction
* **Goal:** Prevent sensitive user data (e.g., credit card numbers, passwords, emails, API keys) from being sent to external third-party model APIs.
* **Implementation:** Use regex or Named Entity Recognition (NER) models to identify and replace PII with placeholder tags (e.g., replacing `john.doe@email.com` with `[REDACTED_EMAIL]`) before sending the data to the LLM.

```python
import re

def redact_sensitive_pii(text: str) -> str:
    """Masks emails and credit card numbers from raw inputs."""
    # Simple regex models
    email_pattern = r'[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+'
    card_pattern = r'\b(?:\d[ -]*?){13,16}\b'
    
    redacted = re.sub(email_pattern, "[REDACTED_EMAIL]", text)
    redacted = re.sub(card_pattern, "[REDACTED_CARD]", redacted)
    return redacted
```

### 3. Output Guardrails (Verification)
* **Goal:** Verify that the generated output is on-topic, safe, and free from sensitive system secrets.
* **Implementation:** Programmatically scan the output text or use a lightweight guard model (e.g., Llama Guard) to classify the safety and policy alignment of the response before returning it to the user.

---

## 5. Observability, Latency Metrics & Execution Tracing

To debug and optimize complex AI pipelines, you must move beyond simple logging and implement structured metric collection and execution tracing.

### Structured Metric Collection
Every LLM call should be measured and logged with precise metadata:
* **Time-to-First-Token (TTFT):** The time elapsed between sending the request and receiving the first generated token. Highly critical for streaming interfaces.
* **Tokens Per Second:** The speed of generation, calculated as $\frac{\text{Completion Tokens}}{\text{Latency in seconds}}$.
* **Cost Tracking:** The financial cost of the call, calculated dynamically based on input and output token counts.

### Centralized Execution Tracing
For multi-step pipelines (like a RAG search that retrieves chunks, rerankes them, and then runs an agent loop), a single request can trigger dozens of model calls and database queries. 

Centralized tracing platforms (such as **LangSmith** or **Arize Phoenix**) serialize and track these execution chains, generating clean visual graphs that show exactly which step failed, which node caused high latency, or where costs spiked.

```
LangSmith Execution Trace:
└─ [1. PDF Query Agent] (Latency: 2.1s, Cost: $0.004)
   ├─ [2. Document Retrieval] (Latency: 0.3s)
   │  └─ [3. Cosine Vector Search] (Latency: 0.1s)
   └─ [4. ReAct Reasoning Loop] (Latency: 1.8s)
      ├─ [5. Tool Call: compute_metrics] (Latency: 0.2s)
      └─ [6. Synthesis Generation] (Latency: 0.6s)
```

---

## 🔨 Hands-On Production Labs

In this module's labs, you will design and build enterprise-grade guardrails:

1. **The Semantic Caching Proxy:** Build a local API server using FastAPI and Redis that intercepts incoming queries, embeds them, and uses a similarity lookup to return cached LLM responses.
2. **Developing the Input/Output Security Gateway:** Build a middleware pipeline that sanitizes inputs for prompt injections, redacts PII using regex, and validates outputs against strict formatting and safety rules.
3. **Pipeline Instrumentation & Latency Tracker:** Implement a structured decorator in Python that measures and logs detailed metrics for every model call: prompt and completion tokens, total cost, latency, and tokens-per-second.

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Consolidate your systems-engineering understanding of semantic caching, exponential backoffs, rate limiters, input guardrails, and observability tools with 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md)
* **Objective:** Build a resilient, cost-optimized, and observable production pipeline. You must build a FastAPI endpoint that integrates a Redis-based exact match cache, sanitizes inputs, executes model calls through an exponential backoff retry loop, falls back to a cheaper model if the primary model fails, and records detailed latency, cost, and usage metrics to a local log file.
