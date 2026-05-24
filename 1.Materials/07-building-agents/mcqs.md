# Module 07 — MCQ Quiz

---

**Q1.** In LangChain, `AgentExecutor` is responsible for:

- A) Defining the LLM model
- B) Running the agent loop: call model, execute tools, repeat until done
- C) Storing the conversation history
- D) Generating embeddings

---

**Q2.** What does `max_iterations=10` in `AgentExecutor` prevent?

- A) The agent from using more than 10 tools
- B) Infinite loops where the agent never stops reasoning
- C) The agent from making more than 10 API calls per minute
- D) The context window from exceeding 10K tokens

---

**Q3.** In CrewAI, a `Task` is assigned to:

- A) The entire crew
- B) A specific Agent
- C) A Tool
- D) The LLM directly

---

**Q4.** LangGraph represents an agent workflow as a:

- A) List of function calls
- B) Decision tree
- C) State machine (graph of nodes and edges)
- D) Database query

---

**Q5.** The `@tool` decorator in LangChain:

- A) Defines a new LLM model
- B) Converts a regular Python function into a LangChain tool the agent can use
- C) Caches tool results
- D) Makes the tool asynchronous

---

**Q6.** In CrewAI, `Process.sequential` means:

- A) All agents work in parallel
- B) Agents take turns in the order they're listed
- C) Tasks are executed one after another in order
- D) A supervisor controls the order dynamically

---

**Q7.** `handle_parsing_errors=True` in AgentExecutor handles what scenario?

- A) The tool returns an error
- B) The model generates a malformed tool call that can't be parsed
- C) The user enters invalid input
- D) The API rate limit is exceeded

---

**Q8.** When using `subprocess.run()` for a code execution tool, what is a critical safety practice?

- A) Always run as root
- B) Allow all commands
- C) Whitelist only safe commands and set a timeout
- D) Disable output capture

---

**Q9.** In LangGraph, `END` represents:

- A) An error state
- B) The terminal node — when reached, the workflow stops
- C) The start node
- D) A tool call

---

**Q10.** Which framework is the best choice for a workflow where you need fine-grained control over exactly what happens after each agent step (conditionals, loops, state)?

- A) CrewAI
- B) LangChain ReAct Agent
- C) AutoGen
- D) LangGraph

---

**Score:** ___/10

---

## Answer Key

| Question | Correct Answer |
| :---: | :---: |
| **Q1** | B |
| **Q2** | B |
| **Q3** | B |
| **Q4** | C |
| **Q5** | B |
| **Q6** | C |
| **Q7** | B |
| **Q8** | C |
| **Q9** | B |
| **Q10** | D |
