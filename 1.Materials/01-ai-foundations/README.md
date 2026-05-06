# Module 01 — AI Foundations for Developers

**Duration:** 2 Hours  
**Session:** Weekend 1, Saturday (May 17, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
- Understand the difference between AI, ML, DL, and GenAI without heavy math
- Know what an LLM actually is and how it works at an intuitive level
- Understand tokens, context windows, and embeddings
- Know why ChatGPT sometimes lies (hallucination) and why that matters

---

## 🗺️ Topics Covered

1. [What is AI, ML, DL, GenAI?](#1-what-is-ai-ml-dl-genai)
2. [The Evolution: Rule-based → ML → Deep Learning → LLMs](#2-the-evolution)
3. [What is an LLM?](#3-what-is-an-llm)
4. [Tokens, Embeddings, Context Window](#4-tokens-embeddings-context-window)
5. [Prompt vs Completion](#5-prompt-vs-completion)
6. [Transformers — Intuition Only](#6-transformers-intuition)
7. [Why ChatGPT Works (and Fails)](#7-why-chatgpt-works-and-fails)

---

## 1. What is AI, ML, DL, GenAI?

```
AI  ──────────────────────────────────────────────
  └── Machine Learning (ML) ──────────────────────
        └── Deep Learning (DL) ───────────────────
              └── Generative AI (GenAI) ───────────
                    └── LLMs (GPT, Claude, Gemini)
```

| Term | What It Means (Simple) | Example |
|------|------------------------|---------|
| **AI** | Any system that mimics human intelligence | Spam filter, Chess engine |
| **ML** | AI that learns from data (no explicit rules) | Netflix recommendations |
| **DL** | ML using neural networks (many layers) | Face recognition |
| **GenAI** | AI that *creates* new content | ChatGPT, DALL-E, Sora |
| **LLM** | GenAI specifically for language/text | GPT-4, Claude, Gemini |

### Key Insight
> You don't need to understand neural networks to build powerful AI applications. You need to understand **what goes in (prompt)** and **what comes out (completion)** — and how to engineer both.

---

## 2. The Evolution

| Era | Approach | Problem |
|-----|----------|---------|
| **Rule-based Systems** (1950s–90s) | `if email contains "viagra" → spam` | Breaks with every edge case |
| **Classical ML** (1990s–2010s) | Learn patterns from labeled data | Needs lots of labeled data |
| **Deep Learning** (2012–2017) | Neural networks with many layers | Needs massive compute & data |
| **Transformers / LLMs** (2017–now) | Self-attention over sequences | Works on almost any text task |

---

## 3. What is an LLM?

An LLM (Large Language Model) is a **next-token predictor** trained on billions of text documents.

```
Input:  "The capital of France is ___"
Output: "Paris"  (with 98% probability)
```

That's it. The "intelligence" comes from training on so much text that it learns patterns, facts, reasoning — all as *statistical relationships between tokens*.

**Key properties:**
- **Stateless by default** — no memory between conversations
- **Probabilistic** — same input can give different outputs (temperature)
- **Bounded by training data** — doesn't know events after its cutoff
- **No internet access by default** — unless you give it tools

---

## 4. Tokens, Embeddings, Context Window

### Tokens
LLMs don't read words — they read **tokens** (chunks of characters).

```
"Hello, world!"  →  ["Hello", ",", " world", "!"]  →  4 tokens
"ChatGPT"        →  ["Chat", "G", "PT"]             →  3 tokens
```

> **Rule of thumb:** 1 token ≈ 0.75 words | 1000 tokens ≈ 750 words

**Why it matters:** Every API call charges by tokens. Context is limited by tokens.

### Context Window
The maximum number of tokens the model can "see" at once.

| Model | Context Window |
|-------|----------------|
| GPT-3.5 | 4K tokens (~3K words) |
| GPT-4 | 128K tokens (~96K words) |
| Claude 3.5 | 200K tokens |
| Gemini 1.5 | 1M tokens |

> If your document is longer than the context window → you need **RAG** (Module 05)

### Embeddings
Numbers that represent the *meaning* of text in multi-dimensional space.

```
"king"   →  [0.2, 0.9, -0.1, ...]
"queen"  →  [0.2, 0.8,  0.1, ...]
"apple"  →  [0.7, -0.3, 0.5, ...]
```

Similar meaning → numbers are close together → basis for **semantic search**

---

## 5. Prompt vs Completion

```
┌─────────────────────────────────────────┐
│  PROMPT (what you send)                 │
│  ─────────────────────                  │
│  System: You are a helpful assistant.   │
│  User: Summarize this in 3 bullet pts:  │
│         [article text]                  │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│  COMPLETION (what you receive)          │
│  ─────────────────────                  │
│  • Point 1...                           │
│  • Point 2...                           │
│  • Point 3...                           │
└─────────────────────────────────────────┘
```

**Prompt components:**
- `System prompt` — sets the model's persona and rules
- `User message` — the actual request
- `Context` — background info, documents, history
- `Examples` — few-shot demonstrations

---

## 6. Transformers — Intuition

You don't need to know the math. Know this:

> A transformer reads the **entire input at once** (not word by word), and learns to pay **attention** to the most relevant parts for each prediction.

```
"The animal didn't cross the street because it was too tired"

What does "it" refer to?
→ Transformer: ANIMAL (high attention)
→ Not: street (low attention)
```

This "self-attention" is why LLMs are so much better at language understanding than previous models.

---

## 7. Why ChatGPT Works (and Fails)

### Why it works:
- Trained on **trillions of tokens** — essentially the entire internet
- Fine-tuned with **human feedback (RLHF)** to be helpful and safe
- Self-attention lets it **reason across long context**

### Why it fails:
| Problem | Cause | Fix |
|---------|-------|-----|
| **Hallucination** | Statistically plausible ≠ factually true | Ground with real data (RAG) |
| **Stale knowledge** | Training cutoff date | Connect to live data sources |
| **No memory** | Stateless by default | Implement memory (Module 06) |
| **Can't act** | Text only — no tools | Give it tools (Module 07) |

---

## 🧪 Live Demos

1. **Predict next word** — show how autocompletion works
2. **Temperature experiment** — same prompt, different temperatures (0.0 vs 1.5)
3. **Hallucination** — ask about a fake paper, fake person, future event
4. **Token counter** — show how different words cost different tokens

---

## 📖 Notes

See [notes.md](./notes.md) for detailed lecture notes.

## 📝 MCQs

See [mcqs.md](./mcqs.md) — 12 questions covering all topics.

## 💻 Assignment

See [assignments.md](./assignments.md) — Assignment 01: Token Counter & Temperature Experiment
