# Module 08 — MCP & AI Tool Integration

**Duration:** 1.5 Hours | **Session:** Weekend 4, Sunday (June 8, 2026)

---

## 🎯 Learning Objectives

- Understand what MCP (Model Context Protocol) is and why it matters
- Connect AI to real-world tools: filesystem, browser, databases, APIs
- See demos of AI reading files, executing commands, and interacting with GitHub

---

## What is MCP?

**Model Context Protocol (MCP)** is an open standard (by Anthropic) that defines how AI models communicate with external tools and data sources.

Think of it as **USB for AI** — a universal connector between LLMs and tools.

```
Without MCP:                    With MCP:
Every tool needs custom         One protocol, any tool works
integration code                with any AI model
   
App ←→ Tool 1 (custom)         App ←→ MCP Server 1 (filesystem)
App ←→ Tool 2 (custom)    →    App ←→ MCP Server 2 (database)
App ←→ Tool 3 (custom)         App ←→ MCP Server 3 (GitHub)
```

---

## MCP Architecture

```
┌─────────────────┐         ┌──────────────────────┐
│   AI Model /    │  MCP    │    MCP Server        │
│   Your App      │◄───────►│  (Tool Provider)     │
│                 │ Protocol│                      │
└─────────────────┘         └──────────────────────┘

MCP defines:
- Resources: data sources the AI can read
- Tools: functions the AI can call
- Prompts: pre-built prompt templates
```

---

## MCP in Practice

### Python MCP Client
```python
from anthropic import Anthropic
import subprocess

client = Anthropic()

# MCP filesystem server gives Claude access to your files
result = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Read the README.md file and summarize it"}],
    # MCP servers are configured separately
)
```

### Available MCP Servers (from Anthropic/community)
```bash
# Install official MCP servers
npm install -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-github
npm install -g @modelcontextprotocol/server-postgres
npm install -g @modelcontextprotocol/server-puppeteer  # browser
```

---

## Connecting AI to Real Tools (Without Full MCP)

Even without MCP, you can connect AI to real tools using function calling.

### AI Reads Files
```python
from openai import OpenAI
import os, json

client = OpenAI()

# Tool definitions
tools = [
    {
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read contents of a file",
            "parameters": {
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "File path to read"}
                },
                "required": ["path"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "list_directory",
            "description": "List files in a directory",
            "parameters": {
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "Directory path"}
                },
                "required": ["path"]
            }
        }
    }
]

# Tool implementations
def read_file(path: str) -> str:
    with open(path, "r") as f:
        return f.read()

def list_directory(path: str) -> str:
    files = os.listdir(path)
    return json.dumps(files)

TOOLS_MAP = {"read_file": read_file, "list_directory": list_directory}

# Agentic loop
def run_file_agent(user_request: str):
    messages = [{"role": "user", "content": user_request}]

    while True:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools,
            tool_choice="auto"
        )

        message = response.choices[0].message

        if response.choices[0].finish_reason == "stop":
            return message.content

        # Execute tool calls
        messages.append(message)
        for tool_call in message.tool_calls:
            fn_name = tool_call.function.name
            args = json.loads(tool_call.function.arguments)
            result = TOOLS_MAP[fn_name](**args)

            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result
            })

# Run
print(run_file_agent("List the files in /tmp and read any Python files you find"))
```

### AI Executes Commands
```python
import subprocess

@tool
def run_shell_command(command: str) -> str:
    """
    Execute a shell command safely.
    Only allows safe, read-only commands.
    """
    SAFE_COMMANDS = ["ls", "cat", "grep", "find", "echo", "pwd", "python"]

    # Safety check
    cmd_parts = command.strip().split()
    if not cmd_parts or cmd_parts[0] not in SAFE_COMMANDS:
        return f"Command not allowed: {cmd_parts[0]}"

    result = subprocess.run(
        command,
        shell=True,
        capture_output=True,
        text=True,
        timeout=10
    )
    return result.stdout or result.stderr
```

### AI Interacts with GitHub
```python
import requests

GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")

@tool
def get_github_issues(repo: str) -> str:
    """Get open issues from a GitHub repository."""
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    url = f"https://api.github.com/repos/{repo}/issues?state=open"
    response = requests.get(url, headers=headers)
    issues = response.json()
    return json.dumps([{"title": i["title"], "number": i["number"]} for i in issues[:10]])

@tool
def create_github_issue(repo: str, title: str, body: str) -> str:
    """Create a new issue on GitHub."""
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    url = f"https://api.github.com/repos/{repo}/issues"
    data = {"title": title, "body": body}
    response = requests.post(url, headers=headers, json=data)
    return f"Issue created: {response.json().get('html_url')}"
```

---

## Demo Scenarios

### Demo 1: AI Code Auditor
```
User: "Review all Python files in /project/src and find security issues"

Agent:
1. list_directory("/project/src") → finds 5 .py files
2. read_file("auth.py") → finds hardcoded password
3. read_file("api.py") → finds SQL injection
4. Report: "Found 2 critical security issues..."
```

### Demo 2: AI DevOps Assistant
```
User: "Check the server logs and tell me if there are any errors"

Agent:
1. run_shell_command("tail -n 100 /var/log/app.log")
2. Analyzes output
3. Report: "Found 3 ERROR entries in the last 100 lines..."
```

### Demo 3: GitHub Issue Tracker
```
User: "Look at open issues in myrepo and categorize them by type"

Agent:
1. get_github_issues("username/myrepo")
2. Analyzes all 15 issues
3. Report: "7 bugs, 5 feature requests, 3 documentation issues"
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md)
