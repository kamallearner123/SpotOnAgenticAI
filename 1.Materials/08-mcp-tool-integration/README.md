# Module 08 — MCP & AI Tool Integration

**Duration:** 1.5 Hours | **Session:** Weekend 4, Sunday (June 8, 2026)

---

## 🎯 Learning Objectives

By the end of this module, you will:
* **Deconstruct the MCP Architecture:** Understand the design, abstractions, and standardized integration pathways of the Model Context Protocol (MCP).
* **Map Client-Host-Server Handshakes:** Trace how hosts, clients, and servers coordinate tool execution, resource reading, and prompt hydration.
* **Build a Custom MCP Server:** Write a complete, custom MCP server in Python to expose secure local tools and data resources.
* **Integrate Multi-Server Agents:** Build a client agent capable of connecting to multiple MCP servers concurrently to perform system actions.

---

## 🗺️ Topics Covered

1. [The MCP Paradigm Shift: Standardizing Tool Architectures](#1-the-mcp-paradigm-shift-standardizing-tool-architectures)
2. [MCP Architecture & Core Abstractions](#2-mcp-architecture--core-abstractions)
3. [Building a Custom MCP Server in Python](#3-building-a-custom-mcp-server-in-python)
4. [Integrating MCP Servers with Agent Clients](#4-integrating-mcp-servers-with-agent-clients)

---

## 1. The MCP Paradigm Shift: Standardizing Tool Architectures

Historically, connecting large language models to custom databases, local filesystems, or software APIs required writing highly custom, bespoke tool wrappers for every single application. 

If you integrated a filesystem reader tool for OpenAI, and later migrated your system to Anthropic's Claude or Google's Gemini, you had to rewrite your tool-binding and execution handler scripts.

```
THE Bespoke (Fragmented) INTEGRATION CHAOS:
[App A] ◄─── (Custom SDK integration) ───► [Filesystem Tool]
[App B] ◄─── (Bespoke JSON schemas) ────► [PostgreSQL Tool]
[App C] ◄─── (Custom Python wrappers) ───► [GitHub Tool]

THE MCP STANDARD (USB-like Connection):
[Any App / LLM Host] ◄──── JSON-RPC over Stdio/SSE ────► [Unified MCP Servers]
                                                           ├── Filesystem Server
                                                           ├── Database Server
                                                           └── API Server
```

The **Model Context Protocol (MCP)**, introduced by Anthropic in late 2024, acts as a **"USB connection for AI"**. It establishes an open, standard protocol defining how a client host (your application or an IDE) communicates with a server that exposes capabilities (tools, data inputs, prompts) to any LLM.

---

## 2. MCP Architecture & Core Abstractions

MCP defines a strict three-tier topology coordinating execution flows:

```
┌────────────────────────────────────────────────────────┐
│                        MCP HOST                        │
│             (e.g., Claude Desktop, Cursor)             │
└──────────────────────────┬─────────────────────────────┘
                           │ (Spawns & Coordinates client)
                           ▼
┌────────────────────────────────────────────────────────┐
│                       MCP CLIENT                       │
│       (Establishes JSON-RPC connection over Stdio)     │
└──────────────────────────┬─────────────────────────────┘
                           │ (Standardized Handshake)
                           ▼
┌────────────────────────────────────────────────────────┐
│                       MCP SERVER                       │
│     Exposes: 1. Resources  2. Prompts  3. Tools        │
└────────────────────────────────────────────────────────┘
```

* **The MCP Host:** The environment running the AI application (such as your terminal agent runner or a developer IDE like VS Code/Claude Desktop).
* **The MCP Client:** The component inside the host that negotiates connection setups, serializes JSON-RPC payloads, and forwards them to servers.
* **The MCP Server:** A lightweight background process (running over standard input/output `Stdio` or Server-Sent Events `SSE`) that exposes capabilities directly.

### The Three Core MCP Abstractions

#### 1. Resources (Readable Data Inputs)
Resources represent structured, read-only data sources that the LLM can query for context (e.g., application logs, database schemas, local text documents). Resources are identified using standard URIs:

```
file:///logs/app.log  --> Points to a local file resource
postgres://users/schema  --> Points to a database schema resource
```

#### 2. Prompts (Standardized Templates)
Exposed prompt templates that the server makes available to the host (e.g., an MCP server can expose a `review-code` prompt template, ensuring any host connecting to the server gains access to standard instructions).

#### 3. Tools (Executable Operations)
Executable functions that allow the LLM to write files, run terminal commands, query databases, or call external APIs. Tools expose strict JSON schema parameters, and their execution outputs are returned back through the JSON-RPC interface.

---

## 3. Building a Custom MCP Server in Python

Let's write a complete, custom MCP server in Python. This server uses standard input/output (`Stdio`) to communicate, exposing a secure mathematical calculator tool and a local file scanner to any client.

```
       Host Console ──► (JSON-RPC over Stdio) ──► python mcp_server.py
                                                      │
                                                      ├── call: list_tools()
                                                      └── call: get_system_status()
```

### Complete Custom Python MCP Server Implementation

```python
import sys
import os
import json
import asyncio
from mcp.server.fastmcp import FastMCP

# 1. Initialize the FastMCP Server Instance
mcp = FastMCP("SystemUtilitiesServer")

# 2. Expose a simple executable Tool
@mcp.tool()
def compute_metrics(formula: str) -> str:
    """
    Evaluates basic arithmetic expressions safely.
    Use this for executing simple mathematical calculations.
    """
    # Safe validation subset check
    allowed_chars = set("0123456789+-*/(). ")
    if not all(char in allowed_chars for char in formula):
        return "Error: Invalid or insecure characters in mathematical formula."
    try:
        result = eval(formula)
        return f"Calculation Result: {result}"
    except Exception as e:
        return f"Execution Error: {e}"

# 3. Expose a resource (read-only system information)
@mcp.resource("system://diagnostics")
def get_system_status() -> str:
    """
    Exposes read-only system diagnostic details:
    active path, environment metadata, and current platform status.
    """
    return json.dumps({
        "status": "HEALTHY",
        "current_working_directory": os.getcwd(),
        "python_version": sys.version,
        "platform": sys.platform
    }, indent=2)

# Execution Entry Point
if __name__ == "__main__":
    # Start the server using stdio transport (perfect for CLI agents and Claude Desktop integrations)
    mcp.run()
```

---

## 4. Integrating MCP Servers with Agent Clients

To connect your custom agent code to one or more running MCP servers, build an **MCP Client**. The client spawns the server as a background subprocess and communicates using standard JSON-RPC over `stdio`.

### Client Integration Script

```python
import asyncio
import subprocess
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def run_mcp_client_workflow():
    # 1. Define how to spawn the server subprocess
    server_params = StdioServerParameters(
        command="python",
        args=["mcp_server.py"] # Path to your custom MCP server script
    )
    
    print("[*] Establishing JSON-RPC connection with local MCP server...")
    async with stdio_client(server_params) as (read_stream, write_stream):
        async with ClientSession(read_stream, write_stream) as session:
            # 2. Initialize handshake session
            await session.initialize()
            print("[✓] Handshake initialized successfully!")
            
            # 3. Query available tools on the server
            tools_response = await session.list_tools()
            print("\nExposed Tools on Server:")
            for tool in tools_response.tools:
                print(f" - Tool Name: {tool.name}")
                print(f"   Description: {tool.description}")
                
            # 4. Invoke a tool dynamically using the session
            print("\n[*] Invoking compute_metrics tool via MCP...")
            result = await session.call_tool(
                name="compute_metrics",
                arguments={"formula": "(15 * 40) / 2"}
            )
            print(f"[Result from MCP Server]: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(run_mcp_client_workflow())
```

---

## 🔨 Hands-On Production Labs

In this module's labs, you will build and integrate custom MCP components:

1. **Developing the SQLite MCP Server:** Build an MCP server that connects to a local SQLite database, exposing secure database query tools and schema reading resources.
2. **Configuring Claude Desktop:** Write a configuration file to integrate your custom SQLite MCP server directly into your local Claude Desktop environment, allowing the Claude application to query your database dynamically.
3. **Multi-Server Orchestrator:** Build a master python client agent that connects to two separate MCP servers concurrently (e.g., a Filesystem server and a DB server) to fetch database rows and write them to a local file.

---

## 📝 MCQ Verification → [mcqs.md](./mcqs.md)
* Challenge your structural understanding of host-client-server pathways, stdio protocols, resources, and custom tool registration with 8 conceptual check questions.

## 💻 Coding Assignment → [assignments.md](./assignments.md)
* **Objective:** Build a robust **Custom Directory Scanner MCP Server**. The server must expose a tool named `scan_directory` that takes a directory path, verifies that the path exists within a permitted workspace (preventing traversal attacks), scans the directory for files, and exposes them as a searchable read-only resource.
