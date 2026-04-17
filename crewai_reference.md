# CrewAI Comprehensive Reference Guide

> Last Updated: April 2026 | Source: Context7 API Documentation

CrewAI is a Python framework for orchestrating autonomous AI agents, offering a lean, fast, and flexible approach to building AI-powered applications with granular control and scalability.

---

## Table of Contents

1. [What CrewAI Can Do](#what-crewai-can-do)
2. [Installation](#installation)
3. [Core Concepts](#core-concepts)
4. [CLI Commands](#cli-commands)
5. [Patterns & Templates](#patterns--templates)
6. [Tools & Extensions](#tools--extensions)
7. [Configuration](#configuration)

---

## What CrewAI Can Do

### Multi-Agent Orchestration
- Create and coordinate multiple autonomous AI agents working together
- Define sequential or hierarchical task execution processes
- Chain crews and flows for complex workflows

### Agent Autonomy
- Assign roles, goals, and backstories to AI agents
- Equip agents with custom tools and capabilities
- Enable structured and unstructured output handling

### Workflow Automation
- Build sequential and parallel task pipelines
- Integrate crews into Flows for multi-stage research/writing
- Use `@start()` and `@listen()` decorators for flow control

### Memory & Context
- Persistent memory across crew interactions
- Configurable LLM providers (OpenAI, Anthropic, Ollama, Gemini, etc.)
- Custom weighting for recency, semantic similarity, and importance

### Tool Integration
- Built-in tools (search, code interpreter, document processing)
- Custom tool creation via `@tool` decorator or BaseTool subclassing
- Integration with external automations via InvokeCrewAIAutomationTool

### Deployment
- Deploy crews to CrewAI Cloud
- Create repository agents for reuse
- CLI-based project management

---

## Installation

### Basic Installation

```bash
pip install crewai
```

### With Tools

```bash
pip install 'crewai[tools]'
```

### Prerequisites

- Python >=3.10 <3.14
- UV dependency manager (recommended)
- LLM API key (OpenAI, Anthropic, etc.) in environment variables

### Create New Project

```bash
# Create a new crew
crewai create crew my_project

# Create a new flow
crewai create flow my_flow_project
```

### Project Structure

```
my_project/
├── pyproject.toml
├── README.md
├── .env
└── src/my_project/
    ├── main.py
    ├── crew.py
    ├── config/
    │   ├── agents.yaml
    │   └── tasks.yaml
    └── tools/
        └── custom_tool.py
```

---

## Core Concepts

### Agent

The fundamental autonomous unit in CrewAI.

```python
from crewai import Agent
from crewai_tools import SerperDevTool

agent = Agent(
    role="Researcher",
    goal="Research the latest AI developments",
    backstory="Expert AI researcher with 10+ years experience",
    tools=[SerperDevTool()],  # Optional tools
    verbose=True,
)
```

**Agent Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `role` | str | The agent's role/function |
| `goal` | str | What the agent aims to achieve |
| `backstory` | str | Persona and context description |
| `tools` | list[Tool] | Tools available to the agent |
| `llm` | str/LLM | Language model configuration |
| `verbose` | bool | Enable verbose logging |

### Task

A unit of work assigned to an agent.

```python
from crewai import Task

task = Task(
    description="Research AI market trends",
    expected_output="Comprehensive market analysis report",
    agent=researcher,
    context=[previous_task],  # Optional: depends on other tasks
)
```

**Task Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `description` | str | Detailed task description |
| `expected_output` | str | What constitutes task completion |
| `agent` | Agent | Agent assigned to this task |
| `context` | list[Task] | Tasks that must complete first |
| `async_execution` | bool | Enable parallel execution |

### Crew

A collection of agents and tasks that work together.

```python
from crewai import Crew, Process

crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[research_task, analysis_task, writing_task],
    process=Process.sequential,  # or Process.hierarchical
    verbose=True,
)

result = crew.kickoff()
```

**Crew Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `agents` | list[Agent] | Agents in the crew |
| `tasks` | list[Task] | Tasks to execute |
| `process` | Process | Execution mode (sequential/hierarchical) |
| `verbose` | bool/int | Logging level |
| `memory` | Memory | Crew memory instance |
| `cache` | bool | Enable caching |

### Flow

A workflow builder using decorators.

```python
from crewai.flow.flow import Flow, listen, start

class MyFlow(Flow):
    @start()
    def begin(self):
        return "initial data"

    @listen(begin)
    def process(self, data):
        return f"processed: {data}"
```

### Process Types

```python
from crewai import Process

# Sequential: tasks execute in order
process=Process.sequential

# Hierarchical: manager agent coordinates
process=Process.hierarchical
```

---

## CLI Commands

### Project Management

| Command | Description |
|---------|-------------|
| `crewai create crew <name>` | Create new crew project |
| `crewai create flow <name>` | Create new flow project |
| `crewai install` | Install dependencies |
| `crewai update` | Update CrewAI version |

### Execution

| Command | Description |
|---------|-------------|
| `crewai run` | Run crew or flow (auto-detect) |
| `crewai flow kickoff` | Run flow (legacy) |

### Training & Testing

| Command | Description |
|---------|-------------|
| `crewai train --n_iterations N --filename file.pkl` | Train the crew |
| `crewai test --n_iterations N` | Test the crew |

### Deployment

| Command | Description |
|---------|-------------|
| `crewai deploy` | Deploy to CrewAI Cloud |

### Direct Execution

```bash
# Via CLI
crewai run

# Via Python
python src/my_project/main.py
```

---

## Patterns & Templates

### Basic Crew Pattern

```python
from crewai import Crew, Agent, Task, Process

# Define agents
researcher = Agent(
    role="Researcher",
    goal="Research market trends",
    backstory="Expert market researcher"
)

writer = Agent(
    role="Writer",
    goal="Compose comprehensive reports",
    backstory="Professional technical writer"
)

# Define tasks
research_task = Task(
    description="Research AI market landscape",
    expected_output="Market data including key players",
    agent=researcher
)

writing_task = Task(
    description="Write report based on research",
    expected_output="Comprehensive report document",
    agent=writer,
    context=[research_task]
)

# Create and run crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential
)

result = crew.kickoff()
```

### Structured Output Pattern

```python
from crewai import Agent
from pydantic import BaseModel

class ResearchFindings(BaseModel):
    main_points: list[str]
    key_technologies: list[str]
    future_predictions: str

researcher = Agent(
    role="AI Researcher",
    goal="Research AI developments",
    backstory="Expert researcher"
)

# With response_format
result = researcher.kickoff(
    "Summarize latest AI developments",
    response_format=ResearchFindings
)

print(result.pydantic.main_points)  # Access structured output
```

### Flow with State Pattern

```python
from crewai.flow.flow import Flow, listen, start
from pydantic import BaseModel

class ResearchState(BaseModel):
    topic: str = ""
    research: str = ""
    report: str = ""

class ResearchFlow(Flow[ResearchState]):
    @start()
    def set_topic(self):
        self.state.topic = "AI Agents"

    @listen(set_topic)
    def do_research(self):
        result = ResearchCrew().crew().kickoff(
            inputs={"topic": self.state.topic}
        )
        self.state.research = result.raw
```

### Async Agent Pattern

```python
from crewai.agent import Agent

analyst = Agent(
    role="Data Analyst",
    goal="Analyze market trends",
    backstory="Expert analyst"
)

# Async execution
result = await analyst.kickoff_async(
    "Analyze current AI market trends",
    response_format=MarketReport,
)
```

### Crew with Memory Pattern

```python
from crewai import Crew, Memory

memory = Memory(
    llm="gpt-4o-mini",
    storage="lancedb",
    recency_weight=0.3,
    semantic_weight=0.5,
    importance_weight=0.2
)

crew = Crew(
    agents=[agent],
    tasks=[task],
    memory=memory
)
```

### Repository Agents Pattern

```python
from crewai import Agent

researcher = Agent(from_repository="market-research-agent")
writer = Agent(from_repository="content-writer-agent")
```

---

## Tools & Extensions

### Built-in Tools

Install with: `pip install 'crewai[tools]'`

Available tools include:

- `PDFSearchTool` - PDF document search
- `SerperDevTool` - Web search
- `CodeInterpreterTool` - Python code execution
- Many more in `crewai_tools` package

### Custom Tool with @tool Decorator

```python
from crewai.tools import tool

@tool("Calculator")
def calculator(expression: str) -> str:
    """Evaluates a mathematical expression."""
    return str(eval(expression))
```

### Custom Tool with BaseTool

```python
from crewai.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

class MyToolInput(BaseModel):
    argument: str = Field(..., description="Description")

class MyCustomTool(BaseTool):
    name: str = "My Custom Tool"
    description: str = "What this tool does"
    args_schema: Type[BaseModel] = MyToolInput

    def _run(self, argument: str) -> str:
        return f"Result: {argument}"
```

### External Automation Tool

```python
from crewai.tools import InvokeCrewAIAutomationTool

data_tool = InvokeCrewAIAutomationTool(
    crew_api_url="https://crew-url.crewai.com",
    crew_bearer_token="your_token",
    crew_name="Data Collection",
    crew_description="Collects data from sources"
)
```

---

## Configuration

### LLM Configuration

```python
from crewai import LLM

# String shorthand
agent = Agent(llm="openai/gpt-4o", ...)

# Full configuration
llm = LLM(
    model="anthropic/claude-sonnet-4-20250514",
    temperature=0.7,
    max_tokens=4000,
)
```

**Supported Providers:**

- `openai/gpt-4o`
- `anthropic/claude-3-haiku-20240307`
- `google/gemini-2.0-flash`
- `ollama/llama3.2`
- `groq/llama-3.3-70b-versatile`
- `bedrock/anthropic.claude-3-sonnet-20240229-v1:0`
- And 20+ more via LiteLLM

### Memory Configuration

```python
from crewai import Memory

# Basic
memory = Memory()

# Custom LLM
memory = Memory(llm="anthropic/claude-3-haiku-20240307")

# Fully offline with Ollama
memory = Memory(
    llm="ollama/llama3.2",
    embedder={"provider": "ollama", "config": {"model_name": "mxbai-embed-large"}}
)
```

### Environment Variables

Create `.env` file:

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-...
```

---

## Quick Reference

### Minimum Example

```python
from crewai import Crew, Agent, Task

agent = Agent(role="Researcher", goal="Research AI", backstory="Expert")
task = Task(description="Research AI trends", expected_output="Report", agent=agent)
crew = Crew(agents=[agent], tasks=[task])
result = crew.kickoff()
```

### Flow Minimum Example

```python
from crewai.flow.flow import Flow, start, listen

class MyFlow(Flow):
    @start()
    def begin(self):
        return "data"

    @listen(begin)
    def process(self, data):
        return f"processed: {data}"

flow = MyFlow()
flow.kickoff()
```

### CLI Quick Commands

```bash
pip install 'crewai[tools]'
crewai create crew my_project
cd my_project
crewai install
crewai run
```

---

## Additional Resources

- Official Docs: https://docs.crewai.com
- GitHub: https://github.com/crewaiinc/crewai
- Context7 Source: https://context7.com (current as of April 2026)