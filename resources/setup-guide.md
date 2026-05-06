# 🛠️ Environment Setup Guide

Complete this **before the first session** on May 17.

---

## 1. Python Installation

Check if Python 3.10+ is installed:
```bash
python --version
# Should show: Python 3.10.x or higher
```

If not, download from [python.org](https://python.org/downloads).

---

## 2. VS Code Setup

1. Download [VS Code](https://code.visualstudio.com/)
2. Install the **Python extension** by Microsoft
3. Install the **Pylance** extension for type checking

---

## 3. Git Setup

```bash
# Check if git is installed
git --version

# Configure (replace with your info)
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

---

## 4. Create a GitHub Account

If you don't have one: [github.com](https://github.com)

---

## 5. Install Ollama (Local LLM)

```bash
# Mac
brew install ollama

# Or download from: https://ollama.com

# Verify installation
ollama --version

# Pull your first model (do this before class — it's 4.7GB)
ollama pull llama3
```

---

## 6. OpenAI API Key (Optional)

If you want to use cloud LLMs:
1. Go to [platform.openai.com](https://platform.openai.com)
2. Create an account
3. Go to API Keys → Create new key
4. Save it somewhere secure

> **Note:** We'll use local Ollama models for most exercises, so this is optional.

---

## 7. Course Python Environment

```bash
# Clone the course repository
git clone https://github.com/[your-trainer]/agentic-ai-course
cd agentic-ai-course

# Create virtual environment
python -m venv venv

# Activate (Mac/Linux)
source venv/bin/activate

# Install all dependencies
pip install -r requirements.txt
```

---

## 8. Required Packages

```bash
pip install openai
pip install langchain langchain-openai langchain-community
pip install chromadb
pip install pypdf
pip install python-dotenv
pip install tiktoken
pip install crewai
pip install langgraph
pip install transformers
pip install duckduckgo-search
pip install streamlit
pip install fastapi uvicorn
pip install redis
pip install requests
pip install sqlparse
```

Or install all at once:
```bash
pip install -r requirements.txt
```

---

## 9. Verify Setup

Run this verification script:

```python
# save as verify_setup.py and run: python verify_setup.py

import sys

print("=== Agentic AI Course - Setup Verification ===\n")

# Check Python version
version = sys.version_info
assert version.major == 3 and version.minor >= 10, f"Need Python 3.10+, got {version}"
print(f"✅ Python {version.major}.{version.minor}")

packages = [
    "openai", "langchain", "chromadb", "pypdf",
    "dotenv", "tiktoken", "crewai", "langgraph",
    "transformers", "streamlit", "fastapi", "requests"
]

for pkg in packages:
    try:
        __import__(pkg.replace("-", "_"))
        print(f"✅ {pkg}")
    except ImportError:
        print(f"❌ {pkg} — run: pip install {pkg}")

print("\n=== Ollama Check ===")
import subprocess
result = subprocess.run(["ollama", "list"], capture_output=True, text=True)
if result.returncode == 0:
    print("✅ Ollama installed")
    if "llama3" in result.stdout:
        print("✅ llama3 model ready")
    else:
        print("⚠️  llama3 not downloaded — run: ollama pull llama3")
else:
    print("❌ Ollama not found — install from ollama.com")

print("\n=== Done ===")
```

---

## 10. requirements.txt

```
openai>=1.30.0
langchain>=0.2.0
langchain-openai>=0.1.0
langchain-community>=0.2.0
chromadb>=0.5.0
pypdf>=4.0.0
python-dotenv>=1.0.0
tiktoken>=0.7.0
crewai>=0.30.0
langgraph>=0.1.0
transformers>=4.40.0
duckduckgo-search>=5.0.0
streamlit>=1.35.0
fastapi>=0.111.0
uvicorn>=0.29.0
redis>=5.0.0
requests>=2.31.0
sqlparse>=0.5.0
```
