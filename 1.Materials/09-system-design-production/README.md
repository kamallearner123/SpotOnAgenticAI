# Module 09 — AI System Design & Production

**Duration:** 2 Hours | **Session:** Weekend 4, Sunday (June 8, 2026)

---

## 🎯 Learning Objectives

- Design production-grade AI application architectures
- Understand token optimization, latency, and cost management
- Implement security guardrails and rate limiting
- Add observability and monitoring to AI systems

---

## Production AI Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│  Web App  │  Mobile App  │  API Clients  │  Slack Bot       │
└───────────────────────────┬──────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│                        API GATEWAY                           │
│  Rate Limiting  │  Auth  │  Load Balancing  │  Logging      │
└───────────────────────────┬──────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│                      AI APPLICATION                          │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │  Guardrails │  │ Prompt Layer  │  │    Caching         │  │
│  │  (safety)   │  │ (templates)   │  │  (Redis/Memcached) │  │
│  └─────────────┘  └──────────────┘  └────────────────────┘  │
└───────────────────────────┬──────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│                        LLM LAYER                             │
│  Primary: GPT-4o  │  Fallback: GPT-4o-mini  │  Local: Ollama│
└───────────────────────────┬──────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│                      DATA LAYER                              │
│  Vector DB  │  PostgreSQL  │  Redis Cache  │  File Storage  │
└──────────────────────────────────────────────────────────────┘
```

---

## 1. Token Optimization

Tokens = money. Optimize them.

```python
# BAD: Sending the entire document every time
def bad_ask(doc: str, question: str) -> str:
    return client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "user", "content": f"{doc}\n\nQuestion: {question}"}
        ]
    )

# GOOD: Use RAG to send only relevant chunks
def good_ask(collection, question: str) -> str:
    relevant_chunks = retrieve(collection, question, k=3)  # Only 3 chunks
    context = "\n".join(relevant_chunks)
    return client.chat.completions.create(
        model="gpt-4o-mini",  # cheaper model for simpler tasks
        messages=[
            {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {question}"}
        ]
    )
```

### Model Selection Strategy

| Task | Recommended Model | Cost |
|------|-------------------|------|
| Complex reasoning | GPT-4o / Claude 3.5 | High |
| Q&A, summarization | GPT-4o-mini / Haiku | Low |
| Classification | GPT-4o-mini | Very Low |
| Embeddings | text-embedding-3-small | Minimal |
| High volume | Groq / Local Ollama | Near zero |

---

## 2. Caching

Don't pay for the same LLM call twice.

```python
import hashlib
import redis
import json

redis_client = redis.Redis(host="localhost", port=6379)
CACHE_TTL = 3600  # 1 hour

def cached_llm_call(prompt: str, model: str = "gpt-4o-mini") -> str:
    # Create cache key from prompt hash
    cache_key = f"llm:{hashlib.md5(prompt.encode()).hexdigest()}"

    # Check cache
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)

    # Make API call
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    result = response.choices[0].message.content

    # Cache the result
    redis_client.setex(cache_key, CACHE_TTL, json.dumps(result))

    return result
```

---

## 3. Rate Limiting

Protect your API costs and prevent abuse.

```python
from fastapi import FastAPI, HTTPException, Depends
from slowapi import Limiter
from slowapi.util import get_remote_address

app = FastAPI()
limiter = Limiter(key_func=get_remote_address)

@app.post("/api/chat")
@limiter.limit("10/minute")  # 10 requests per minute per IP
async def chat(request: ChatRequest):
    ...
```

```python
# Per-user rate limiting with Redis
def check_rate_limit(user_id: str, max_requests: int = 100, window: int = 3600):
    key = f"rate:{user_id}"
    current = redis_client.incr(key)
    if current == 1:
        redis_client.expire(key, window)
    if current > max_requests:
        raise Exception(f"Rate limit exceeded. Try again in {redis_client.ttl(key)} seconds")
```

---

## 4. Guardrails & Safety

```python
# Input validation
def validate_input(user_message: str) -> str:
    # Check length
    if len(user_message) > 2000:
        raise ValueError("Message too long (max 2000 chars)")

    # Check for injection attempts
    injection_patterns = [
        "ignore previous instructions",
        "you are now",
        "forget everything",
        "new system prompt"
    ]
    message_lower = user_message.lower()
    for pattern in injection_patterns:
        if pattern in message_lower:
            raise ValueError("Potentially adversarial input detected")

    return user_message

# Output validation
def validate_output(response: str, topic: str) -> bool:
    """Check if response is on-topic"""
    check_prompt = f"""
    Is this response about {topic}? Answer only YES or NO.
    Response: {response}
    """
    result = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": check_prompt}]
    )
    return "YES" in result.choices[0].message.content.upper()
```

---

## 5. Retry & Fallback

```python
import time
from openai import OpenAI, RateLimitError, APIError

def resilient_llm_call(
    messages: list,
    primary_model: str = "gpt-4o",
    fallback_model: str = "gpt-4o-mini",
    max_retries: int = 3
) -> str:
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model=primary_model,
                messages=messages
            )
            return response.choices[0].message.content

        except RateLimitError:
            wait_time = 2 ** attempt  # exponential backoff: 1s, 2s, 4s
            print(f"Rate limited. Waiting {wait_time}s...")
            time.sleep(wait_time)

        except APIError:
            # Fallback to cheaper model
            print(f"API error on {primary_model}. Falling back to {fallback_model}")
            response = client.chat.completions.create(
                model=fallback_model,
                messages=messages
            )
            return response.choices[0].message.content

    raise Exception("All retry attempts failed")
```

---

## 6. Observability & Logging

```python
import logging
import time
from dataclasses import dataclass

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("ai_app")

@dataclass
class LLMMetrics:
    model: str
    prompt_tokens: int
    completion_tokens: int
    latency_ms: float
    success: bool
    error: str = None

def instrumented_call(messages: list, model: str = "gpt-4o-mini") -> tuple[str, LLMMetrics]:
    start = time.time()

    try:
        response = client.chat.completions.create(model=model, messages=messages)
        content = response.choices[0].message.content
        latency = (time.time() - start) * 1000

        metrics = LLMMetrics(
            model=model,
            prompt_tokens=response.usage.prompt_tokens,
            completion_tokens=response.usage.completion_tokens,
            latency_ms=latency,
            success=True
        )
        logger.info(f"LLM call | model={model} | tokens={metrics.prompt_tokens+metrics.completion_tokens} | latency={latency:.0f}ms")
        return content, metrics

    except Exception as e:
        metrics = LLMMetrics(model=model, prompt_tokens=0, completion_tokens=0, latency_ms=0, success=False, error=str(e))
        logger.error(f"LLM call FAILED | model={model} | error={e}")
        raise
```

---

## How Companies Build Production AI Systems

```
1. Start with a working prototype (LangChain / direct API)
2. Add prompt templates and versioning
3. Implement caching for repeated queries
4. Add rate limiting and auth
5. Add monitoring (LangSmith / custom logging)
6. Set up cost alerts (OpenAI dashboard)
7. Add human-in-the-loop for critical actions
8. Evaluate quality with evals framework
9. A/B test different models/prompts
10. Iterate based on real user feedback
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md)
