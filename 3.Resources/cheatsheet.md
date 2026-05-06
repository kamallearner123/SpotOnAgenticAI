# Agentic AI — Quick Reference Cheatsheet

---

## 🔑 OpenAI API

```python
from openai import OpenAI
import os

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# Chat completion
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Your question here"}
    ],
    temperature=0.7,
    max_tokens=500
)
print(response.choices[0].message.content)
print(f"Tokens used: {response.usage.total_tokens}")
```

---

## 🔗 Ollama (Local LLM) 

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

response = client.chat.completions.create(
    model="llama3",  # or mistral, deepseek-coder, phi3:mini
    messages=[{"role": "user", "content": "Hello!"}]
)
```

```bash
# CLI commands
ollama pull llama3
ollama run llama3
ollama list
ollama rm llama3
```

---

## 📦 LangChain LCEL

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

chain = (
    ChatPromptTemplate.from_messages([
        ("system", "You are a {role}."),
        ("user", "{question}")
    ])
    | ChatOpenAI(model="gpt-4o-mini")
    | StrOutputParser()
)

result = chain.invoke({"role": "Python expert", "question": "What is a generator?"})
```

---

## 🗄️ ChromaDB (Vector DB)

```python
import chromadb

# In-memory
client = chromadb.Client()

# Persistent
client = chromadb.PersistentClient(path="./chroma_db")

collection = client.create_collection("my_collection")

# Add
collection.add(documents=["text1", "text2"], ids=["id1", "id2"])

# Query
results = collection.query(query_texts=["search query"], n_results=3)
docs = results["documents"][0]
```

---

## 🤖 LangChain Agent

```python
from langchain.tools import tool
from langchain_openai import ChatOpenAI
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub

@tool
def my_tool(input: str) -> str:
    """Tool description — this is what the LLM sees."""
    return f"Result for: {input}"

llm = ChatOpenAI(model="gpt-4o", temperature=0)
prompt = hub.pull("hwchase17/react")
agent = create_react_agent(llm, [my_tool], prompt)
executor = AgentExecutor(agent=agent, tools=[my_tool], verbose=True, max_iterations=10)

result = executor.invoke({"input": "Your task here"})
```

---

## 👥 CrewAI

```python
from crewai import Agent, Task, Crew, Process

agent = Agent(role="Researcher", goal="Find info", backstory="Expert researcher", llm=llm)
task = Task(description="Research X", expected_output="Findings report", agent=agent)
crew = Crew(agents=[agent], tasks=[task], process=Process.sequential)
result = crew.kickoff()
```

---

## 🔧 OpenAI Tool Calling

```python
tools = [{
    "type": "function",
    "function": {
        "name": "function_name",
        "description": "What this function does",
        "parameters": {
            "type": "object",
            "properties": {
                "param": {"type": "string", "description": "Param description"}
            },
            "required": ["param"]
        }
    }
}]

response = client.chat.completions.create(
    model="gpt-4o", messages=messages, tools=tools, tool_choice="auto"
)

if response.choices[0].finish_reason == "tool_calls":
    for call in response.choices[0].message.tool_calls:
        fn_name = call.function.name
        args = json.loads(call.function.arguments)
        # execute fn_name(**args)
```

---

## 🧩 RAG Pipeline

```python
# 1. Chunk
from langchain.text_splitter import RecursiveCharacterTextSplitter
chunks = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200).split_text(text)

# 2. Index
collection.add(documents=chunks, ids=[f"c{i}" for i in range(len(chunks))])

# 3. Retrieve
results = collection.query(query_texts=[question], n_results=3)
context = "\n\n".join(results["documents"][0])

# 4. Generate
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Answer from context only. Say 'I don't know' if not found."},
        {"role": "user", "content": f"Context:\n{context}\n\nQ: {question}"}
    ]
)
```

---

## 🔁 Retry with Backoff

```python
import time

def with_retry(fn, max_retries=3):
    for attempt in range(max_retries):
        try:
            return fn()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

---

## 💡 Token Estimation

```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4o-mini") -> int:
    enc = tiktoken.encoding_for_model(model)
    return len(enc.encode(text))
```

---

## 🔐 .env Pattern

```python
# .env file (never commit!)
# OPENAI_API_KEY=sk-...

from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")
```

---

## 📊 Model Cost Reference (Approximate)

| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|-------|--------------------|--------------------|
| gpt-4o | $5.00 | $15.00 |
| gpt-4o-mini | $0.15 | $0.60 |
| text-embedding-3-small | $0.02 | — |
| claude-3-5-sonnet | $3.00 | $15.00 |
| ollama (local) | Free | Free |
