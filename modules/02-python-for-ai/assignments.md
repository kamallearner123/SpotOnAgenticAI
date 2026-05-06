# Module 02 — Assignment

## Assignment 02: Build a Multi-turn Chatbot with Personality

**Due:** May 30, 2026 (before Session 5)  
**Submission:** Push to your GitHub

---

## Task 1: Enhanced Multi-turn Chatbot (50 points)

Build on the simple chatbot from class. Your chatbot must:

1. Accept a "persona" from the user at startup (e.g., "Python expert", "Socratic teacher")
2. Maintain conversation history properly
3. Show token usage after each response
4. Allow the user to type `/clear` to reset conversation history
5. Allow the user to type `/save` to save the conversation to a text file
6. Allow the user to type `/history` to show the full conversation so far

```python
# Starter structure
from openai import OpenAI
import os, json
from datetime import datetime

client = OpenAI()

def create_system_prompt(persona: str) -> str:
    return f"You are {persona}. Maintain this persona throughout the entire conversation."

def save_conversation(history: list, persona: str):
    filename = f"chat_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
    with open(filename, "w") as f:
        f.write(f"Persona: {persona}\n{'='*50}\n\n")
        for msg in history:
            if msg["role"] != "system":
                role = "YOU" if msg["role"] == "user" else "AI"
                f.write(f"{role}: {msg['content']}\n\n")
    print(f"Saved to {filename}")

def main():
    persona = input("Enter AI persona (e.g., 'a senior Python developer'): ").strip()
    history = [{"role": "system", "content": create_system_prompt(persona)}]

    print(f"\nChatting with: {persona}")
    print("Commands: /clear, /save, /history, /quit\n")

    while True:
        user_input = input("You: ").strip()
        # TODO: implement command handling and API calls
        ...

if __name__ == "__main__":
    main()
```

---

## Task 2: PDF Summarizer (30 points)

Build a script that:
1. Accepts a PDF file path as a command-line argument
2. Reads the PDF
3. Splits it into sections (by page or by paragraph)
4. Summarizes each section
5. Produces a final executive summary

```bash
# Usage:
python pdf_summarizer.py report.pdf
```

**Output format:**
```
=== PDF SUMMARY: report.pdf ===
Pages: 12 | ~4,500 words | ~6,000 tokens

Section 1 (Pages 1-3): Introduction
→ [summary of first 3 pages]

Section 2 (Pages 4-7): Key Findings
→ [summary of pages 4-7]

=== EXECUTIVE SUMMARY ===
[Overall 3-paragraph summary of the entire document]
```

---

## Task 3: Async Parallel Summarizer (20 points)

Modify your PDF summarizer to use `asyncio` so all section summaries are generated **in parallel** (not sequentially). Measure and report the time saved.

```python
import asyncio
import time

# Sequential version: measure time
start = time.time()
for section in sections:
    summary = summarize_section(section)
seq_time = time.time() - start

# Parallel version: measure time
start = time.time()
summaries = asyncio.run(summarize_all_parallel(sections))
par_time = time.time() - start

print(f"Sequential: {seq_time:.1f}s | Parallel: {par_time:.1f}s | Speedup: {seq_time/par_time:.1f}x")
```

---

## Submission Checklist

- [ ] `chatbot.py` — all 6 features working
- [ ] `pdf_summarizer.py` — sequential version
- [ ] `pdf_summarizer_async.py` — parallel version with timing comparison
- [ ] `REPORT.md` — note what speedup you observed and why
