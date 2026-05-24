# Module 06 — Agentic AI Fundamentals

**Duration:** 3 Hours | **Session:** Weekend 3, Sunday (June 1, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Distinguish Chatbots from Agents:** Understand the fundamental paradigm shift from static, single-turn chat utilities to autonomous, multi-step state loops.
* **Master the Agent Anatomy:** Deconstruct the four foundational pillars of an agent: Brain (LLM), Memory Systems, Tool Integrations, and Planning/Reflection protocols.
* **Implement the ReAct Loop:** Gain a thorough understanding of the Reasoning and Acting pattern, mapping the cyclic progression of Thought $\rightarrow$ Action $\rightarrow$ Observation.
* **Deconstruct Function Calling Internals:** Master the raw JSON handshake protocols that coordinate the tool-execution cycle between LLMs and client code.
* **Design Complex Agent Flowcharts:** Conceptualize single-agent systems, multi-agent collaborations, supervisor architectures, and planner-executor loops.

---

## 🗺️ Topics Covered

1. [Chatbots vs. Agents: The Autonomy Paradigm](#1-chatbots-vs-agents-the-autonomy-paradigm)
2. [The Core Pillars of an Agent: Brain, Memory, Tools, Planning](#2-the-core-pillars-of-an-agent-brain-memory-tools-planning)
3. [The ReAct Pattern: Reasoning & Acting in Action](#3-the-react-pattern-reasoning--acting-in-action)
4. [Tool Calling Protocols: Under the Hood](#4-tool-calling-protocols-under-the-hood)
5. [Enterprise Agent Architectures and Topologies](#5-enterprise-agent-architectures-and-topologies)

---

## 1. Chatbots vs. Agents: The Autonomy Paradigm

Most initial Generative AI applications are **static pipelines** or **simple chatbots** (e.g., standard RAG chatbots, single-turn text templates). While powerful, these systems are highly rigid. They follow pre-defined, linear code paths. If a document chunk retrieved during a RAG search contains a spelling error or lacks a specific key, the system has no way to detect the failure, try a different search query, or correct its own mistake.

An **AI Agent** represents a transition from linear scripts to **dynamic state loops**. It uses the LLM as a continuous reasoning engine that dynamically determines the execution path based on the objective, available tools, and runtime environment feedback.

```
STATIC PIPELINE (Linear, Deterministic):
User Input ──► Fetch Embeddings ──► Query ChromaDB ──► Render Prompt ──► Output

AGENTIC LOOP (Cyclic, Stateful, Adaptive):
User Goal ──► Formulate Plan ──► Run Tool ──► Check Result ──► Reflect & Self-Correct ──► Complete
                ▲                                                  │
                └─────────────────── [Iterate Loop] ───────────────┘
```

### Comparative Analysis: Chatbots vs. Agents

| Architectural Dimension | Legacy Chatbot | Stateful AI Agent |
| :--- | :--- | :--- |
| **Operational Trigger** | Prompt message expecting a single answer | High-level objective or goal statement |
| **Execution Path** | Deterministic, single-turn | Dynamic, multi-step execution loop |
| **Tool Execution** | None (pure text in, text out) | Active, autonomous invocation of external resources |
| **State Retention** | Short-term conversation buffer | Short-term + persistent long-term episodic memory |
| **System Behavior** | Passive autocomplete responder | Active planning, self-correction, and goal tracking |

---

## 2. The Core Pillars of an Agent: Brain, Memory, Tools, Planning

Every production agent consists of four core computational components:

```
┌────────────────────────────────────────────────────────────────────────┐
│                              AGENT BRAIN                               │
├───────────────────┬───────────────────┬────────────────┬───────────────┤
│    1. REASONING   │     2. MEMORY     │   3. PLANNERS  │   4. TOOLS    │
│  (Foundation LLM) │ (Context/Episodic)│ (Reflective)   │ (Function APIs)│
└───────────────────┴───────────────────┴────────────────┴───────────────┘
```

### 1. The Brain (Foundation LLM)
The LLM serves as the central processing unit. It interprets the user's high-level goal, translates observations into next steps, parses tool outputs, and formats the final answer. High-capacity models with excellent instruction-following capabilities (such as Claude 3.5 Sonnet or GPT-4o) are essential for serving as an agent's brain.

### 2. Memory Systems
An agent requires structured memory buffers to maintain coherence across complex, multi-step execution loops:
* **Short-Term Memory (In-Context Buffer):** The active conversation history and tool execution logs. This is passed to the LLM during every loop iteration.
* **Long-Term Memory (Episodic Storage):** A persistent database (often a vector database) storing summaries of past execution runs, allowing the agent to recall previous strategies across separate sessions.
* **Semantic Memory (Knowledge Base):** Access to proprietary business logic or documentation via RAG search.

### 3. Planners and Reflection Engines
* **Planning (Sub-Goal Decomposition):** Breaking a massive objective down into logical sub-tasks before executing them.
* **Reflection (Self-Correction):** Analyzing output logs to detect errors (e.g., an empty API response or a syntax error in generated code). The agent uses this feedback to adjust its plan, try different parameters, or seek human intervention.

### 4. Tool Integrations (Capabilities)
Tools are the agent's hands. They are standard Python functions that execute operations outside the model's text space:
* Web search tools (DuckDuckGo, Tavily).
* Code execution sandboxes.
* Database query clients.
* Filesystem APIs.

---

## 3. The ReAct Pattern: Reasoning & Acting in Action

First detailed by Yao et al. in 2022, the **ReAct (Reasoning + Acting)** pattern is the foundational execution loop of modern agents. It guides the LLM to alternate between generating structured reasoning traces (**Thoughts**) and executing external operations (**Actions**), recording the output (**Observations**).

```
┌──────────────────────────────────────────────────────────┐
│                      THE ReAct LOOP                      │
├──────────────────────────────────────────────────────────┤
│                  User Objective (Goal)                   │
│                            │                             │
│                            ▼                             │
│       ┌──────────────────────────────────────────┐       │
│       │ Thought: "What is my next logical step?" │       │
│       └────────────────────┬─────────────────────┘       │
│                            │                             │
│                            ▼                             │
│       ┌──────────────────────────────────────────┐       │
│       │ Action: Call tool with specific args     │       │
│       └────────────────────┬─────────────────────┘       │
│                            │                             │
│                            ▼                             │
│       ┌──────────────────────────────────────────┐       │
│       │ Observation: Ingest tool execution output│       │
│       └────────────────────┬─────────────────────┘       │
│                            │                             │
│                            ▼                             │
│         Is goal achieved? ──► [Yes] ──► Final Answer     │
│                            │                             │
│                          [No]                            │
│                            ▼                             │
│                      Iterate Loop                        │
└──────────────────────────────────────────────────────────┘
```

### Step-by-Step ReAct Execution Trace
Below is a trace of a ReAct agent answering: *"What is the current stock price of Apple, and how does it compare to its 2025 average?"*

```
[Thought 1] 
I need to find the current stock price of Apple (AAPL) and also locate its average stock price in 2025. I will start by searching the web for the current AAPL stock price.

[Action 1] 
call: search_web(query="AAPL stock price today")

[Observation 1] 
"Apple Inc (AAPL) is currently trading at $214.50 as of May 2026."

[Thought 2] 
I have the current price ($214.50). Now I need the average stock price of Apple for the year 2025. I will query the search engine for this historical average.

[Action 2] 
call: search_web(query="AAPL average stock price in 2025")

[Observation 2] 
"The average closing price for Apple stock (AAPL) across the entire year of 2025 was $188.30."

[Thought 3] 
Now I have both values:
Current: $214.50
2025 Average: $188.30
I need to calculate the difference: $214.50 - $188.30 = $26.20 increase.
I will compute the percentage change: ($26.20 / $188.30) * 100 = 13.9%.
I am ready to formulate the final answer.

[Final Answer] 
Apple stock (AAPL) is currently trading at $214.50, which is an increase of $26.20 (or approximately 13.9%) compared to its 2025 average closing price of $188.30.
```

---

## 4. Tool Calling Protocols: Under the Hood

To make tool calling reliable, modern LLMs are trained to output highly specific, structured schema definitions instead of conversational text when they decide to run a tool.

### The JSON Handshake Flow

```
1. CLIENT HANDSHAKE:
Defines available tools as a JSON schema array and sends it alongside the user's prompt.

2. LLM DECISION:
Detects that a tool is required. Generates a tool-calling payload:
{
  "finish_reason": "tool_calls",
  "message": {
    "tool_calls": [{
      "id": "call_123xyz",
      "type": "function",
      "function": {
        "name": "search_web",
        "arguments": "{\"query\":\"AAPL stock price today\"}"
      }
    }]
  }
}

3. CLIENT EXECUTION:
Client code catches the payload, parses the arguments, runs the local Python tool function,
and returns the observation payload:
{
  "role": "tool",
  "tool_call_id": "call_123xyz",
  "name": "search_web",
  "content": "AAPL is trading at $214.50"
}

4. FINAL SYNTHESIS:
The client appends this tool observation back to the conversation history and calls the LLM once more.
The LLM reads the observation and generates the final response.
```

---

## 5. Enterprise Agent Architectures and Topologies

When moving beyond basic single-agent loops, developers structure multi-agent systems to solve complex problems through specialized collaboration.

```
1. SINGLE AGENT TOPOLOGY (Generalist):
User ──► Generalist Agent ──► Tools ──► User

2. HIERARCHICAL MULTI-AGENT (Orchestration):
User ──► Supervisor/Manager Agent
              ├── Researcher Agent (Web/DB Tools)
              ├── Writer Agent (Text Editor Tools)
              └── Editor Agent (Grammar/Security Tools)

3. CYCLIC FLOW TOPOLOGY (LangGraph style state transitions):
[Parser Node] ──► [Compiler Node] ──► [Test Sandbox Node] 
       ▲                                     │
       └───── [Fail: Return Traceback] ──────┘
```

* **Single Agent Generalist:** Excellent for simple, self-contained workflows (e.g., fetching system logs or resolving support tickets).
* **Supervisor/Worker Pattern:** A manager agent delegates tasks to highly specialized sub-agents based on their role definitions, reviews their output, and coordinates the final response.
* **Stateful Cyclic Graphs:** Workflows where execution flows through a series of steps (nodes) with conditional logic that can loop back to previous steps based on validation results (e.g., a code generation agent that compiles, runs tests, and loops back to fix errors until the tests pass).

---

## 🔨 Hands-On Production Labs

In this module's labs, you will build and trace a custom agent loop:

1. **The Tool Definitions Registry:** Implement a series of native Python tools (e.g., a secure filesystem writer, a basic calculator, and a local web scaper) and generate their corresponding JSON schemas.
2. **Developing a Pure Python ReAct Engine:** Build a custom ReAct agent loop from scratch in pure Python without using any external frameworks. You will implement the parser, manage conversation state, invoke tools dynamically, and route outputs back to the model.
3. **Debugging and Tracing execution logs:** Set up detailed terminal logger outputs to inspect the exact Query, Key, and Value payloads of the tool-calling handshake.

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Consolidate your conceptual understanding of ReAct loops, tool registries, short-term vs. long-term memory, and function calling handshakes with 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md)
* **Objective:** Implement a fully autonomous ReAct execution loop in pure Python. Your script must initialize a tool registry with a web search tool and a local file writer, execute a multi-step planning loop, catch tool exceptions gracefully, and write a final synthesized research report to a local markdown file.
