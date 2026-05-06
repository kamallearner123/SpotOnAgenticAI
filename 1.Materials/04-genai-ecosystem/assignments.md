# Module 04 — Assignment

## Assignment 04: Ecosystem Exploration

**Due:** May 30, 2026  
**Submission:** GitHub repository

---

## Task 1: Run a Local LLM (40 points)

1. Install Ollama from [ollama.com](https://ollama.com)
2. Pull and run **two different** local models
3. Call both from Python using the OpenAI-compatible API
4. Compare their responses on the same 5 prompts

**Test prompts:**
```
1. "Explain recursion in one paragraph for a junior developer"
2. "Write a Python function that checks if a string is a palindrome"
3. "What is the capital of India?"
4. "Explain the difference between supervised and unsupervised learning"
5. "Write a regex to validate an email address"
```

**Report format:**
```markdown
| Prompt | Llama3 Response | Mistral Response | Winner |
|--------|-----------------|------------------|--------|
| 1      | ...             | ...              | ?      |
```

---

## Task 2: LangChain LCEL Chain (30 points)

Build a LangChain chain that:
1. Takes a topic as input
2. Uses a prompt template to ask for a 3-bullet summary
3. Parses the output
4. Runs for 5 different topics

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an expert teacher. Explain topics clearly and concisely."),
    ("user", "Summarize {topic} in exactly 3 bullet points for a software developer.")
])

model = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

chain = prompt | model | parser

topics = ["Docker", "Redis", "GraphQL", "WebSockets", "OAuth2"]
for topic in topics:
    result = chain.invoke({"topic": topic})
    print(f"\n=== {topic} ===\n{result}")
```

---

## Task 3: HuggingFace Pipeline (30 points)

Use the HuggingFace `transformers` library (no API key needed) to:
1. Run sentiment analysis on 10 product reviews
2. Run text classification to categorize support tickets
3. Generate a short text completion

```python
from transformers import pipeline

# Sentiment
sentiment = pipeline("sentiment-analysis")
reviews = ["Amazing product!", "Broke after 2 days.", "It's okay I guess."]
results = sentiment(reviews)

# Zero-shot classification
classifier = pipeline("zero-shot-classification")
support_ticket = "I can't log into my account since yesterday"
categories = ["billing", "authentication", "bug report", "feature request"]
result = classifier(support_ticket, candidate_labels=categories)

# Text generation
generator = pipeline("text-generation", model="gpt2")
continuation = generator("The future of AI is", max_length=50, num_return_sequences=3)
```

---

## Submission Checklist

- [ ] `local_llm_comparison.py` — comparison table with analysis
- [ ] `langchain_chain.py` — 5 topics processed
- [ ] `huggingface_explore.py` — all 3 pipelines working
- [ ] `REPORT.md` — "Which local model did you prefer and why?"
