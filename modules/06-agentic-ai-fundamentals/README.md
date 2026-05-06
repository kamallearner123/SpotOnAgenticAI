# Module 06 — Agentic AI Fundamentals

**Duration:** 3 Hours | **Session:** Weekend 3, Sunday (June 1, 2026)

---

## 🎯 Learning Objectives

- Understand what an AI agent actually is (vs a chatbot)
- Know the 5 components every agent has
- Understand the ReAct reasoning pattern
- Read and understand agent architecture diagrams

---

## What is an AI Agent?

> An AI Agent is a system that uses an LLM as its brain to **autonomously decide** what actions to take, execute those actions using tools, observe results, and iterate until it achieves a goal.

### Chatbot vs Agent

| | Chatbot | Agent |
|--|---------|-------|
| **Input** | Message | Goal |
| **Output** | Response | Result |
| **Steps** | 1 (single turn) | Many (multi-step) |
| **Tools** | None | Many |
| **Memory** | Short-term only | Short + Long term |
| **Decision making** | None | Autonomous |
| **Example** | "What is Python?" | "Research Python frameworks and write a comparison report" |

---

## The 5 Components of Every Agent

```
┌─────────────────────────────────────────────────────┐
│                    AI AGENT                          │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌─────────────────┐ │
│  │  BRAIN   │   │  MEMORY  │   │     TOOLS       │ │
│  │  (LLM)   │◄──│          │   │  - web_search   │ │
│  │          │   │ Short-   │   │  - run_code     │ │
│  │ Reasons  │   │ term     │   │  - read_file    │ │
│  │ Decides  │   │ Long-    │   │  - send_email   │ │
│  │ Reflects │   │ term     │   │  - query_db     │ │
│  └────┬─────┘   └──────────┘   └────────┬────────┘ │
│       │                                  │          │
│  ┌────▼─────────────────────────────────▼────────┐ │
│  │              PLANNING & REFLECTION             │ │
│  │  Goal → Steps → Execute → Observe → Adjust    │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### 1. Brain (LLM)
The reasoning engine. Decides what to do next, interprets results, generates responses.

### 2. Memory
```
Short-term memory = conversation history (what happened this session)
Long-term memory  = vector DB / database (persistent across sessions)
Semantic memory   = embeddings of past conversations, searchable
Episodic memory   = logs of past actions and their outcomes
```

### 3. Tools
Functions the agent can call. Tools are the agent's hands.
```python
# Tool definition example
def search_web(query: str) -> str:
    """Search the web and return results"""
    ...

def run_python_code(code: str) -> str:
    """Execute Python code and return output"""
    ...

def read_file(path: str) -> str:
    """Read a file and return its contents"""
    ...
```

### 4. Planning
Breaking a complex goal into steps.
```
Goal: "Research and write a blog post about LangGraph"

Plan:
1. Search web for "LangGraph tutorial"
2. Search web for "LangGraph vs LangChain"
3. Read the top 3 results
4. Synthesize findings
5. Write a 500-word blog post
6. Save to file
```

### 5. Reflection
The agent reviews its own outputs and improves them.
```
Initial output → Self-critique → Revised output → Better result
```

---

## The ReAct Pattern

**ReAct = Reasoning + Acting**

The agent alternates between:
- **Thought** → reasoning about what to do
- **Action** → calling a tool
- **Observation** → reading the result
- **Repeat** until task complete

```
User: "What is the current price of Bitcoin?"

[Thought]: I need to search for the current Bitcoin price.
[Action]: search_web("Bitcoin price today")
[Observation]: "Bitcoin (BTC) is trading at $67,423 as of May 2026"

[Thought]: I have the information. I can now answer.
[Final Answer]: "Bitcoin is currently trading at $67,423 USD."
```

### More Complex Example
```
User: "Find any security vulnerabilities in main.py and fix them"

[Thought]: I need to read the file first.
[Action]: read_file("main.py")
[Observation]: [file contents with SQL injection vulnerability]

[Thought]: I found a SQL injection. I should fix it using parameterized queries.
[Action]: write_file("main.py", [fixed code])
[Observation]: File written successfully.

[Thought]: I should run tests to make sure the fix works.
[Action]: run_command("python -m pytest tests/")
[Observation]: "5 tests passed, 0 failed"

[Final Answer]: "Fixed SQL injection vulnerability in line 23. Used parameterized queries. All tests pass."
```

---

## Agent Architectures

### 1. Single Agent
```
User → Agent → Tools → Agent → Response
```
Best for: Simple, self-contained tasks

### 2. Multi-Agent
```
User → Orchestrator Agent
                ├── Research Agent → Web Search Tool
                ├── Writing Agent  → Text Editor Tool
                └── Review Agent   → Grammar Tool
```
Best for: Complex tasks requiring specialization

### 3. Supervisor-Worker Pattern
```
User → Supervisor (decides who does what)
          ├── Worker A (executes sub-task)
          ├── Worker B (executes sub-task)
          └── Worker C (executes sub-task)
              ↓
         Supervisor (aggregates results)
              ↓
           Response
```

### 4. Planner-Executor Pattern
```
User Goal → Planner (creates step-by-step plan)
               ↓
           Executor (runs each step)
               ↓
           Verifier (checks if done)
               ↓
           Response or loop back
```

---

## Tool Calling in Practice

Modern LLMs can natively call tools. You define the tool schema, the model decides when/how to call it.

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the internet for current information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "The search query"
                    }
                },
                "required": ["query"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What is the weather in Hyderabad today?"}],
    tools=tools,
    tool_choice="auto"  # model decides when to use tools
)

# If model chose to use a tool:
if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    function_name = tool_call.function.name
    arguments = json.loads(tool_call.function.arguments)
    # Now execute the tool...
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md)
