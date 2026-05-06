# Module 07 — Assignment

## 🏆 Mini Project 02: Research Assistant Agent

**Due:** June 13, 2026  
**This is a graded mini-project**  
**Submission:** GitHub repo + 3-minute demo video

---

## Project Goal

Build a multi-step research assistant that, given a topic, autonomously:
1. Searches the web for information
2. Synthesizes findings from multiple sources
3. Produces a structured research report

---

## Requirements

### Core Features (60 points)
- [ ] Accepts a research topic from the user
- [ ] Uses at least 2 different "search" tool calls (can simulate or use DuckDuckGo)
- [ ] Agent must make at least 3 reasoning steps (Thought → Action → Observation)
- [ ] Produces a structured report with: Summary, Key Findings, and Conclusion
- [ ] Saves the report to a Markdown file

### Bonus Features (30 points — choose 2)
- [ ] Multi-agent: separate Researcher and Writer agents using CrewAI
- [ ] Add a "fact-checker" agent that validates claims
- [ ] Support follow-up questions about the generated report
- [ ] Track sources and include citations in the report

### Code Quality (10 points)
- [ ] README with setup and usage
- [ ] Error handling for failed tool calls
- [ ] Clean agent prompts

---

## Option A: Single Agent (LangChain)

```python
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub
from langchain_community.tools import DuckDuckGoSearchRun
import datetime

search = DuckDuckGoSearchRun()

@tool
def web_search(query: str) -> str:
    """Search the web for current information on a topic."""
    return search.run(query)

@tool
def save_report(title: str, content: str) -> str:
    """Save a research report to a Markdown file."""
    filename = f"reports/{title.replace(' ', '_')}_{datetime.date.today()}.md"
    import os; os.makedirs("reports", exist_ok=True)
    with open(filename, "w") as f:
        f.write(f"# Research Report: {title}\n\n")
        f.write(f"*Generated: {datetime.datetime.now()}*\n\n")
        f.write(content)
    return f"Report saved to {filename}"

llm = ChatOpenAI(model="gpt-4o", temperature=0.3)
tools = [web_search, save_report]
prompt = hub.pull("hwchase17/react")
agent = create_react_agent(llm, tools, prompt)

executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10,
    handle_parsing_errors=True
)

topic = input("Research topic: ")
result = executor.invoke({
    "input": f"""Research the following topic thoroughly and produce a structured report.
    
Topic: {topic}

Your report must include:
1. Executive Summary (2-3 sentences)
2. Key Findings (5+ bullet points)
3. Current Trends
4. Conclusion

Search for information multiple times with different queries. Save the final report."""
})
```

---

## Option B: Multi-Agent (CrewAI)

```python
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI
from langchain_community.tools import DuckDuckGoSearchRun

llm = ChatOpenAI(model="gpt-4o-mini")

researcher = Agent(
    role="Research Analyst",
    goal="Find comprehensive, accurate information on any given topic",
    backstory="You are a thorough researcher who always finds multiple perspectives and current data.",
    tools=[DuckDuckGoSearchRun()],
    llm=llm,
    verbose=True
)

writer = Agent(
    role="Technical Writer",
    goal="Transform research findings into clear, well-structured reports",
    backstory="You write crisp, insightful reports that developers love to read.",
    llm=llm,
    verbose=True
)

critic = Agent(
    role="Editor",
    goal="Ensure report accuracy, completeness, and clarity",
    backstory="You catch gaps in reasoning and ensure technical accuracy.",
    llm=llm,
    verbose=True
)

def create_research_crew(topic: str) -> Crew:
    research_task = Task(
        description=f"Research '{topic}' thoroughly. Find key facts, current trends, and expert opinions. Collect at least 5 distinct findings.",
        expected_output="A detailed list of research findings with supporting evidence",
        agent=researcher
    )

    write_task = Task(
        description="Write a comprehensive research report from the findings. Include Executive Summary, Key Findings, Trends, and Conclusion.",
        expected_output="A complete research report in Markdown format",
        agent=writer
    )

    review_task = Task(
        description="Review the report for accuracy and completeness. Add any missing information. Output the final polished version.",
        expected_output="Final, polished research report in Markdown format",
        agent=critic
    )

    return Crew(
        agents=[researcher, writer, critic],
        tasks=[research_task, write_task, review_task],
        process=Process.sequential,
        verbose=True
    )

topic = input("Research topic: ")
crew = create_research_crew(topic)
result = crew.kickoff()
print(result)
```

---

## Evaluation

```
Agent makes multi-step decisions        25%
Report is well-structured and useful    25%
File is saved correctly                 20%
Code quality and error handling         15%
README and setup instructions           15%
```
