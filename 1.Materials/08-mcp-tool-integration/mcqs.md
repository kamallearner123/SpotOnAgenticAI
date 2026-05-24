# Module 08 — MCQ Quiz

---

**Q1.** MCP (Model Context Protocol) was created by:

- A) OpenAI
- B) Google
- C) Anthropic
- D) Meta

---

**Q2.** The best analogy for MCP is:

- A) HDMI — a video/audio connector
- B) USB — a universal connector between devices
- C) Bluetooth — a wireless protocol
- D) HTTPS — a secure communication protocol

---

**Q3.** In MCP, a "Resource" refers to:

- A) A GPU computation unit
- B) A data source the AI can read (like a file or database)
- C) A tool the AI can call
- D) The AI model itself

---

**Q4.** When using function calling to give an AI filesystem access, what is a critical security practice?

- A) Allow read/write access to all files
- B) Run file operations as administrator
- C) Whitelist specific directories and sanitize all paths
- D) Disable error handling for speed

---

**Q5.** Which of the following is NOT an available official MCP server?

- A) `@modelcontextprotocol/server-filesystem`
- B) `@modelcontextprotocol/server-github`
- C) `@modelcontextprotocol/server-openai`
- D) `@modelcontextprotocol/server-postgres`

---

**Q6.** When the AI calls a tool and the tool returns an error, the agent should:

- A) Immediately terminate the session
- B) Ignore the error and continue
- C) Receive the error as an observation and decide how to handle it
- D) Restart the entire conversation

---

**Q7.** What is the main benefit of MCP over custom tool integrations?

- A) MCP tools are always free
- B) MCP only works with Claude
- C) MCP provides a standard protocol so any tool works with any AI
- D) MCP is faster than REST APIs

---

**Q8.** When building an AI tool that executes shell commands, `timeout=10` in `subprocess.run()` ensures:

- A) The command runs at least 10 seconds
- B) Commands that take longer than 10 seconds are terminated
- C) Only 10 commands can run per session
- D) The output is limited to 10 lines

---

**Score:** ___/8

---

## Answer Key

| Question | Correct Answer |
| :---: | :---: |
| **Q1** | C |
| **Q2** | B |
| **Q3** | B |
| **Q4** | C |
| **Q5** | C |
| **Q6** | C |
| **Q7** | C |
| **Q8** | B |
