# Module 06 — Assignment

## Assignment 06: Design Your Agent

**Due:** June 13, 2026  
**Submission:** GitHub repository

---

## Task 1: Agent Architecture Design (30 points)

Choose ONE real-world problem and design a complete agent architecture:

**Problem options (or propose your own):**
- A DevOps assistant that monitors servers and creates incident tickets
- A content creation agent that researches and writes blog posts
- A code review agent that reads PRs and suggests improvements
- A meeting assistant that takes notes and creates action items

**Deliverable:** A `DESIGN.md` file containing:

```markdown
## Agent Name: [Your name]

## Problem Statement
[2-3 sentences explaining what the agent does and why it's useful]

## Architecture Type
[Single / Multi-agent / Supervisor-Worker / Planner-Executor]

## Components

### Brain
- Model: [which LLM and why]
- Temperature: [value and reasoning]

### Memory
- Short-term: [how conversation history is stored]
- Long-term: [what is persisted and where]

### Tools
| Tool Name | Description | Input | Output |
|-----------|-------------|-------|--------|
| tool_1    | ...         | ...   | ...    |

### Planning Strategy
[How does the agent break down complex goals?]

### Reflection
[Does the agent review its own outputs? How?]

## Example Interaction
[Walk through a complete user request from input to output, showing all Thought/Action/Observation steps]

## Risks & Mitigations
[What could go wrong and how will you handle it?]
```

---

## Task 2: Implement Tool Calling (40 points)

Build a working agent using OpenAI's tool calling API with at least **3 custom tools**.

Suggested toolset for a "Personal Productivity Agent":

```python
from openai import OpenAI
import json, os, datetime

client = OpenAI()

# Tool 1: Create a note
def create_note(title: str, content: str) -> str:
    filename = f"notes/{title.replace(' ', '_')}.txt"
    os.makedirs("notes", exist_ok=True)
    with open(filename, "w") as f:
        f.write(f"# {title}\n\n{content}")
    return f"Note saved: {filename}"

# Tool 2: Search notes
def search_notes(keyword: str) -> str:
    results = []
    if not os.path.exists("notes"):
        return "No notes found"
    for filename in os.listdir("notes"):
        with open(f"notes/{filename}") as f:
            content = f.read()
            if keyword.lower() in content.lower():
                results.append({"file": filename, "excerpt": content[:200]})
    return json.dumps(results) if results else f"No notes found for: {keyword}"

# Tool 3: Get current date/time
def get_datetime() -> str:
    return datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Tool schemas
tools = [
    {
        "type": "function",
        "function": {
            "name": "create_note",
            "description": "Save a note with a title and content",
            "parameters": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "content": {"type": "string"}
                },
                "required": ["title", "content"]
            }
        }
    },
    # Add search_notes and get_datetime schemas...
]

TOOLS_MAP = {
    "create_note": create_note,
    "search_notes": search_notes,
    "get_datetime": get_datetime,
}

def run_agent(user_request: str):
    messages = [
        {"role": "system", "content": "You are a personal productivity assistant. Use tools to help the user manage their notes."},
        {"role": "user", "content": user_request}
    ]

    while True:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools,
            tool_choice="auto"
        )
        msg = response.choices[0].message

        if response.choices[0].finish_reason == "stop":
            return msg.content

        # Process tool calls
        messages.append(msg)
        for tool_call in msg.tool_calls:
            fn = tool_call.function.name
            args = json.loads(tool_call.function.arguments)
            result = TOOLS_MAP[fn](**args)
            print(f"[Tool: {fn}] → {result}")
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result
            })

# Test your agent
print(run_agent("Save a note about today's meeting: discussed RAG pipeline, decided to use ChromaDB"))
print(run_agent("Find my notes about ChromaDB"))
print(run_agent("What time is it and summarize what I should do next?"))
```

---

## Task 3: ReAct Trace Analysis (30 points)

Run an existing agent (you can use LangChain with `verbose=True`) and:
1. Capture the full ReAct trace for 3 different tasks
2. Analyze the trace — where did the agent do well? Where did it get confused?
3. Suggest 2 improvements to the agent's system prompt or tool definitions

**Deliverable:** `react_analysis.md` with the traces and your analysis.

---

## Submission Checklist

- [ ] `DESIGN.md` — complete agent architecture design
- [ ] `productivity_agent.py` — working tool-calling agent
- [ ] `react_analysis.md` — 3 traces with analysis
- [ ] Notes folder with at least 3 sample notes created by the agent
