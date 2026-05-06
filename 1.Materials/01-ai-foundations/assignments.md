# Module 01 — Assignment

## Assignment 01: Token Counter & Temperature Explorer

**Due:** May 23, 2026 (before Session 3)  
**Submission:** Push to your GitHub and share the link

---

## Task 1: Token Counter (30 points)

Write a Python script that:
1. Takes a string as input 
2. Counts its approximate token count (use the `tiktoken` library)
3. Estimates the API cost (assume $0.15 per 1M tokens for gpt-4o-mini)

```python
# Starter code
import tiktoken

def count_tokens(text: str, model: str = "gpt-4o-mini") -> int:
    encoding = tiktoken.encoding_for_model(model)
    tokens = encoding.encode(text)
    return len(tokens)

def estimate_cost(token_count: int, cost_per_million: float = 0.15) -> float:
    return (token_count / 1_000_000) * cost_per_million

# Test with different texts
texts = [
    "Hello, world!",
    "The quick brown fox jumps over the lazy dog",
    open("sample_document.txt").read() if os.path.exists("sample_document.txt") else "No file found"
]

for text in texts:
    tokens = count_tokens(text)
    cost = estimate_cost(tokens)
    print(f"Text: {text[:50]}...")
    print(f"Tokens: {tokens} | Estimated cost: ${cost:.6f}\n")
```

**What to submit:** `token_counter.py` with your implementation and a brief report (in comments or a README) noting what you observed.

---

## Task 2: Temperature Experiment (40 points)

Write a script that calls the OpenAI API with the **same prompt** at 5 different temperature values:

`temperatures = [0.0, 0.3, 0.7, 1.0, 1.5]`

**Prompt to use:**
```
"Write one sentence describing the color blue."
```

For each temperature:
- Run the prompt 3 times
- Record all responses
- Note how they differ

**Expected output format:**
```
Temperature: 0.0
  Run 1: "Blue is the color of a clear sky on a sunny day."
  Run 2: "Blue is the color of a clear sky on a sunny day."
  Run 3: "Blue is the color of a clear sky on a sunny day."

Temperature: 1.5
  Run 1: "Blue is the electric pulse of midnight rain and forgotten jazz."
  Run 2: "Blue shimmers like an ocean holding secrets of distant storms."
  Run 3: "Blue — a word that tastes like cold water and quiet Saturdays."
```

**Analysis (in comments or README):**
- At what temperature did responses stop being repetitive?
- At what temperature did responses become too creative/nonsensical?
- What temperature would you use for a SQL generator? Why?

---

## Task 3: Hallucination Test (30 points)

Design a prompt that makes a real LLM hallucinate. Then, design a follow-up prompt that *grounds* the model to avoid hallucination.

**Steps:**
1. Ask the model about a made-up research paper (fabricate a title and author)
2. Ask the model about an event that hasn't happened yet (use a future date)
3. Ask the model a factual question, but include a false premise

For each: show the hallucinated response, then show how you fixed it with a better prompt.

---

## Submission Checklist

- [ ] `token_counter.py` — working script with output
- [ ] `temperature_experiment.py` — results for all 5 temperatures
- [ ] `hallucination_test.py` — 3 examples with original + fixed prompts
- [ ] Brief `REPORT.md` with your observations (1 paragraph per task)
