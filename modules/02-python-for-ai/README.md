# Module 02 — Python for AI Builders

**Duration:** 2 Hours | **Session:** Weekend 1, Sunday (May 18, 2026)

---

## 🎯 Learning Objectives

- Set up a proper Python environment for AI development
- Call the OpenAI API and handle responses
- Read text from files and PDFs programmatically
- Build a simple command-line chatbot from scratch

---

## Topics Covered

1. [Python Essentials](#1-python-essentials)
2. [Virtual Environments](#2-virtual-environments)
3. [APIs in Python](#3-apis-in-python)
4. [JSON Handling](#4-json-handling)
5. [Async Basics](#5-async-basics)
6. [Hands-on: Build a Chatbot](#6-hands-on)

---

## 1. Python Essentials

```python
# List comprehensions
messages = [m["content"] for m in history if m["role"] == "user"]

# F-strings for prompts
topic = "machine learning"
prompt = f"Explain {topic} in simple terms for a software developer."

# Error handling
try:
    response = client.chat.completions.create(...)
except openai.RateLimitError:
    print("Rate limited — wait and retry")
```

---

## 2. Virtual Environments

```bash
# Create
python -m venv venv

# Activate (Mac/Linux)
source venv/bin/activate

# Install packages
pip install openai langchain chromadb pypdf python-dotenv

# Save dependencies
pip freeze > requirements.txt
```

---

## 3. APIs in Python

```python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is 2+2?"}
    ]
)

print(response.choices[0].message.content)
```

**Secure API key with `.env`:**
```
OPENAI_API_KEY=sk-...your-key...
```
```python
from dotenv import load_dotenv
load_dotenv()
```

---

## 4. JSON Handling

```python
import json

# Parse JSON from LLM response
response_text = '{"sentiment": "positive", "score": 0.92}'
parsed = json.loads(response_text)
print(parsed["sentiment"])  # positive

# Common pattern: ask LLM to respond in JSON
prompt = """
Analyze this review and respond ONLY in JSON:
{"sentiment": "positive|negative|neutral", "score": 0.0-1.0}

Review: "Great product, loved it!"
"""
```

---

## 5. Async Basics

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def get_response(prompt: str) -> str:
    response = await client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

async def main():
    # Run 3 prompts in parallel
    results = await asyncio.gather(
        get_response("Summarize Python"),
        get_response("Summarize JavaScript"),
        get_response("Summarize Rust"),
    )
    for r in results:
        print(r)

asyncio.run(main())
```

---

## 6. Hands-on

### A: PDF Summarizer
```python
from pypdf import PdfReader
from openai import OpenAI

def read_pdf(path: str) -> str:
    reader = PdfReader(path)
    return "\n".join(page.extract_text() for page in reader.pages)

def summarize(text: str) -> str:
    client = OpenAI()
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Summarize in bullet points."},
            {"role": "user", "content": text[:4000]}
        ]
    )
    return response.choices[0].message.content

print(summarize(read_pdf("sample.pdf")))
```

### B: Multi-turn Terminal Chatbot
```python
from openai import OpenAI

client = OpenAI()
history = [{"role": "system", "content": "You are a helpful assistant."}]

print("Chatbot ready. Type 'quit' to exit.\n")

while True:
    user_input = input("You: ").strip()
    if user_input.lower() == "quit":
        break

    history.append({"role": "user", "content": user_input})

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=history
    )

    reply = response.choices[0].message.content
    history.append({"role": "assistant", "content": reply})
    print(f"AI: {reply}\n")
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md)
