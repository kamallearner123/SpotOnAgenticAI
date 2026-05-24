# Module 07 — Building AI Agents

**Duration:** 4 Hours | **Sessions:** Weekend 4, Sat & Sun (June 7–8, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Select the Optimal Agent Framework:** Compare the design choices, state models, and control characteristics of CrewAI, LangGraph, and raw LangChain agents.
* **Orchestrate Multi-Agent Crews:** Build declarative, role-playing multi-agent systems using CrewAI, incorporating backstories, strict task targets, and agent-to-agent delegation.
* **Construct Stateful Agent Graphs:** Build cyclic, resilient, and highly deterministic agent systems using LangGraph, incorporating node computations and conditional graph routing.
* **Mitigate Agent Loop Failures:** Design production guardrails: handle malformed parsing, limit iteration counts, enforce timeouts, and manage token consumption.
* **Build Practical Engineering Assistants:** Construct a self-correcting coding assistant and an autonomous multi-agent research team.

---

## 🗺️ Topics Covered

1. [Framework Selection: CrewAI vs. LangGraph vs. LangChain](#1-framework-selection-crewai-vs-langgraph-vs-langchain)
2. [Declarative Teamwork with CrewAI](#2-declarative-teamwork-with-crewai)
3. [Deterministic Stateful Loops with LangGraph](#3-deterministic-stateful-loops-with-langgraph)
4. [Production Resilience: Halting Stuck Agents and Parsing Exceptions](#4-production-resilience-halting-stuck-agents-and-parsing-exceptions)
5. [Hands-On Code Implementations](#5-hands-on-code-implementations)

---

## 1. Framework Selection: CrewAI vs. LangGraph vs. LangChain

Building multi-step agent loops requires choosing an orchestration layer. Frameworks vary based on two main variables: **autonomy** vs. **determinism**.

```
HIGH AUTONOMY (Low Control):
  CrewAI ──────► Agents negotiate tasks dynamically based on roles and backstories.
                 Excellent for content creation, research, and fuzzy workloads.

HIGH DETERMINISM (High Control):
  LangGraph ───► Developers compile a strict State Machine Graph of nodes and edges.
                 Excellent for self-correcting compilers, billing chains, and strict pipelines.
```

### Comprehensive Framework Comparison

| Dimension | CrewAI | LangGraph | LangChain Agents (Legacy) |
| :--- | :--- | :--- | :--- |
| **Primary Design Model** | Role-playing, declarative crews | Stateful, developer-compiled cyclic graphs | Single-agent ReAct loops |
| **State Management** | Internal, managed by framework | Explicit, centralized `State` schemas | Local in-memory conversation list |
| **Execution Control** | Heuristic, autonomous delegation | Highly deterministic, code-driven routing | Probabilistic, dictated by LLM tool calls |
| **Cyclic Routing** | Native (task-to-task transition) | Native (cyclic node-to-node edges) | Brittle (often runs into loop timeouts) |
| **Best Use Cases** | Collaborative research, copywriting | Self-correcting systems, strict enterprise APIs | Single-user assistant utilities |

---

## 2. Declarative Teamwork with CrewAI

**CrewAI** is a declarative framework designed to orchestrate cooperative multi-agent teams. Instead of coding exact execution flows, you define agents as specialists with clear roles, backstories, and goals, and assign them to a sequence of tasks.

### The Role-Play Ingestion Mechanics
Under the hood, CrewAI uses your agent parameter definitions (Role, Goal, Backstory) to construct rich system prompts that prime each agent for its specific task.

```
┌──────────────────────────────────────────────┐
│                  CREWAI AGENT                │
├──────────────────────────────────────────────┤
│ • Role: Senior Research Analyst              │
│ • Goal: Find and catalog raw data...         │
│ • Backstory: You are a critical thinker...   │
└──────────────────────┬───────────────────────┘
                       │ (Compiled System Prompt)
                       ▼
"You are a Senior Research Analyst. Your goal is to find and catalog raw data...
 You are a critical thinker... Your execution should remain highly objective."
```

### Production Multi-Agent Crew Implementation
Here is a complete, runnable pattern for a collaborative research team:

```python
import os
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

# Initialize the central LLM engine
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0.2)

# 1. Define Specialized Agents
researcher = Agent(
    role="Principal Research Analyst",
    goal="Extract verified technical facts and architecture diagrams about software systems.",
    backstory="You are a meticulous, skeptical systems analyst with a background in systems architecture.",
    llm=llm,
    verbose=True,
    allow_delegation=True # Allows the agent to ask the writer or editor for feedback
)

writer = Agent(
    role="Technical Content Synthesizer",
    goal="Translate raw system architectures into developer-friendly Markdown documentation.",
    backstory="You are an expert technical writer. You specialize in crisp, clean Markdown layouts.",
    llm=llm,
    verbose=True,
    allow_delegation=False
)

# 2. Define Sequential Tasks
task_research = Task(
    description="Research the core operational differences between CrewAI and LangGraph. Focus on state handling.",
    expected_output="A bulleted technical analysis detailing memory models and execution flow control.",
    agent=researcher
)

task_write = Task(
    description="Summarize the researcher's findings into a highly structured, professional Markdown document.",
    expected_output="A formatted Markdown report containing comparison tables.",
    agent=writer
)

# 3. Assemble and Kickoff the Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[task_research, task_write],
    process=Process.sequential, # Tasks execute in order, passing context down the pipeline
    verbose=True
)

output_report = crew.kickoff()
print(f"\n[Generated Output Report]:\n{output_report}")
```

---

## 3. Deterministic Stateful Loops with LangGraph

When your agent workflow requires strict control (e.g., *“Run the code $\rightarrow$ if compile fails, send traceback back to coder; if compile passes, deploy”*), CrewAI's declarative model is too unpredictable. **LangGraph** models the workflow as a strict **State Machine Graph**.

```
                 ┌───────────────┐
                 │  Enter Graph  │
                 └───────┬───────┘
                         │
                         ▼
                  ┌─────────────┐
                  │ Coder Node  │◄───────────────────┐
                  └──────┬──────┘                    │
                         │                           │
                         ▼                           │
                 ┌──────────────┐                    │
                 │ Sandbox Node │                    │
                 └──────┬───────┘                    │ (Fail: Return error)
                        │                            │
                        ▼                            │
             [Conditional Router Edge]               │
            /                         \              │
           /                           \             │
  (Pass: Exit Graph)             (Fail: Fix Code)    │
         /                               \           │
        ▼                                 ▼          │
      [END]                         [Compiler Node] ─┘
```

### Core Architecture Components
1. **Central State (TypedDict):** A centralized database schema tracking variables, conversation history, and status flags across all nodes.
2. **Nodes (Execution Steps):** Standard Python functions that take the current `State` as input, execute logic (e.g., an LLM call or running a script), and return *updated* state fields.
3. **Edges (Transitions):** Connections routing execution from one node to the next.
4. **Conditional Edges (Routing Logic):** Python helper functions that analyze the current `State` values to dynamically route the execution path (e.g., choosing whether to exit the loop or run a self-correction step).

### Stateful Self-Correction Implementation

```python
from typing import TypedDict, List
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

# 1. Define the Central Centralized State Schema
class CodeState(TypedDict):
    code: str
    error_log: str
    iteration_count: int

# Initialize LLM
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0.1)

# 2. Define Node Computations
def coder_node(state: CodeState) -> dict:
    """LLM attempts to write or fix the python function based on error history."""
    print("[*] Coder Node: Analyzing state and generating python code...")
    prompt = f"Write a clean python function 'divide(a, b)' that handles ZeroDivisionError."
    if state["error_log"]:
        prompt += f"\nYour previous code failed with error:\n{state['error_log']}\nFix the bug."
        
    response = llm.predict(prompt)
    return {
        "code": response, 
        "iteration_count": state["iteration_count"] + 1
    }

def executor_node(state: CodeState) -> dict:
    """Executes the generated code inside a try/catch loop."""
    print("[*] Executor Node: Evaluating python function execution...")
    # Safe evaluation fallback logic simulator
    if "try" not in state["code"]:
        return {"error_log": "ZeroDivisionError: division by zero. Try/except block is missing."}
    return {"error_log": ""}

# 3. Define Conditional Router Edge
def route_edge(state: CodeState) -> str:
    """Decides if the code is correct or if we have hit the iteration threshold."""
    if not state["error_log"]:
        print("[✓] Logic check passed. Ending graph.")
        return END
    if state["iteration_count"] >= 3:
        print("[✕] Max iterations reached. Halting graph.")
        return END
    print("[!] Errors detected. Routing back to Coder Node.")
    return "coder"

# 4. Construct and Compile Graph
builder = StateGraph(CodeState)

# Register Nodes
builder.add_node("coder", coder_node)
builder.add_node("executor", executor_node)

# Set Logic Flow Transitions
builder.set_entry_point("coder")
builder.add_edge("coder", "executor")

# Bind Conditional Edge
builder.add_conditional_edges(
    "executor",
    route_edge,
    {END: END, "coder": "coder"} # Maps returned keys to actual target node names
)

compiled_app = builder.compile()

# 5. Ingest State and Run
initial_state = {"code": "", "error_log": "", "iteration_count": 0}
final_output = compiled_app.invoke(initial_state)
print(f"\nFinal State Code:\n{final_output['code']}")
```

---

## 4. Production Resilience: Halting Stuck Agents and Parsing Exceptions

When deploying autonomous agents, you must implement safety guardrails to keep them from wasting tokens in infinite loops or crashing on unparseable responses.

```python
# Production Agent Guardrails:
# 1. Max Iterations (Prevent infinite loops)
# 2. Hard Execution Timeout (Prevent API hangs)
# 3. Handle Parsing Errors Gracefully
```

### Core Production Guardrails
1. **Infinite Loop Protection (`max_iterations`):** Always cap the maximum number of loops an agent can perform. If an agent gets stuck in a cycle (e.g., *Search web $\rightarrow$ get error $\rightarrow$ Search web $\rightarrow$ get error*), it will run continuously until it consumes your entire API token budget.
2. **Execution Timeouts (`max_execution_time`):** Enforce strict execution timeouts on all agent processes to prevent system hangs.
3. **Graceful Parsing Recovery (`handle_parsing_errors`):** If the LLM generates a malformed tool call, configure your parser to catch the exception and send a structured error message *back to the LLM* (e.g., *"JSON Decode Error: Missing close bracket. Please retry with correct syntax"*), allowing the model to self-correct.

---

## 5. Hands-On Code Implementations

### Project A: Autonomous Coding & Execution Assistant

This agent takes a coding goal, writes a script, runs it in a subprocess, parses errors, and dynamically refactors the code until it runs successfully.

```python
import subprocess
import os
import sys
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

class CodeSandboxAgent:
    def __init__(self, target_filepath: str = "temp_runner.py", max_retries: int = 4):
        self.client = OpenAI()
        self.filepath = target_filepath
        self.max_retries = max_retries

    def generate_code(self, prompt: str, error_log: str = "") -> str:
        system_instructions = (
            "You are an automated coding bot. Generate ONLY raw, executable Python code. "
            "Do not wrap your output in markdown code blocks like ```python. Do not write explanations."
        )
        user_content = f"Task: Write a script to {prompt}."
        if error_log:
            user_content += f"\nYour previous execution failed with this traceback:\n{error_log}\nFix the bug."

        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": system_instructions},
                {"role": "user", "content": user_content}
            ],
            temperature=0.1
        )
        return response.choices[0].message.content.strip()

    def execute_in_sandbox(self) -> str:
        """Runs the generated file in a subprocess sandbox, capturing standard output and error tracebacks."""
        try:
            result = subprocess.run(
                [sys.executable, self.filepath],
                capture_output=True,
                text=True,
                timeout=5
            )
            if result.returncode != 0:
                return result.stderr # Return traceback if compile or runtime failed
            return "" # Return empty string on success
        except subprocess.TimeoutExpired:
            return "TimeoutExpired: The script execution exceeded the 5-second safety limit."

    def run_self_correction_loop(self, user_goal: str):
        error_log = ""
        for attempt in range(1, self.max_retries + 1):
            print(f"[*] Attempt {attempt}/{self.max_retries}...")
            # 1. Write Code
            raw_python = self.generate_code(user_goal, error_log)
            
            with open(self.filepath, "w") as f:
                f.write(raw_python)
                
            # 2. Run Code
            error_log = self.execute_in_sandbox()
            
            if not error_log:
                print(f"[✓] Code executed successfully on attempt {attempt}!")
                print("------------------------------------------")
                with open(self.filepath, "r") as f:
                    print(f.read())
                print("------------------------------------------")
                # Clean up local file
                os.remove(self.filepath)
                return
                
            print(f"[!] Compilation failed: {error_log.strip()}")
            
        print("[✕] Failed to resolve logic errors within maximum retry limit.")
        if os.path.exists(self.filepath):
            os.remove(self.filepath)

# Execution Entry Point
if __name__ == "__main__":
    agent = CodeSandboxAgent()
    agent.run_self_correction_loop("create a list of 5 elements, pop index 8 and print the element.")
```

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Consolidate your conceptual understanding of multi-agent delegation, Graph States, edges, conditional loops, and agent error guardrails with 10 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md) | 🏆 Mini Project 02: Research Agent
* **Objective:** Complete your second major graded project: the **Stateful Research Agent**. Using LangGraph or CrewAI, build a multi-agent system consisting of a Research Analyst (equipped with search tools) and a Technical Editor. The crew must autonomously plan a research vector, retrieve web-scale data on a selected technology, analyze findings, pass the output to the editor for formatting validation, and output a production-grade PDF or Markdown document with verified citations.
