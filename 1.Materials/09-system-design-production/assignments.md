# Module 09 — Assignment

## Assignment 09: Harden Your AI App

**Due:** June 13, 2026

---

## Task: Add Production Features to Your PDF Chatbot

Take your Mini Project 01 (PDF Chatbot) and add:

### 1. Caching (30 points)
Cache LLM responses using a simple dictionary (simulate Redis):
```python
import hashlib

_cache = {}

def cached_answer(collection, question: str) -> str:
    cache_key = hashlib.md5(question.encode()).hexdigest()
    if cache_key in _cache:
        print("[CACHE HIT]")
        return _cache[cache_key]
    result = answer_question(collection, question)
    _cache[cache_key] = result
    return result
```

Demonstrate the cache works by asking the same question twice and showing the speed difference.

### 2. Rate Limiting (25 points)
Implement simple per-user rate limiting:
```python
from collections import defaultdict
import time

request_counts = defaultdict(list)

def check_rate_limit(user_id: str, max_per_minute: int = 10):
    now = time.time()
    minute_ago = now - 60
    # Remove old requests
    request_counts[user_id] = [t for t in request_counts[user_id] if t > minute_ago]
    if len(request_counts[user_id]) >= max_per_minute:
        raise Exception(f"Rate limit: {max_per_minute} requests/minute exceeded")
    request_counts[user_id].append(now)
```

### 3. Guardrails (25 points)
Add input validation:
- Reject questions over 500 characters
- Detect and block prompt injection attempts
- Validate that responses are on-topic

### 4. Logging & Metrics (20 points)
Add structured logging:
```python
import logging, time

logging.basicConfig(
    filename="ai_app.log",
    format="%(asctime)s | %(levelname)s | %(message)s",
    level=logging.INFO
)

def logged_answer(question: str, answer: str, latency_ms: float):
    logging.info(f"Q={question[:50]} | latency={latency_ms:.0f}ms | answer_len={len(answer)}")
```

At the end, print a summary:
```
=== Session Summary ===
Total questions: 12
Cache hit rate: 33%
Avg latency: 1.2s
Blocked requests: 2
```

---

## Submission Checklist

- [ ] `pdf_chatbot_production.py` — hardened version
- [ ] `ai_app.log` — sample log file from a test session
- [ ] `REPORT.md` — describe the tradeoffs you made
