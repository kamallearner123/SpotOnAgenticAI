# Module 07 — Building AI Agents

**Duration:** 4 Hours | **Sessions:** Weekend 4, Sat & Sun (June 7–8, 2026)

---

## 🎯 Learning Objectives

- Build real agents using LangChain, CrewAI, and LangGraph
- Implement memory, error handling, and tool registration
- Build a web search agent, coding assistant, and research agent
- Understand agent loops and when to use which framework

---

## Framework Decision Guide

```
Simple task, one agent          → LangChain Agent
Role-based teamwork             → CrewAI
Complex stateful workflow       → LangGraph
Agent-to-agent conversation     → AutoGen
```

---

## Part 1: LangChain Agents

### Basic Agent with Tools
```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_react_agent, AgentExecutor
from langchain.tools import tool
from langchain import hub

# Define tools
@tool
def search_web(query: str) -> str:
    """Search the web for information."""
    # In real use, integrate with SerpAPI or Tavily
    return f"Search results for: {query}"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Error: {e}"

# Create agent
llm = ChatOpenAI(model="gpt-4o", temperature=0)
tools = [search_web, calculate]

prompt = hub.pull("hwchase17/react")

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# Run
result = agent_executor.invoke({"input": "What is 15% of 240?"})
print(result["output"])
```

### Agent with Memory
```python
from langchain.memory import ConversationBufferMemory
from langchain.agents import create_react_agent, AgentExecutor

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)

# Now the agent remembers previous conversations
agent_executor.invoke({"input": "My name is Kamal"})
agent_executor.invoke({"input": "What is my name?"})  # Remembers: Kamal
```

---

## Part 2: CrewAI — Multi-Agent Teams

### Basic Crew Setup
```python
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

# Define agents with roles
researcher = Agent(
    role="Senior Research Analyst",
    goal="Find accurate, up-to-date information on any topic",
    backstory="You are an expert researcher with years of experience finding reliable information.",
    llm=llm,
    verbose=True
)

writer = Agent(
    role="Technical Writer",
    goal="Write clear, engaging content for software developers",
    backstory="You write technical content that is accurate, concise, and developer-friendly.",
    llm=llm,
    verbose=True
)

reviewer = Agent(
    role="Editor",
    goal="Review content for accuracy, clarity, and grammar",
    backstory="You ensure all published content meets high quality standards.",
    llm=llm,
    verbose=True
)

# Define tasks
research_task = Task(
    description="Research the latest developments in LangGraph for AI agents. Focus on key features, use cases, and limitations.",
    expected_output="A structured research report with key findings",
    agent=researcher
)

write_task = Task(
    description="Write a 500-word blog post based on the research findings, targeting Python developers.",
    expected_output="A complete, well-structured blog post in markdown format",
    agent=writer
)

review_task = Task(
    description="Review the blog post for technical accuracy and clarity. Suggest improvements.",
    expected_output="The reviewed and improved blog post",
    agent=reviewer
)

# Create crew
crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research_task, write_task, review_task],
    process=Process.sequential,  # tasks run in order
    verbose=True
)

result = crew.kickoff()
print(result)
```

---

## Part 3: LangGraph — Stateful Agent Workflows

LangGraph lets you build agents as **state machines** — full control over the flow.

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
import operator

# Define state
class AgentState(TypedDict):
    messages: Annotated[list, operator.add]
    next: str

llm = ChatOpenAI(model="gpt-4o-mini")

# Define nodes (steps in the workflow)
def call_model(state: AgentState) -> AgentState:
    """Main reasoning step"""
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState) -> str:
    """Decide: call tools or finish?"""
    last_message = state["messages"][-1]
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tools"
    return END

def call_tools(state: AgentState) -> AgentState:
    """Execute tool calls"""
    # Execute each tool call...
    return {"messages": [...results...]}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("agent", call_model)
workflow.add_node("tools", call_tools)

workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue)
workflow.add_edge("tools", "agent")  # After tools, go back to agent

app = workflow.compile()

# Run
result = app.invoke({
    "messages": [{"role": "user", "content": "Research and summarize agentic AI trends"}]
})
```

---

## Build Projects

### 🔍 Project 1: Web Search Agent

```python
from langchain.tools import tool
from langchain_community.tools import DuckDuckGoSearchRun
from langchain_openai import ChatOpenAI
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub

search = DuckDuckGoSearchRun()

@tool
def web_search(query: str) -> str:
    """Search the web for current information."""
    return search.run(query)

@tool
def summarize(text: str) -> str:
    """Summarize a long piece of text."""
    llm = ChatOpenAI(model="gpt-4o-mini")
    return llm.predict(f"Summarize this in 3 bullet points:\n{text}")

llm = ChatOpenAI(model="gpt-4o", temperature=0)
prompt = hub.pull("hwchase17/react")
agent = create_react_agent(llm, [web_search, summarize], prompt)
executor = AgentExecutor(agent=agent, tools=[web_search, summarize], verbose=True)

result = executor.invoke({"input": "What are the top 3 AI agent frameworks in 2026?"})
```

### 💻 Project 2: Coding Assistant

```python
import subprocess

@tool
def run_python(code: str) -> str:
    """Execute Python code and return output."""
    try:
        result = subprocess.run(
            ["python", "-c", code],
            capture_output=True,
            text=True,
            timeout=10
        )
        return result.stdout or result.stderr
    except subprocess.TimeoutExpired:
        return "Timeout: Code took too long to execute"

@tool
def read_file(path: str) -> str:
    """Read a file and return its contents."""
    try:
        with open(path, "r") as f:
            return f.read()
    except FileNotFoundError:
        return f"File not found: {path}"

@tool
def write_file(path: str, content: str) -> str:
    """Write content to a file."""
    with open(path, "w") as f:
        f.write(content)
    return f"Written to {path}"

# Now create an agent that can read, write, and run code
```

### 📊 Project 3: Research Assistant (CrewAI)

See [projects/03-web-search-agent.md](../../projects/03-web-search-agent.md) for the full implementation.

---

## Advanced: Error Handling in Agents

```python
from langchain.agents import AgentExecutor

executor = AgentExecutor(
    agent=agent,
    tools=tools,
    handle_parsing_errors=True,   # handle malformed tool calls
    max_iterations=10,            # prevent infinite loops
    max_execution_time=60,        # timeout in seconds
    verbose=True,
    early_stopping_method="generate"
)
```

---

## 📝 MCQs → [mcqs.md](./mcqs.md)
## 💻 Assignment → [assignments.md](./assignments.md) | 🏆 Mini Project 02: Research Agent
