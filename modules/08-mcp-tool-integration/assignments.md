# Module 08 — Assignment

## Assignment 08: AI Tool Integration

**Due:** June 13, 2026

---

## Task 1: Build a GitHub Assistant Agent (60 points)

Build an agent with GitHub tools that can:
1. List open issues from any public repository
2. Categorize issues by type (bug, feature, docs, etc.)
3. Generate a priority report
4. (Bonus) Create a new issue with a summary

```python
import requests, json, os
from openai import OpenAI

client = OpenAI()
GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")

def get_issues(repo: str, state: str = "open") -> str:
    headers = {"Authorization": f"token {GITHUB_TOKEN}"} if GITHUB_TOKEN else {}
    url = f"https://api.github.com/repos/{repo}/issues?state={state}&per_page=30"
    response = requests.get(url, headers=headers)
    if response.status_code != 200:
        return f"Error: {response.status_code}"
    issues = response.json()
    return json.dumps([{
        "number": i["number"],
        "title": i["title"],
        "labels": [l["name"] for l in i.get("labels", [])],
        "created_at": i["created_at"]
    } for i in issues])

# Define tool schema and implement the agent loop
# Test with public repos like: "langchain-ai/langchain"
```

---

## Task 2: File System Agent (40 points)

Build a safe file system agent that can:
1. List files in a specified directory
2. Read text files
3. Search for a keyword across all files
4. Create summary notes

**Safety requirements:**
- Only operate within a designated `workspace/` directory
- Reject any path traversal attempts (`../`, absolute paths outside workspace)
- Log all operations

---

## Submission Checklist

- [ ] `github_agent.py` — working GitHub issue analyzer
- [ ] `filesystem_agent.py` — safe file system agent with logging
- [ ] `workspace/` — sample files for testing
- [ ] `REPORT.md` — 1 page on what you learned about tool integration
