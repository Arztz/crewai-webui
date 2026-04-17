# CrewAI WebUI Builder - Requirements Document

> Created: April 2026 | Version: 1.0

## Vision

A visual no-code web interface for building CrewAI workflows where every component (Tool, Agent, Task, Crew) is a **separate, reusable block** stored in a database. Components can be composed together dynamically. Includes an **event-driven trigger system** where 10-30+ events can wait in a loop and kickoff crews when conditions are met.

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Core Architecture](#core-architecture)
3. [Entity Designs](#entity-designs)
4. [Trigger/Event System](#triggerevent-system)
5. [UI/UX Requirements](#uiux-requirements)
6. [API Requirements](#api-requirements)
7. [Data Models](#data-models)
8. [Workflows](#workflows)
9. [Comparison with Existing Solutions](#comparison-with-existing-solutions)
10. [Future Enhancements](#future-enhancements)

---

## Problem Statement

### Current Pain Points

1. **Monolithic Code** - CrewAI code is written as blocks that need to be separated:
   ```python
   # Current way: everything in one file
   tool = @tool(...)
   agent = Agent(tools=[tool])
   task = Task(agent=agent)
   crew = Crew(tasks=[task])
   ```

2. **No Reusability** - If you need the same agent in multiple crews, you copy-paste code

3. **No Visual Builder** - Developers must write code to compose crews

4. **Limited Triggers** - Only cron scheduling, no complex event-driven patterns

5. **No Database Persistence** - Crews exist only as code, not queryable

---

## Core Architecture

### Component Separation Model

```
┌─────────────────────────────────────────────────────────────┐
│                      DATABASE                              │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│   TOOLS    │   AGENTS   │   TASKS    │      CREWS         │
│  (table)  │  (table)  │  (table)  │      (table)       │
├─────────────┼─────────────┼─────────────┼─────────────────┤
│  tool_id   │  agent_id  │  task_id   │      crew_id      │
│  name      │  role      │  description│      name         │
│  code      │  goal      │  expected_ou│      process     │
│  schema    │  backstory │  agent_id   │      agents[]     │
│  type      │  tools[]   │  tools[]   │      tasks[]       │
│            │  llm_config│  context[] │      memory       │
│            │            │            │      triggers[]   │
└─────────────┴─────────────┴─────────────┴─────────────────────┘
```

### Composition Flow

```
┌──────────────────────────────────────────────────────────────────┐
│              FLEXIBLE CREATION (Any Order)                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   STEP 1: Create any entity first                               │
│   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │
│   │ + Create   │ │ + Create   │ │ + Create   │              │
│   │   TOOL     │ │   AGENT    │ │   TASK     │              │
│   └─────────────┘ └─────────────┘ └─────────────┘              │
│         │               │               │                      │
│         ▼               ▼               ▼                      │
│   ┌─────────────────────────────────────────────┐              │
│   │  Database: Empty references (null)           │              │
│   │  - tool.tools_ids = [] (empty)              │              │
│   │  - task.agent_id = null (not assigned)      │              │
│   └─────────────────────────────────────────────┘              │
│                                                                  │
│   STEP 2: Link/Compose later (any time)                         │
│                                                                  │
│   Scenario A: Agent first → Tools later                        │
│   ┌─────────────┐      ┌─────────────┐                         │
│   │ Agent       │◄────►│   Tools     │                         │
│   │ (existing)  │ link │  (new)      │                         │
│   └─────────────┘      └─────────────┘                         │
│                                                                  │
│   Scenario B: Task first → Agent later                          │
│   ┌─────────────┐      ┌─────────────┐                         │
│   │ Task        │◄────►│   Agent     │                         │
│   │ (existing)  │ link │  (new)      │                         │
│   └─────────────┘      └─────────────┘                         │
│                                                                  │
│   STEP 3: Assemble in Crew (final step)                        │
│   ┌────────────────────────────────────────────────────────┐   │
│   │  CREW                                                    │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐             │   │
│   │  │ Agent 1  │  │ Agent 2  │  │ Agent 3  │             │   │
│   │  │ (linked) │  │ (linked) │  │ (linked) │             │   │
│   │  └────┬─────┘  └────┬─────┘  └────┬─────┘             │   │
│   │       │             │             │                     │   │
│   │       ▼             ▼             ▼                     │   │
│   │  ┌─────────┐   ┌─────────┐   ┌─────────┐              │   │
│   │  │ Task 1  │   │ Task 2  │   │ Task 3  │              │   │
│   │  └─────────┘   └─────────┘   └─────────┘              │   │
│   └────────────────────────────────────────────────────────┘   │
│                                                                  │
│   KEY: Creation order is flexible - link anytime!              │
└──────────────────────────────────────────────────────────────────┘
```

### Flexible Workflow Examples

#### Example 1: Agent First, Then Tools

```
1. Go to Agents → Click [+ New Agent]
   - Name: "Researcher"
   - Role: "Research Analyst"
   - Goal: "Research market trends"
   - Backstory: "Expert in market analysis"
   - Save → Agent created (tools = [])
   
2. Go to Tools → Click [+ New Tool]
   - Name: "Web Search"
   - Description: "Search the web"
   - Code: (write tool code)
   - Save → Tool created
   
3. Return to Agents → Open "Researcher"
   - In "Tools" section, click [Add Tool]
   - Select "Web Search" from library
   - Save → Agent now has tool linked

Result: Agent created before tools even existed!
```

#### Example 2: Task First, Then Agent

```
1. Go to Tasks → Click [+ New Task]
   - Title: "Research Competitors"
   - Description: "Find top 5 competitors"
   - Expected Output: "Competitor list with analysis"
   - Agent: (leave as "Not Assigned")
   - Save → Task created (agent_id = null)

2. Go to Agents → Create Agent
   - Name: "Business Analyst"
   - Role: "Competitor Analysis Expert"
   
3. Return to Tasks → Open "Research Competitors"
   - In "Agent" dropdown, select "Business Analyst"
   - Save → Task now linked to agent

Result: Task existed before agent was created!
```

#### Example 3: Crew Assembly (Final Step)

```
1. Go to Crews → Click [+ New Crew]
   - Name: "Market Analysis Crew"
   - Process: (select later)

2. In Crew canvas:
   - Drag "Researcher" from agent library
   - Drag "Business Analyst" from agent library
   - Drag "Report Writer" from agent library
   - Drag tasks and connect to agents
   
3. Set Process Mode:
   - Toggle between Sequential / Hierarchical
   - Visual representation changes

4. Save → Crew assembled from pre-existing components
```

#### Key Principle

> **Create first, link later** - Any entity can be created independently.
> References are optional at creation time, filled in anytime after.

---

## Entity Designs

### 1. Tool Entity

**Purpose**: Reusable function that agents can use

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Tool name (unique) |
| `description` | string | What the tool does |
| `type` | enum | `decorator` / `basetool` / `mcp` / `http` |
| `code` | text | Python code for the tool |
| `input_schema` | JSON | Pydantic input schema |
| `output_schema` | JSON | Pydantic output schema |
| `is_active` | boolean | Enable/disable |
| `version` | integer | Version number |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

**UI Operations**:
- Create tool via code editor
- Create tool via form builder
- Import from crewai_tools library
- Test tool in sandbox

### 2. Agent Entity

**Purpose**: Autonomous worker with role, goal, backstory, and tools

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Agent name |
| `role` | string | Agent's role description |
| `goal` | string | Agent's objective |
| `backstory` | text | Persona context |
| `llm_provider` | enum | `openai` / `anthropic` / `ollama` / etc |
| `llm_model` | string | Model name (e.g., gpt-4o) |
| `llm_config` | JSON | temperature, max_tokens, etc |
| `tools` | UUID[] | References to Tool entities |
| `verbose` | boolean | Enable verbose logging |
| ` Allow_delegation` | boolean | Can delegate tasks |
| `max_iterations` | integer | Max execution loops |
| `is_template` | boolean | Template for cloning |
| `version` | integer | Version number |

**UI Operations**:
- Create via form
- Clone existing agent
- Import from repository
- Connect tools (drag-drop)
- Test agent standalone

### 3. Task Entity

**Purpose**: Work unit assigned to an agent

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `title` | string | Task title |
| `description` | text | Detailed instructions |
| `expected_output` | text | What constitutes completion |
| `agent_id` | UUID | Reference to Agent entity |
| `tools` | UUID[] | Additional tools for this task |
| `context_tasks` | UUID[] | Tasks that must complete first |
| `async_execution` | boolean | Run in parallel |
| `output_format` | enum | `text` / `json` / `pydantic` |
| `pydantic_model` | JSON | Pydantic model for output |
| `priority` | enum | `low` / `normal` / `high` |
| `timeout_seconds` | integer | Max execution time |
| `version` | integer | Version number |

**UI Operations**:
- Create via form
- Connect agent (dropdown/search)
- Connect context tasks (visual line)
- Define output format
- Preview description

### 4. Crew Entity

**Purpose**: Collection of agents and tasks orchestrated together

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Crew name |
| `description` | text | Crew purpose |
| `process` | enum | `sequential` / `hierarchical` |
| `verbose` | integer | Log level (0-3) |
| `agents` | UUID[] | References to Agent entities |
| `tasks` | UUID[] | References to Task entities |
| `manager_agent_id` | UUID | Manager for hierarchical |
| `memory` | boolean | Enable crew memory |
| `memory_config` | JSON | Memory settings |
| `cache` | boolean | Enable caching |
| `triggers` | UUID[] | Triggers that activate this crew |
| `output_format` | enum | How to return results |
| `can_trigger_other_crews` | boolean | Enable crew-to-crew chaining |
| `trigger_chain_on` | enum | `completed` / `failed` / `always` |
| `output_fields` | JSON | Define output fields for chaining |
| `is_active` | boolean | Enable/disable |
| `version` | integer | Version number |

**UI Operations**:
- Compose via drag-drop
- Connect agents/tasks visually
- Set execution process
- Connect triggers
- Run manually
- View execution history

---

## Trigger/Event System

### The Core Innovation

This is the key differentiator from CrewAI Studio and other builders. A system where **10-30+ events can wait in a loop**, listening for conditions, then kickoff crews.

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    EVENT LISTENER DAEMON                      │
│  (Background service running 24/7)                       │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│  │ TRIGGER 1  │ │ TRIGGER 2  │ │ TRIGGER N  │        │
│  │ (waiting)   │ │ (waiting)   │ │ (waiting)  │        │
│  │            │ │            │ │            │        │
│  │ condition: │ │ condition: │ │ condition:│        │
│  │ event=x   │ │ event=y    │ │ event=z   │        │
│  │ where x>y │ │ where...   │ │ where...  │        │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘        │
│        │             │             │                │
│        └─────────────┼─────────────┘                │
│                      ▼                             │
│              ┌─────────────────┐                   │
│              │ EVENT DISPATCHER │                   │
│              └────────┬────────┘                   │
│                       │                              │
│         ┌─────────────┼─────────────┐              │
│         ▼             ▼             ▼              │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│   │  CREW 1   │ │  CREW 2  │ │  CREW 3  │         │
│   │ kickoff() │ │ kickoff()│ │ kickoff()│         │
│   └──────────┘ └──────────┘ └──────────┘         │
└─────────────────────────────────────────────────────┘
```

### Trigger Entity

**Purpose**: Monitor events and kickoff crews when conditions are met

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Trigger name |
| `description` | text | What this trigger does |
| `type` | enum | `event` / `schedule` / `webhook` / `manual` / `crew_completion` |
| `event_source` | enum | `database` / `api` / `queue` / `webhook` / `cron` / `crew` |
| `event_type` | string | Event to listen for |
| `filter_condition` | JSON | JSON condition for filtering |
| `source_crew_id` | UUID | Source crew that triggers this (when type=crew_completion) |
| `crew_id` | UUID | Crew to kickoff |
| `input_mapping` | JSON | Map event fields to crew input |
| `output_mapping` | JSON | Map output to next crew input |
| `continue_on_failure` | boolean | Continue chain even if crew fails |
| `priority` | integer | Execution priority |
| `enabled` | boolean | Enable/disable |
| `max_concurrent` | integer | Max parallel executions |
| `cooldown_seconds` | integer | Min time between triggers |
| `last_triggered_at` | timestamp | Last execution time |
| `trigger_count` | integer | Total times triggered |

### Event Source Types

#### 1. Database Events
```python
# Example: Monitor database table for new rows
trigger = Trigger(
    type="event",
    event_source="database",
    event_type="new_row",
    filter_condition={
        "table": "leads",
        "where": "status = 'new' AND assigned_to IS NULL"
    }
)
```

#### 2. Queue Events (Redis/RabbitMQ)
```python
# Example: Listen to message queue
trigger = Trigger(
    type="event",
    event_source="queue",
    event_type="message",
    filter_condition={
        "queue": "orders",
        "routing_key": "order.created"
    }
)
```

#### 3. Webhook Events
```json
{
    "type": "webhook",
    "event_source": "webhook",
    "endpoint": "/webhooks/stripe",
    "event_type": "payment.succeeded",
    "filter_condition": {
        "amount_gte": 1000
    }
}
```

#### 4. Schedule Events (Cron)
```json
{
    "type": "schedule",
    "event_source": "cron",
    "cron_expression": "0 9 * * 1-5",
    "timezone": "UTC"
}
```

#### 5. API Events
```json
{
    "type": "api",
    "event_source": "api",
    "endpoint": "/api/v1/trigger",
    "auth_required": true
}
```

#### 6. Crew Completion Events (Crew-to-Crew Chain)
This is the key feature - **once a crew finishes, it can trigger another crew**.
```python
# Example: Crew1 finishes → triggers Crew2
trigger = Trigger(
    type="crew_completion",
    event_source="crew",
    source_crew_id="crew-1-uuid",
    event_type="completed",  # or "failed"
    crew_id="crew-2-uuid",  # Crew to trigger next
    output_mapping={
        "research_results": "{{crew1.output}}",
        "summary": "{{crew1.tasks[0].output}}"
    }
)
```

**Use Cases:**
- **Sequential Crews**: Crew1 (sequential) → Crew2 (hierarchical)
- **Pipeline**: Research → Analysis → Report (each crew triggers next)
- **Conditional**: If Crew1 succeeds → Crew2, if fails → Crew3
- **Fan-out**: Crew1 triggers multiple crews in parallel

**Supported Chain Combinations:**
| Source Crew | → | Triggered Crew |
|-------------|---|---------------|
| Sequential | → | Sequential |
| Sequential | → | Hierarchical |
| Hierarchical | → | Sequential |
| Hierarchical | → | Hierarchical |

### Crew-to-Crew Chain Visual

In the canvas, crew chains look like:

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  CREW CHAIN EDITOR                                                    │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│   [CREW 1: Research ] ────⚡completed────────► [CREW 2: Analysis]   │
│        ▼                                                     ▼        │
│   ┌─────────────────┐                           ┌─────────────┐ │
│   │ Process: seq ▼  │                           │ Process:    │ │
│   │ [1] Research    │                           │   hierar-    │ │
│   │ [2] Summarize   │                           │   chical ▼   │ │
│   └─────────────────┘                           └──────┬──────┘ │
│                                              │        │
│                                              ▼        ▼        │
│                                        ┌─────────┐  ┌──────┐│
│                                        │ Crew 3A │  │Crew3B││  (parallel)
│                                        │ReportWr│  │Notifr││
│                                        └────────┘  └──────┘│
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

**Chain Line Styling:**
| Event Type | Line Style | Color |
|------------|-----------|-------|
| `completed` | Solid → arrow | Green |
| `failed` | Dashed → arrow | Red |
| `started` | Dotted → arrow | Blue |

**Chain Modal:**
```
┌─────────────────────────────────────┐
│  CHAIN CONFIGURATION                 │
├─────────────────────────────────────┤
│                                     │
│  When [CREW 1: Research]           │
│  ┌──────────────────────────────┐  │
│  │ ✓ completes normally          │  │
│  │ ○ fails                   │  │
│  │ ○ starts                 │  │
│  └──────────────────────────────┘  │
│                                     │
│  Then trigger [CREW 2: Analysis]  │
│                                     │
│  ─────────────────────────────────   │
│                                     │
│  Map output:                        │
│  ├─ research_results → input.topic │
│  ├─ summary → input.summary        │
│  └─ new field → input.custom      │
│                                     │
│  Options:                          │
│  ├─ ☐ Continue if source fails   │
│  ├─ ☐ Wait for completion       │
│  └─ ☐ Pass output as-is        │
│                                     │
│  [Cancel]  [Save Chain]           │
└─────────────────────────────────────┘
```

### Event Condition Builder

UI for building conditions without code:

```
┌────────────────────────────────────────────────────────┐
│  CONDITION BUILDER                                     │
├────────────────────────────────────────────────────────┤
│                                                        │
│  WHEN [event.field] [operator] [value]                 │
│                                                        │
│  Field:    [dropdown ▼] event.amount                   │
│  Operator: [dropdown ▼] >                             │
│  Value:    [input _____] 1000                         │
│                                                        │
│  ────���────────────────────────────────                │
│                                                        │
│  AND/OR [add condition ▼]                             │
│                                                        │
│  [+ Add Group] [+ Add Condition]                      │
│                                                        │
│  PREVIEW:                                              │
│  event.amount > 1000 AND event.currency == "USD"    │
└────────────────────────────────────────────────────────┘
```

### Event Queue Management

UI to see all waiting triggers:

```
┌────────────────────────────────────────────────────────────────────┐
│  TRIGGER DASHBOARD                                         │
├──────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌────────────────┐ ┌────────────────┐ ┌───────────────┐  │
│  │ ACTIVE (12)    │ │ WAITING (18)   │ │  FAILED (2)   │  │
│  └────────────────┘ └────────────────┘ └───────────────┘  │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ WAITING TRIGGERS                                      │   │
│  ├───────────────┬──────────────┬─────────┬───────────┤   │
│  │ Name          │ Event Source │ Crew    │ Status    │   │
│  ├───────────────┼──────────────┼─────────┼───────────┤   │
│  │ New Lead      │ database     │ Sales..│ waiting  │   │
│  │ Payment OK   │ webhook      │ Billi..│ waiting  │   │
│  │ Daily Report │ cron         │ Repo...│ waiting  │   │
│  │ Order New    │ queue       │ Fullf..│ waiting  │   │
│  │ ...         │ ...         │ ...    │ waiting  │   │
│  └───────────────┴──────────────┴─────────┴───────────┘   │
│                                                            │
│  [+ Add Trigger]  [↻ Refresh]   [▶ Enable All]            │
└────────────────────────────────────────────────────────────┘
```

---

## UI/UX Requirements

### Design Philosophy

> **No forms, no code visible** - Pure visual mindmap/Miro-style experience.
> Everything represented as visual nodes with connections.

### Page Structure

1. **Dashboard** - Overview, quick stats, recent runs
2. **Tools** - Tool library (visual cards on canvas)
3. **Agents** - Agent library (visual cards on canvas)
4. **Tasks** - Task library (visual cards on canvas)
5. **Crews** - Crew composition (main mindmap workspace)
6. **Triggers** - Event/trigger management (flow view)
7. **Executions** - Run history and logs
8. **Settings** - System configuration

---

### Miro/Mindmap-Style UI Specification

#### Main Canvas View (No Code Visible)

```
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│  🧠 CREW EDITOR - Market Analysis Crew                         [+ Add] [↻] [💾] [📤 Export] │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                       │
│                    ┌─────────────────┐                                                  │
│                    │   MANAGER AGENT  │                                                  │
│                    │   ┌─────────┐   │                                                  │
│                    │   │ Coord-  │   │                                                  │
│                    │   │ inator │   │                                                  │
│                    │   └─────────┘   │                                                  │
│                    └────────┬────────┘                                                  │
│                             │                                                           │
│           ┌─────────────────┼─────────────────┐                                              │
│           │                 │                 │                                              │
│           ▼                 ▼                 ▼                                              │
│    ┌────────────┐   ┌────────────┐   ┌────────────┐                                   │
│    │  AGENT   │   │  AGENT   │   │  AGENT    │                                   │
│    │Researcher │   │ Analyst  │   │  Writer   │                                   │
│    │  🔍     │   │  📊     │   │  ✍️      │                                   │
│    └────┬─────┘   └────┬─────┘   └────┬─────┘                                   │
│         │               │               │                                                │
│         ▼               ▼               ▼                                                │
│    ┌─────────┐   ┌─────────┐   ┌─────────┐                                           │
│    │  TASK  │   │  TASK  │   │  TASK   │                                           │
│    │Research │   │Analyze │   │ Write  │                                           │
│    │ Market │   │  Data  │   │ Report │                                           │
│    └─────────┘   └─────────┘   └─────────┘                                           │
│                                                                                       │
│  ────│───────────────────────────────────────────────────────────────────────────│───────     │
│       │ PROCESS MODE:  ▶ Sequential    ○ Hierarchical                    [▶ Run] │     │
│       └───────────────────────────────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

#### Process Mode Toggle (Critical!)

Located at bottom of canvas, always visible:

```
┌─────────────────────────────────────────────┐
│  EXECUTION MODE                            │
├─────────────────────────────────────────────┤
│                                          │
│  ┌──────────────────┐  ┌─────────────┐  │
│  │  ▶ Sequential  │  │ Hierarchical │  │
│  ├────────────────┤  ├─────────────┤  │
│  │ Tasks run in   │  │ Manager    │  │
│  │ order        │  │ coordinates │  │
│  │ A → B → C   │  │ A, B, C   │  │
│  │              │  │            │  │
│  │ ● Selected  │  │ ○ Off      │  │
│  └──────────────┘  └─────���─��─────┘  │
└─────────────────────────────────────────────┘
```

**Sequential Mode Visual:**
```
  [Task 1] → [Task 2] → [Task 3]
     │           │           │
     ▼           ▼           ▼
   Agent A    Agent B    Agent C
```

**Hierarchical Mode Visual:**
```
        ┌──────────┐
        │ Manager │
        │ Agent  │
        └────┬───┘
     ┌──────┼──────┬──────┐
     ▼     ▼     ▼
  Agent  Agent  Agent
   A      B      C
  Task   Task   Task
```

### Execution Mode = UI Behavior

The selected execution mode **directly affects drag-drop and line drawing behavior**:

#### Sequential Mode

| UI Element | Behavior |
|-----------|---------|
| **Drag Task** | Auto-order based on drop position (left-to-right = execution order) |
| **Connecting Line** | Solid arrow → shows dependency direction |
| **Drop Zone** | Highlights where task will insert in sequence |
| **Task Order** | User explicitly controls order via drag position |
| **Visual Constraint** | Can reorder tasks by dragging |
| **Auto-numbering** | Tasks show execution order: [1], [2], [3] |

**Canvas Example (Sequential):**
```
  [1] ───────→ [2] ───────→ [3]
  Research     Analyze      Write
     │            │           │
     ▼            ▼           ▼
  Researcher  Analyst     Writer
```

- Lines are **solid**
- Order visible via **numbers**
- Can **drag to reorder** tasks in sequence

#### Hierarchical Mode

| UI Element | Behavior |
|-----------|---------|
| **Manager Node** | Required - shown at top, special highlight |
| **Connect to Manager** | Tasks connect TO manager, not to each other |
| **Sub-agents** | Auto-connected to manager (no manual linking needed) |
| **Drop Zone** | Only drops as sub-agent under manager |
| **Visual Constraint** | Cannot reorder (manager handles) |
| **Parallel Indicators** | Tasks under same agent can run parallel |

**Canvas Example (Hierarchical):**
```
        ┌─────────────┐
        │ ★ Manager  │ ← Special star icon, purple border
        │  Coordinate│
        └─────┬─────┘
     ┌────────┼────────┬────────┐
     │        │        │        │
     ▼        ▼        ▼
  Agent    Agent    Agent
  Resrch   Analyst  Writer   ← All linked to manager
     │        │        │
     ▼        ▼        ▼
  Task     Task     Task
```

- **Manager node** has special styling (purple, star)
- **No direct lines** between tasks
- **Manager controls** delegation automatically
- **Adding new agent**: automatically slots under manager

#### Mode Switching

```
┌─────────────────────────────────────────────────────────────┐
│  SWITCHING MODES                                        │
├────────────────��────────────────────────────────────────────┤
│                                                       │
│  Sequential → Hierarchical:                           │
│  - Prompts: "Select Manager Agent"                     │
│  - Converts task order links to sub-agent tree        │
│  - Existing connections preserved                    │
│                                                       │
│  Hierarchical → Sequential:                          │
│  - Prompts: "Confirm task order"                   │
│  - Converts tree structure to ordered list          │
│  - Tasks auto-numbered [1, 2, 3]                │
│                                                       │
│  ⚠️  May lose some flexibility when converting     │
└─────────────────────────────────────────────────────────────┘
```

#### Visual Differences Summary

| Aspect | Sequential | Hierarchical |
|--------|------------|---------------|
| **Lines** | Solid → arrows | Lines from manager |
| **Numbering** | [1] [2] [3] shown | No numbers (manager-controlled) |
| **Task Position** | User controls order | Manager controls parallel/sequence |
| **Manager Node** | Optional/ignored | Required, special styling |
| **Drag Constraint** | Reorder by position | Drop under manager only |

---

#### Visual Nodes (No Code Forms)

**Tool Node:**
```
┌──────────────────┐
│    🔧 TOOL      │
│  Web Search     │
├──────────────────┤
│  🔗 Linked to:   │
│  • Research Agent│
│  • Data Agent   │
└──────────────────┘
```

**Agent Node:**
```
┌──────────────────┐
│    👤 AGENT    │
│  Researcher    │
├──────────────────┤
│  Role: Market  │
│       Researcher│
│  ──────────────│
│  Tools: 3     │
│  [🔧🔍📄]    │
│  ──────────────│
│  LLM: GPT-4o  │
└──────────────────┘
        │
        ▼ (drag to connect)
```

**Task Node:**
```
┌──────────────────┐
│    📋 TASK     │
│  Research      │
│  Market       │
├──────────────────┤
│  → Assigned to: │
│    Researcher │
│  ────────────│
│  Expected:    │
│  Market Report│
└──────────────────┘
```

**Crew Node:**
```
┌────────────────────┐
│   🚀 CREW       │
│  Market Team    │
├──────────────────┤
│  Process: seq │
│  ────────────│
│  Agents: 3   │
│  Tasks: 3    │
│  ────────────│
│  Triggers: 2  │
│  [⚡📅🔗]   │
└────────────────────┘
         │
         ▼ (can fork to another crew)
```

---

#### Connection Interactions

```
┌─────────────────────────────────────────────────────────────────┐
│  HOW TO CONNECT (Drag & Drop)                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Tool → Agent:                                                   │
│  1. Click tool node                                             │
│  2. Drag from 🔗 port                                         │
│  3. Drop on agent node                                          │
│  4. Line appears:  🔧───────→👤                               │
│                                                                  │
│  Agent → Task:                                                  │
│  1. Click agent node                                           │
│  2. Drag from 📋 port                                          │
│  3. Drop on task node                                           │
│  4. Line appears:  👤���──────→📋                              │
│                                                                  │
│  Task ↔ Task (context):                                          │
│  1. Click first task                                            │
│  2. Drag from → port                                           │
│  3. Drop on second task                                         │
│  4. Line appears:  📋───────→📋 (dashed)                   │
│                                                                  │
│  Crew → Trigger:                                               │
│  1. Click crew node                                            │
│  2. Drag from ⚡ port                                          │
│  3. Drop on trigger hub                                         │
│  4. Line appears:  🚀───────⚡                              │
└─────────────────────────────────────────────────────────────────┘
```

---

#### Sidebar Library (Collapsible)

```
┌─────────────────────────────────────────┐
│  📚 LIBRARY              [≡] Collapse │
├─────────────────────────────────────────┤
│                                          │
│  🔧 TOOLS (12)                        │
│  ├─ Web Search                      │
│  ├─ PDF Reader                    │
│  ├─ Calculator                  │
│  └─ ...                          │
│                                          │
│  👤 AGENTS (8)                         │
│  ├─ Researcher                  │
│  ├─ Analyst                   │
│  ├─ Writer                   │
│  └─ ...                         │
│                                          │
│  📋 TASKS (15)                         │
│  ├─ Research Market            │
│  ├─ Analyze Data            │
│  ├─ Write Report           │
│  └─ ...                         │
│                                          │
│  🚀 CREWS (5)                          │
│  ├─ Market Analysis         │
│  ├─ Content Pipeline     │
│  └─ ...                         │
│                                          │
│  ⚡ TRIGGERS (3)                        │
│  ├─ New Lead               │
│  ├─ Daily Report          │
│  └─ ...                          │
└─────────────────────────────────────────┘
```

**How to add from library:**
1. Find item in sidebar
2. Drag onto canvas
3. Drop anywhere
4. Node created from template

---

#### Node Context Menu (Right-click)

```
┌─────────────────────────────────────────┐
│  Right-click on AGENT node               │
├─────────────────────────────────────────┤
│                                         │
│  ✏️ Edit Details                      │
│  📋 Create Task (linked)               │
│  🔗 Add Tool                        │
│  📋 Duplicate Agent                │
│  📤 Export as Template             │
│  ─────────────────                  │
│  🗑️ Delete                        │
└─────────────────────────────────────────┘
```

---

#### Floating Toolbar

```
┌─────────────────────────────────────────┐
│  [+ Node ▼]  [🤝 Connect]  [↩ Undo]    │
│             [↪ Redo]  [🔍 Zoom]       │
├─────────────────────────────────────────┤
│                                          │
│  [+ Node ▼] shows:                      │
│  ├─ + Tool                           │
│  ├─ + Agent                         │
│  ├─ + Task                         │
│  ├─ + Crew                        │
│  └─ + Trigger                     │
└─────────────────────────────────────────┘
```

---

#### No Code Visible - Even for Tools

```
┌─────────────────────────────────────────────┐
│  TOOL EDITOR (Modal)                       │
├─────────────────────────────────────────────┤
│                                            │
│  Name:  [________________]                   │
│                                            │
│  Description:                             │
│  [________________________________]       │
│                                            │
│  ┌─────────────────────────────────────┐    │
│  │  VISUAL BUILDER                     │    │
│  │                                     │    │
│  │  Input:                            │    │
│  │  [query: String ________]         │    │
│  │             [+ Add Input]         │    │
│  │                                     │    │
│  │  Output:                           │    │
│  │  [results: List ________]       ���    │
│  │                                     │    │
│  │  ┌────────────────────────┐   │    │
│  │  │ ██ CODE PREVIEW  │   │    │
│  │  │ (auto-generated) │   │    │
│  │  └────────────────────────┘   │    │
│  └─────────────────────────────────────┘     │
│                                            │
│  [Cancel]  [Test]  [Save]                 │
└─────────────────────────────────────────────┘
```

---

#### Required Views

1. **Canvas View** - Mindmap/miro-style visual editing (MAIN)
2. **List View** - Compact table (for searching)
3. **Detail Modal** - Quick edit (not full-page form)
4. **Test View** - Sandbox execution
4. **Test View** - Sandbox execution
5. **History View** - Execution logs

---

## API Requirements

### Entity APIs

```
# Tools
POST   /api/tools              - Create tool
GET    /api/tools             - List tools
GET    /api/tools/:id         - Get tool
PUT    /api/tools/:id         - Update tool
DELETE /api/tools/:id        - Delete tool
POST   /api/tools/:id/test   - Test tool

# Agents
POST   /api/agents            - Create agent
GET    /api/agents           - List agents
GET    /api/agents/:id        - Get agent
PUT    /api/agents/:id       - Update agent
DELETE /api/agents/:id      - Delete agent
POST   /api/agents/:id/tools - Add tool to agent

# Tasks
POST   /api/tasks             - Create task
GET    /api/tasks            - List tasks
GET    /api/tasks/:id        - Get task
PUT    /api/tasks/:id        - Update task
DELETE /api/tasks/:id       - Delete task
POST   /api/tasks/:id/agent - Assign agent

# Crews
POST   /api/crews            - Create crew
GET    /api/crews            - List crews
GET    /api/crews/:id        - Get crew
PUT    /api/crews/:id       - Update crew
DELETE /api/crews/:id       - Delete crew
POST   /api/crews/:id/run   - Run crew
POST   /api/crews/:id/kickoff - Alias for run

# Triggers
POST   /api/triggers        - Create trigger
GET    /api/triggers        - List triggers
GET    /api/triggers/:id   - Get trigger
PUT    /api/triggers/:id    - Update trigger
DELETE /api/triggers/:id   - Delete trigger
POST   /api/triggers/:id/enable  - Enable
POST   /api/triggers/:id/disable - Disable

# Executions
GET    /api/executions       - List executions
GET    /api/executions/:id  - Get execution details
```

### Webhook Endpoints

```
POST /webhooks/:trigger_name  - External trigger
POST /webhooks/generic        - Generic webhook
```

---

## Data Models

### Database Schema (PostgreSQL)

```sql
-- Tools table
CREATE TABLE tools (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL,
    code TEXT,
    input_schema JSONB,
    output_schema JSONB,
    is_active BOOLEAN DEFAULT true,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Agents table
CREATE TABLE agents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    role TEXT NOT NULL,
    goal TEXT NOT NULL,
    backstory TEXT,
    llm_provider VARCHAR(50),
    llm_model VARCHAR(100),
    llm_config JSONB,
    tools UUID[],
    verbose BOOLEAN DEFAULT false,
    allow_delegation BOOLEAN DEFAULT false,
    max_iterations INTEGER DEFAULT 10,
    is_template BOOLEAN DEFAULT false,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Tasks table
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    expected_output TEXT,
    agent_id UUID REFERENCES agents(id),
    tools UUID[],
    context_tasks UUID[],
    async_execution BOOLEAN DEFAULT false,
    output_format VARCHAR(50) DEFAULT 'text',
    pydantic_model JSONB,
    priority VARCHAR(20) DEFAULT 'normal',
    timeout_seconds INTEGER DEFAULT 300,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Crews table
CREATE TABLE crews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    process VARCHAR(50) DEFAULT 'sequential',
    verbose INTEGER DEFAULT 1,
    agents UUID[],
    tasks UUID[],
    manager_agent_id UUID REFERENCES agents(id),
    memory BOOLEAN DEFAULT false,
    memory_config JSONB,
    cache BOOLEAN DEFAULT true,
    triggers UUID[],
    output_format VARCHAR(50) DEFAULT 'text',
    is_active BOOLEAN DEFAULT true,
    version INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Triggers table
CREATE TABLE triggers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL,
    event_source VARCHAR(50) NOT NULL,
    event_type VARCHAR(100),
    filter_condition JSONB,
    crew_id UUID REFERENCES crews(id),
    input_mapping JSONB,
    priority INTEGER DEFAULT 0,
    enabled BOOLEAN DEFAULT true,
    max_concurrent INTEGER DEFAULT 1,
    cooldown_seconds INTEGER DEFAULT 60,
    last_triggered_at TIMESTAMP,
    trigger_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Executions table
CREATE TABLE executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    crew_id UUID REFERENCES crews(id),
    trigger_id UUID REFERENCES triggers(id),
    status VARCHAR(50) NOT NULL,
    input_data JSONB,
    output_data JSONB,
    error_message TEXT,
    started_at TIMESTAMP NOT NULL,
    completed_at TIMESTAMP,
    duration_seconds INTEGER,
    token_usage JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Workflows

### Workflow 1: Create and Connect Tool → Agent → Task → Crew

```
1. Go to Tools page
2. Click [+ New Tool]
3. Enter name, description
4. Write code or use form builder
5. Save → Tool saved to DB

6. Go to Agents page
7. Click [+ New Agent]
8. Enter role, goal, backstory
9. Select LLM provider/model
10. Drag tool from library
11. Save → Agent saved with tool reference

12. Go to Tasks page
13. Click [+ New Task]
14. Enter description, expected output
15. Select agent from dropdown
16. Save → Task saved with agent reference

17. Go to Crews page (canvas)
18. Drag agent from library
19. Drag task from library
20. Connect task to agent (visual line)
21. Set process (sequential/hierarchical)
22. Save crew
23. Click Run → Crew executes
```

### Workflow 2: Event-Driven Automation

```
1. Create crew (as above)
2. Go to Triggers page
3. Click [+ New Trigger]
4. Enter name, description
5. Select trigger type: Event
6. Select event source: database
7. Set filter: table=leads, status=new
8. Select crew to kickoff
9. Map event fields to crew input
10. Save trigger

11. Trigger now waits in event loop
12. When new lead inserted:
    - Event detected
    - Condition evaluated
    - Crew kicked off
    - Execution logged

13. Go to Executions page
14. View execution history
15. See results, logs, errors
```

### Workflow 3: Reuse Components

```
Scenario: Need same researcher in multiple crews

1. Go to Agents page
2. Find existing "Market Researcher" agent
3. Click [Duplicate]
4. Rename to "Competitor Research Specialist"
5. Modify goal: "Research competitors"
6. Save as new agent

7. Go to Crews page
8. Create new crew
9. Drag existing tasks OR
10. Create new tasks using new agent

Result: Two crews share base agent definition,
         but have different configurations
```

---

## Comparison with Existing Solutions

| Feature | CrewAI Studio | Langflow | **This Project** |
|---------|---------------|----------|------------------|
| Visual editor | ✅ | ✅ | ✅ |
| Drag-drop composition | ✅ | ✅ | ✅ |
| Database persistence | ❌ | ❌ | ✅ |
| Reusable components | Limited | Limited | **Full** |
| Event-driven triggers | Basic | ❌ | **Advanced** |
| 10+ concurrent triggers | ❌ | ❌ | ✅ |
| Condition builder UI | ✅ | ✅ | ✅ |
| Export to Python | ✅ | ✅ | ✅ |
| Self-hosted | ❌ | ✅ | ✅ |
| Trigger dashboard | ❌ | ❌ | ✅ |

### Key Differentiators

1. **Component Reuse** - Any tool/agent/task can be used in multiple crews
2. **Event System** - 10-30+ triggers waiting simultaneously
3. **Database First** - All components queryable and composable
4. **Visual Composition** - Drag-drop on canvas
5. **Full Control** - Self-hosted, no vendor lock-in

---

## Future Enhancements

### Phase 2

1. **Multi-tenant** - Support multiple organizations
2. **Team Collaboration** - Share components across teams
3. **Version History** - Full audit trail
4. **A/B Testing** - Test crew variants

### Phase 3

1. **AI Copilot** - Natural language to build crews
2. **Templates** - Pre-built crew templates
3. **Marketplace** - Share/sell components
4. **Analytics** - Performance insights

### Phase 4

1. **MCP Integration** - Full Model Context Protocol
2. **Distributed Execution** - Run across multiple servers
3. **Custom Nodes** - Plugin system for custom components
4. **Enterprise SSO** - SAML/OAuth integration

---

## Technical Stack Recommendation

### Backend
- **Framework**: FastAPI (Python) - aligns with CrewAI
- **Database**: PostgreSQL
- **ORM**: SQLModel or Prisma
- **Queue**: Redis for event queue

### Frontend
- **Framework**: React or Vue.js
- **Canvas Library**: React Flow or similar
- **State**: Zustand or Pinia
- **API Client**: TanStack Query

### Infrastructure
- **Container**: Docker
- **Deployment**: Kubernetes or Railway
- **Monitoring**: OpenTelemetry + Grafana

---

## Summary

This requirements document outlines a **database-backed visual builder** for CrewAI with:

1. **Separate reusable blocks** - Tool, Agent, Task, Crew as independent entities
2. **Easy composition** - Drag-drop to connect
3. **Event-driven triggers** - 10-30+ events waiting in loop
4. **Visual condition builder** - No code needed
5. **Full reusability** - Any component used anywhere
6. **Self-hosted** - No vendor lock-in

This fills the gap between CrewAI Studio (cloud-only, not database-backed) and writing code directly (not visual, not reusable).