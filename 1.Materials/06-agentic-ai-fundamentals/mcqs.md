# Module 06 — MCQ Quiz

---

**Q1.** What is the key difference between a chatbot and an AI agent?

- A) Agents use more expensive models
- B) Agents can autonomously decide actions, use tools, and iterate toward a goal
- C) Chatbots are smarter than agents
- D) Agents only work with local models

---

**Q2.** In the ReAct pattern, "ReAct" stands for:

- A) Real-time Action
- B) Reasoning + Acting
- C) Recursive Agent Calling
- D) Retrieval + Action + Context

---

**Q3.** Which of the following is NOT one of the 5 components of an AI agent?

- A) Brain (LLM)
- B) Memory
- C) Training data
- D) Tools

---

**Q4.** "Long-term memory" in an agent system typically refers to:

- A) The model's training data
- B) The current conversation history
- C) Persistent storage (vector DB or database) that persists across sessions
- D) The maximum context window size

---

**Q5.** In the ReAct pattern, after executing an action (tool call), the agent:

- A) Immediately returns the result to the user
- B) Records the Observation and reasons about what to do next
- C) Calls another tool randomly
- D) Resets its memory

---

**Q6.** A "Supervisor-Worker" multi-agent pattern means:

- A) One agent supervises the LLM's training
- B) A supervisor agent delegates sub-tasks to specialized worker agents
- C) Workers supervise the quality of the supervisor's output
- D) Multiple supervisors coordinate a single worker

---

**Q7.** Tool calling in OpenAI's API is defined using:

- A) Python function decorators only
- B) A JSON schema describing function name, description, and parameters
- C) Markdown comments in the system prompt
- D) A separate API endpoint

---

**Q8.** `finish_reason == "tool_calls"` in an OpenAI API response means:

- A) The model has finished generating its response
- B) There was an error in the tool
- C) The model wants to call a tool instead of generating text
- D) All tools have been exhausted

---

**Q9.** What is "reflection" in an agentic AI system?

- A) The agent copying its own code
- B) The agent evaluating and improving its own outputs
- C) The model's attention mechanism
- D) Logging agent actions to a database

---

**Q10.** The "Planner-Executor" agent architecture separates:

- A) The LLM from the vector database
- B) Training from inference
- C) The step-by-step planning from the actual execution of each step
- D) The user interface from the backend

---

**Score:** ___/10

---

## Answer Key

| Question | Correct Answer |
| :---: | :---: |
| **Q1** | B |
| **Q2** | B |
| **Q3** | C |
| **Q4** | C |
| **Q5** | B |
| **Q6** | B |
| **Q7** | B |
| **Q8** | C |
| **Q9** | B |
| **Q10** | C |
