# Module 09 — MCQ Quiz

---

**Q1.** Which technique prevents paying for the same LLM call twice?

- A) Rate limiting
- B) Response caching ✅
- C) Token batching
- D) Model fallback

---

**Q2.** In production AI systems, exponential backoff is used to:

- A) Increase the context window over time
- B) Gradually reduce API costs
- C) Wait increasingly longer between retries after failures ✅
- D) Scale the number of worker agents

---

**Q3.** A "model fallback" strategy means:

- A) Using a smaller, cheaper model when the primary model fails ✅
- B) Falling back to a rule-based system when AI fails
- C) Using cached responses when the model is slow
- D) Switching to a local model only

---

**Q4.** Token optimization in RAG means:

- A) Compressing the PDF before indexing
- B) Only sending relevant retrieved chunks (not the full document) to the LLM ✅
- C) Using fewer embedding dimensions
- D) Reducing the number of vector DB queries

---

**Q5.** Which component is responsible for preventing abuse of your AI API?

- A) Vector database
- B) Embedding model
- C) Rate limiting ✅
- D) Prompt template

---

**Q6.** "Guardrails" in AI systems refer to:

- A) Hardware safety features on GPU clusters
- B) Input/output validation and safety checks that keep the AI on-task ✅
- C) The physical server rack barriers
- D) API authentication tokens

---

**Q7.** What does `redis_client.setex(key, 3600, value)` do?

- A) Sets a Redis key that expires in 3600 milliseconds
- B) Sets a Redis key that expires in 3600 seconds (1 hour) ✅
- C) Sets a Redis key that is accessed 3600 times
- D) Creates 3600 Redis connections

---

**Q8.** Observability in AI systems primarily means:

- A) Making the AI visually appealing
- B) Logging, monitoring, and tracing AI system behavior to diagnose issues ✅
- C) Making the AI's weights transparent
- D) Testing the AI with human evaluators

---

**Q9.** When should you use a smaller, cheaper model (like gpt-4o-mini) instead of a large model?

- A) Always — large models waste money
- B) Never — quality is always worth the cost
- C) For simpler tasks like classification, summarization, or templated responses ✅
- D) Only for local deployments

---

**Q10.** The correct order for building a production AI system is:

- A) Deploy → Test → Build
- B) Build prototype → Add caching → Add auth → Add monitoring → Evaluate ✅
- C) Add auth → Build prototype → Deploy → Test
- D) Evaluate → Deploy → Build → Monitor

---

**Score:** ___/10
