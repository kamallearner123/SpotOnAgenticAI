# Module 10 — Capstone Projects

**Duration:** 1.5 Hours | **Session:** Weekend 5, Sunday (June 15, 2026)

---

## 🎯 Capstone Day Goals

- Present your capstone project (8–10 minutes per person)
- Demonstrate working code — not slides
- Receive feedback from trainer and peers
- Celebrate completing the course! 🎉

---

## Capstone Requirements

Your capstone must:
1. ✅ Use at least one LLM (cloud or local)
2. ✅ Use at least 2 of the following: Tools, RAG, Memory, Multi-agent, MCP
3. ✅ Solve a real problem you care about
4. ✅ Include a live demo
5. ✅ Be submitted as a GitHub repository with a clear README

---

## Project Ideas

### 1. 🤖 Personal AI Assistant
```
What: A terminal or web-based personal assistant
Tools: Web search, file management, calendar/notes
Memory: Persistent long-term memory (ChromaDB)
Skills: Remembers your preferences, learns over time

Demo script:
- "Add a reminder to prepare for standup at 9am tomorrow"
- "What did I work on last week?" (reads your notes)
- "Find that email I saved about the AWS migration"
```

### 2. 🎓 Interview Preparation Agent
```
What: AI that conducts mock technical interviews
Tools: Code runner, web search for latest interview trends
Multi-agent: Interviewer agent + Evaluator agent
RAG: Index top interview Q&A from PDFs

Demo script:
- Start a Python interview session
- Answer questions, get real-time feedback
- Receive a performance report at the end
```

### 3. 🛎️ Customer Support Bot
```
What: Domain-specific support chatbot
RAG: Company docs, FAQs, product manuals
Tools: Ticket creation, knowledge base search
Guardrails: Only answers in-scope questions

Demo script:
- "How do I reset my password?"
- "I want a refund" (creates support ticket)
- "What are your business hours?"
```

### 4. ⚙️ AI DevOps Assistant
```
What: CLI agent for infrastructure tasks
Tools: Shell commands, GitHub API, log reader
Memory: Remembers common issues and their fixes
Safety: Requires confirmation before destructive ops

Demo script:
- "Check server logs for errors in last hour"
- "Open a GitHub issue for the memory leak we fixed"
- "What issues did we have last week?"
```

### 5. 📚 AI Teacher / Tutor
```
What: Subject-specific tutoring agent
RAG: Course materials, textbooks
Multi-agent: Explainer + Quiz generator + Progress tracker
Memory: Tracks what student has learned

Demo script:
- "Explain recursion to me"
- "Give me a practice problem"
- "What topics haven't I covered yet?"
```

### 6. 🔬 Autonomous Research Agent
```
What: Agent that researches any topic deeply
Tools: Web search, PDF reader, note taker
Multi-agent: Search → Read → Synthesize → Write
Output: Structured research report

Demo script:
- "Research the current state of quantum computing in 2026"
- Agent autonomously searches, reads 10+ sources, synthesizes
- Produces a 1000-word report with citations
```

---

## Presentation Structure (8–10 minutes)

```
1. Problem Statement (1 min)
   "I built this because..."

2. Architecture (2 min)
   - What components did you use?
   - Show a quick diagram

3. Live Demo (4 min)
   - Show real working code
   - Demonstrate at least 2 features

4. Challenges & Learnings (2 min)
   - What was hard?
   - What would you do differently?

5. Q&A (1 min)
```

---

## Evaluation Criteria

| Criteria | Weight |
|----------|--------|
| Does it actually work? (live demo) | 40% |
| Technical depth (complexity of AI components used) | 25% |
| Problem relevance and creativity | 20% |
| Code quality and documentation | 15% |

---

## Submission Checklist

Before June 15, make sure your repo has:

- [ ] `README.md` with project description and setup instructions
- [ ] `requirements.txt` or `pyproject.toml`
- [ ] `.env.example` (API keys template, never commit real keys)
- [ ] Working code that the trainer can run
- [ ] Brief architecture diagram (even ASCII art is fine)

---

## 🏆 Awards

- **Best Technical Implementation** — Most complex/impressive use of AI components
- **Best Problem Solved** — Most useful/practical project
- **Best Presentation** — Clearest and most engaging demo

---

## Past Inspiration

> "The best capstone projects aren't the most complex — they're the ones that solve a real problem that the builder genuinely had."
>
> — Kamal Kumar Mukiri
