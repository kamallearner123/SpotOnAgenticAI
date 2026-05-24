# Module 10 — Capstone Projects

**Duration:** 1.5 Hours | **Session:** Weekend 5, Sunday (June 15, 2026)

---

## 🎯 Capstone Day Goals

By the end of this final module, you will:
* **Present an End-to-End System:** Showcase a production-ready, autonomous AI system to your peers and the trainer.
* **Demonstrate Real Working Code:** Deliver a live, interactive demonstration of your system running in real-time (not just slides).
* **Discuss System Architectures:** Walk through your design choices, state machines, database schemas, and guardrail implementations.
* **Graduate as an AI System Engineer:** Celebrate completing the masterclass and receive your course certification! 🎉

---

## 🗺️ Topics Covered

1. [Capstone Project Tracks & Boilerplate Architectures](#1-capstone-project-tracks--boilerplate-architectures)
2. [Project Rubric and Rigorous Evaluation Criteria](#2-project-rubric-and-rigorous-evaluation-criteria)
3. [Presentation Framework & Architecture Walkthroughs](#3-presentation-framework--architecture-walkthroughs)
4. [Submission Checklist and Production Requirements](#4-submission-checklist-and-production-requirements)

---

## 1. Capstone Project Tracks & Boilerplate Architectures

Your capstone project must integrate at least **two** core architectural patterns built during the course (e.g., combining custom RAG with tool-calling agents, or building stateful multi-agent graphs with secure local sandboxes).

Here are three curated tracks with concrete boilerplate starter skeletons to jumpstart your development:

---

### Track A: The Enterprise PDF Chatbot & RAG Assistant

A system designed to ingest, chunk, embed, and query massive collections of technical documentation with verified citations.

```
┌────────────────────────────────────────────────────────┐
│                      TRACK A PATH                      │
├────────────────────────────────────────────────────────┤
│ Parse PDFs ──► Recursive Split ──► ChromaDB + Reranker │
│                                         │              │
│                                         ▼              │
│ Stream Grounded Answer ◄─────── Grounding Prompt       │
└────────────────────────────────────────────────────────┘
```

#### Key Architecture Components
* **Ingestion:** Page-aware recursive text parser.
* **Database:** ChromaDB or pgvector with an HNSW index.
* **Retrieval:** Cosine semantic similarity + sparse keyword (BM25) matching + Cohere/BGE reranker.
* **UI:** Streamlit or FastAPI background socket API.

#### Structural Boilerplate Setup
```python
# File: RAG_Boilerplate.py
import chromadb
from openai import OpenAI
from pydantic import BaseModel, Field

class CitedResponse(BaseModel):
    answer: str = Field(description="The final answer derived ONLY from the provided context.")
    citations: list[str] = Field(description="List of document identifiers explicitly referenced in the answer.")

class EnterpriseRAGPipeline:
    def __init__(self, collection_name: str = "capstone_docs"):
        self.chroma_client = chromadb.PersistentClient(path="./chroma_db")
        self.client = OpenAI()
        self.collection = self.chroma_client.get_or_create_collection(name=collection_name)

    def retrieve_grounded_context(self, query: str, top_k: int = 5) -> str:
        # Vector retrieval logic goes here
        pass

    def synthesize_answer(self, query: str) -> CitedResponse:
        context = self.retrieve_grounded_context(query)
        
        response = self.client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Answer the user query using only the provided context. Return citations."},
                {"role": "user", "content": f"Context:\n{context}\n\nQuery: {query}"}
            ],
            response_format=CitedResponse
        )
        return response.choices[0].message.parsed
```

---

### Track B: Autonomous Web Search & Deep Research Agent

An agent that takes a research objective, plans its query strategy, browses the web, downloads pages, synthesizes facts, and outputs a structured Markdown report.

```
┌────────────────────────────────────────────────────────┐
│                      TRACK B PATH                      │
├────────────────────────────────────────────────────────┤
│ Goal ──► Planning Loop ──► Web Search ──► Page Scrape  │
│                             ▲                  │       │
│                             └──── Observation ─┘       │
│                                        │               │
│                                        ▼               │
│                            Synthesize Markdown Report  │
└────────────────────────────────────────────────────────┘
```

#### Key Architecture Components
* **Brain:** ReAct reasoning loop.
* **Capabilities:** Tavily/DuckDuckGo API for search, BeautifulSoup/Selenium for scraping, and a local markdown file writer.
* **Memory:** Conversation state buffer combined with a SQLite database to cache past query outcomes.

#### Structural Boilerplate Setup
```python
# File: Research_Agent_Boilerplate.py
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI

class DeepResearchCrew:
    def __init__(self, topic: str):
        self.topic = topic
        self.llm = ChatOpenAI(model="gpt-4o", temperature=0.2)

    def assemble_crew(self) -> str:
        # Define agents with roles and goals
        researcher = Agent(
            role="Information Retrieval Specialist",
            goal="Locate raw, verified facts and data points about the target topic.",
            backstory="You search the web objectively, locating academic papers and reliable reports.",
            llm=self.llm,
            verbose=True
        )
        writer = Agent(
            role="Lead Technical Writer",
            goal="Synthesize raw facts into a comprehensive technical document.",
            backstory="You structure raw notes into beautiful markdown files with clear tables.",
            llm=self.llm,
            verbose=True
        )
        
        # Define sequential tasks
        t1 = Task(description=f"Gather detailed metrics about {self.topic}.", expected_output="Raw bulleted notes.", agent=researcher)
        t2 = Task(description="Draft the final markdown report.", expected_output="A complete Markdown file.", agent=writer)
        
        crew = Crew(agents=[researcher, writer], tasks=[t1, t2], process=Process.sequential)
        return crew.kickoff()
```

---

### Track C: Autonomous Coding & Execution Assistant

An agent that reads software code, executes it inside a subprocess sandbox, redirects stderr to its own prompt context, and iteratively self-corrects until it compiles and runs without errors.

```
┌────────────────────────────────────────────────────────┐
│                      TRACK C PATH                      │
├────────────────────────────────────────────────────────┤
│ Prompt ──► Write Code ──► Subprocess Run ──► Success?  │
│                              ▲                  │      │
│                              └───── Traceback ──┘      │
│                                        │               │
│                                        ▼               │
│                                   Exit Sandbox (END)   │
└────────────────────────────────────────────────────────┘
```

#### Key Architecture Components
* **State Management:** LangGraph cyclic state schema.
* **Nodes:** Code generation, local sandbox compilation, and verification tests.
* **Security:** Command whitelist, process isolation, and runtime timeouts.

#### Structural Boilerplate Setup
```python
# File: Coding_Graph_Boilerplate.py
from typing import TypedDict
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI

class CoderState(TypedDict):
    objective: str
    code: str
    traceback: str
    attempts: int

class CodeAgentGraph:
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4o-mini", temperature=0.1)
        self.graph = StateGraph(CoderState)
        
    def generate_code_node(self, state: CoderState) -> dict:
        # Code generation logic goes here
        pass

    def execute_sandbox_node(self, state: CoderState) -> dict:
        # Subprocess execution and traceback capture logic goes here
        pass

    def route_evaluation(self, state: CoderState) -> str:
        # Routing logic (success/fail/retry) goes here
        pass
```

---

## 2. Project Rubric and Rigorous Evaluation Criteria

Your capstone project will be evaluated by the trainer and graded according to four core enterprise metrics:

```
┌────────────────────────────────────────────────────────────────────────┐
│                         CAPSTONE EVALUATION METRICS                    │
├───────────────────┬──────────────────┬─────────────────┬───────────────┤
│ 1. SYSTEM LAB     │ 2. RESILIENCE    │ 3. SECURITY     │ 4. QUALITY    │
│    (Live Demo)    │   (Exceptions)   │  (Guardrails)   │  (Code Specs) │
│       [40%]       │      [25%]       │      [20%]      │     [15%]     │
└───────────────────┴──────────────────┴─────────────────┴───────────────┘
```

### 1. Functional System Loop (40%)
* *Standard:* The application must execute a complete, successful loop live in front of the cohort. Slides and static screenshots are not acceptable. The system must process real, unscripted user input and successfully generate the expected outcome.

### 2. Error Resilience & Exception Handling (25%)
* *Standard:* How does the system handle network dropouts, rate limit spikes (HTTP 429), or empty search queries? The system should implement robust error recovery (exponential backoffs with jitter, model fallbacks, retry parameters, and validation recovery).

### 3. API Security & Guardrails (20%)
* *Standard:* The system must implement robust security practices (never committing API keys to GitHub, sanitizing input vectors for prompt injection attempts, redacting sensitive user PII before sending it to third-party APIs, and running execution sandboxes with strict command whitelists and timeouts).

### 4. Code Quality & Systems Architecture (15%)
* *Standard:* Clear structure of components, clean separation of concern patterns (e.g., separating prompts from main logic), proper logging and observability tracing, and a readable repository with clear environment setup instructions in the `README.md`.

---

## 3. Presentation Framework & Architecture Walkthroughs

On Capstone Day, you will have **8 to 10 minutes** to present your project. Structure your presentation to focus on system engineering:

```
┌──────────────────────────────────────────────────────────┐
│               CAPSTONE PRESENTATION SLICES               │
├──────────────────────────────────────────────────────────┤
│ 1. Problem Statement & Architecture Diagram      [2 min] │
│ 2. Deep Dive: Dynamic Logic & Guardrails         [2 min] │
│ 3. Working Live System Demonstration             [4 min] │
│ 4. Engineering Trade-offs & Future Work          [2 min] │
└──────────────────────────────────────────────────────────┘
```

1. **Problem Statement & Architecture (2 minutes):** Describe the concrete engineering problem your system solves. Walk through your system architecture diagram, explaining the flow of data, state updates, databases, and tool-calling models.
2. **Deep Dive: Logic & Resilience (2 minutes):** Walk through a key, complex section of your code (e.g., your LangGraph state transitions, your custom RAG embedding/reranking loop, or your input sanitization guardrails). Explain the technical design choices behind this implementation.
3. **Live Demonstration (4 minutes):** Demonstrate your working system live. Run a few unscripted test scenarios to show how your agent plans, executes actions, recovers from errors, and delivers the final output.
4. **Engineering Trade-offs & Future Work (2 minutes):** Discuss what was hard, what engineering compromises you had to make (e.g., balancing cost vs. accuracy), and how you would scale the architecture for enterprise production environments.

---

## 4. Submission Checklist and Production Requirements

Ensure your final project repository contains all of the following components before Capstone Day (June 15):

* [ ] **Comprehensive `README.md`:** Detailed description of the system, a clean system architecture diagram, precise environment setup instructions using standard dependency locking tools (`requirements.txt` or `pyproject.toml`), and usage guides.
* [ ] **Environment Security Template (`.env.example`):** A template file documenting required environment variables (e.g., `OPENAI_API_KEY`, `TAVILY_API_KEY`). **Never commit your actual `.env` file containing active private API keys to your repository.**
* [ ] **Robust Code Architecture:** Highly commented, type-safe, and cleanly structured python codebase. Custom tools, prompt templates, database configurations, and application interfaces must be modularized and cleanly separated.
* [ ] **Aesthetic User Interface:** A clean command-line interface with detailed progress logging, or a beautiful web-based interface (e.g., built with Streamlit, FastAPI, or Gradio).

---

## 🏆 Graduation Awards

* **Best Technical Implementation:** Awarded to the project demonstrating the highest technical depth, robust error resilience, and complex, stateful execution graphs.
* **Most Pragmatic System:** Awarded to the project that addresses a high-impact, real-world business problem with a highly cost-efficient and practical architecture.
* **Best Systems Design Presentation:** Awarded to the builder who delivers the clearest architectural breakdown, best documentation, and cleanest live demonstration.
