# CrewAI WebUI Builder - Requirements Document

> Created: April 2026 | Version: 1.1 | Status: Implementation Sync

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
│  name      │  name      │  title     │      name         │
│  description│  role      │  description│      process     │
│  type      │  goal      │  expected_ou│      agents[]     │
│            │  backstory │  agent_id   │      tasks[]       │
│            │  tools[]   │  context[] │      x, y (canvas)│
└─────────────┴─────────────┴─────────────┴─────────────────────┘
```

### Canvas-Based Composition

```
┌──────────────────────────────────────────────────────────────────┐
│              FLEXIBLE CREATION (Any Order)                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   STEP 1: Create any entity first (via + button or sidebar)      │
│   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │
│   │ + Create   │ │ + Create   │ │ + Create   │              │
│   │   TOOL     │ │   AGENT    │ │   TASK     │              │
│   └─────────────┘ └─────────────┘ └─────────────┘              │
│         │               │               │                      │
│         ▼               ▼               ▼                      │
│   ┌─────────────────────────────────────────────┐              │
│   │  Database: Empty references (null)           │              │
│   │  - agent.tools = [] (empty)                 │              │
│   │  - task.agent = null (not assigned)        │              │
│   └─────────────────────────────────────────────┘              │
│                                                                  │
│   STEP 2: Link/Compose on Canvas                                 │
│                                                                  │
│   ┌─────────────────────────────────────────────┐              │
│   │  CANVAS                                    │              │
│   │                                             │              │
│   │  🚀 Market Team (crew)                    │              │
│   │  ┌──────────────────────────────┐        │              │
│   │  │ Agents: 👤 Research 👤 Analyze│        │              │
│   │  │ Tasks: 📋 1. Research 📋 2... │        │              │
│   │  └──────────────────────────────┘        │              │
│   │                                             │              │
│   │  Drag Tool → Agent badge = link tool       │              │
│   │  Drag Agent → Crew = add to crew           │              │
│   └─────────────────────────────────────────────┘              │
│                                                                  │
│   KEY: Creation order is flexible - link anytime!              │
└──────────────────────────────────────────────────────────────────┘
```

### Flexible Workflow Examples

#### Example 1: Agent First, Then Tools

```
1. Click [+] on Agents section in sidebar
   - Name: "Researcher"
   - Role: "Research Analyst"
   - Goal: "Research market trends"
   - Backstory: "Expert in market analysis"
   - Save → Agent created (tools = [])

2. Click [+] on Tools section in sidebar
   - Name: "Web Search"
   - Description: "Search the web"
   - Save → Tool created

3. Drag Tool "Web Search" onto Agent "Researcher" badge
   → Tool linked to agent

Result: Agent created before tools even existed!
```

#### Example 2: Task First, Then Agent

```
1. Click [+] on Tasks section in sidebar
   - Title: "Research Competitors"
   - Description: "Find top 5 competitors"
   - Save → Task created (agent = null)

2. Click [+] on Agents section
   - Name: "Business Analyst"
   - Role: "Competitor Analysis Expert"

3. Open Task "Research Competitors" modal
   - Click [+ Assign Agent]
   - Select "Business Analyst"
   → Task now linked to agent

Result: Task existed before agent was created!
```

#### Example 3: Crew Assembly (Final Step)

```
1. Click [+] on Crews section in sidebar
   - Name: "Market Analysis Crew"
   - Process: Sequential

2. On canvas, drag "Researcher" from Agents onto Crew
3. Drag "Business Analyst" onto Crew
4. Drag tasks onto Crew
5. Set Process Mode in crew header

Result: Crew assembled from pre-existing components
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
| `agents` | string[] | Names of linked agents (inverse relation) |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

**UI Operations**:
- Create tool via modal form
- Edit tool via modal
- Link to agents via drag-drop (Tool → Agent badge)
- Remove link via × button in agent modal

### 2. Agent Entity

**Purpose**: Autonomous worker with role, goal, backstory, and tools

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Agent name (unique key in store) |
| `role` | string | Agent's role description |
| `goal` | string | Agent's objective |
| `backstory` | text | Persona context |
| `tools` | string[] | Names of linked tools |
| `verbose` | boolean | Enable verbose logging |

**UI Operations**:
- Create via sidebar [+] button → modal form
- Edit via double-click sidebar item or click badge on canvas
- Link tools via drag-drop (Tool → Agent badge)
- Remove tool link via × in modal

### 3. Task Entity

**Purpose**: Work unit assigned to an agent

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `title` | string | Task title |
| `description` | text | Detailed instructions |
| `agent` | string | Name of assigned agent (null if unassigned) |
| `context` | string[] | Names of dependent tasks |

**UI Operations**:
- Create via sidebar [+] button → modal form
- Edit via double-click sidebar item or click badge on canvas
- Assign agent via modal [+ Assign Agent] button
- Add dependency via modal [+ Add Dependency]

### 4. Crew Entity

**Purpose**: Collection of agents and tasks orchestrated together

**Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `name` | string | Crew name |
| `process` | enum | `sequential` / `hierarchical` |
| `agents` | string[] | Names of agents in this crew |
| `tasks` | string[] | Names of tasks in this crew |
| `x` | number | Canvas X position |
| `y` | number | Canvas Y position |

**UI Operations**:
- Create via sidebar [+] button → modal form
- Edit via click on crew node (not on badge/header)
- Add agents/tasks via drag-drop from sidebar
- Remove via × button on badges
- Toggle process mode via dropdown in edit modal

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

---

## UI/UX Requirements

### Design Philosophy

> **No forms, no code visible** - Pure visual mindmap/Miro-style experience.
> Everything represented as visual nodes with connections.

### Page Structure

1. **Main Canvas View** - Mindmap/miro-style visual editing (PRIMARY)
2. **Sidebar** - Entity library with collapsible sections
3. **Modals** - Create/edit forms, link selection
4. **Help Panel** - Usage instructions overlay

---

### Implemented UI Specification

#### Header Bar

```
┌────────────────────────────────────────────────────────────┐
│  🚀 CrewAI WebUI                        [? Help] [▶ Run]  │
└────────────────────────────────────────────────────────────┘
```

- Logo/title on left
- Help button toggles help overlay
- Run button executes selected crew

#### Sidebar (Left Panel)

```
┌─────────────────────────┐
│  🔍 Search entities...  │
├─────────────────────────┤
│  ▼ 🔧 Tools (3)    [+]  │
│    ├─ web_search        │
│    ├─ pdf_reader        │
│    └─ calculator        │
│                         │
│  ▼ 👤 Agents (4)    [+]  │
│    ├─ Research          │
│    ├─ Analyze           │
│    └─ ...               │
│                         │
│  ▼ 📋 Tasks (4)    [+]  │
│  ▼ 🚀 Crews (2)    [+]  │
└─────────────────────────┘
```

**Features:**
- Search/filter input at top
- Collapsible sections (click header to toggle ▼/▶)
- Entity count in parentheses
- [+] button to create new entity
- Sidebar items are draggable to canvas

**Drag Behavior:**
- Sidebar items are `draggable="true"`
- `dragstart` event captures type and name
- `dragend` event cleans up drag state

#### Canvas (Main Area)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  │
│   ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  │
│   ·  ·  ┌─────────────────────────────────────────┐  ·  ·  ·  │
│   ·  ·  │ 🚀 Market Team                         │  ·  ·  ·  │
│   ·  ·  │ [▶ Sequential]                        │  ·  ·  ·  │
│   ·  ·  ├─────────────────────────────────────────┤  ·  ·  ·  │
│   ·  ·  │ Agents                                │  ·  ·  ·  │
│   ·  ·  │ ┌────────────┐ ┌────────────┐      │  ·  ·  ·  │
│   ·  ·  │ │👤 Research │ │👤 Analyze  │      │  ·  ·  ·  │
│   ·  ·  │ │ 🔧 web     │ │ 🔧 pdf     │      │  ·  ·  ·  │
│   ·  ·  │ └────────────┘ └────────────┘      │  ·  ·  ·  │
│   ·  ·  │                                      │  ·  ·  ·  │
│   ·  ·  │ Tasks                                │  ·  ·  ·  │
│   ·  ·  │ 📋 1. Research  📋 2. Analyze       │  ·  ·  ·  │
│   ·  ·  └─────────────────────────────────────────┘  ·  ·  ·
│   ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·  ·
└─────────────────────────────────────────────────────────────────┘
```

**Background:** Dot grid pattern (radial-gradient, 40px spacing)

**Panning:**
- Click and drag on canvas background (not on nodes)
- Tracks `offsetX`, `offsetY` on canvas inner transform

**Crew Nodes:**
- Positioned absolutely with `left`, `top`
- Header area (colored) = drag handle (only header, not entire node)
- Body contains Agents and Tasks sections
- Click node (not badge) → opens edit modal
- Click agent/task badge → opens respective edit modal
- × button on badges removes from crew

**Agent Badge Layout:**
```
┌─────────────────────────┐
│ 👤 AgentName      [×]  │
│   🔧 tool1 🔧 tool2   │  (if tools linked)
└─────────────────────────┘
```

- Two-line layout: name + tools below
- Tools shown as small colored spans

**Task Badge Layout:**
```
📋 1. TaskName [×]
```
- Numbered by position in crew.tasks array
- × button removes from crew

#### Drag & Drop Behaviors

**From Sidebar to Canvas:**
| Drag | Drop Target | Action |
|------|-------------|--------|
| Tool | Canvas (empty) | No action |
| Tool | Agent badge | Link tool to agent |
| Agent | Crew node | Add agent to crew |
| Task | Crew node | Add task to crew |
| Crew | Canvas (empty) | Reposition crew |

**Implementation Notes:**
- `dragover` on crew node highlights with border
- `drop` on crew node adds entity to crew
- Tool→Agent badge drop links tool to agent (updates both store.tools and store.agents)
- `dragend` on document resets `draggedData = null` and clears highlights

#### Process Mode

**Sequential (default):**
- Tasks run left-to-right in order
- Badge shows: ▶ Sequential

**Hierarchical:**
- Manager coordinates agents
- Badge shows: ⭐ Hierarchical
- Node has purple border instead of red

#### Modal Dialogs

**Create Modal:**
```
┌─────────────────────────────────┐
│  Create Tool / Agent / Task / Crew│
├─────────────────────────────────┤
│  [form fields based on type]   │
│                                 │
│  [Cancel]              [Create] │
└─────────────────────────────────┘
```

**Edit Modal (Agent example):**
```
┌─────────────────────────────────┐
│  Edit Agent                       │
├─────────────────────────────────┤
│  Name: [___________]            │
│  Role: [___________]            │
│  Goal: [___________]            │
│  Backstory: [__________]        │
│                                 │
│  Tools (click × to remove)     │
│  ┌──────────────────────────┐   │
│  │ 🔧 web_search [×]        │   │
│  │ 🔧 calculator [×]        │   │
│  └──────────────────────────┘   │
│  [+ Add Tool]                   │
│                                 │
│  [Cancel]              [Save]   │
└─────────────────────────────────┘
```

**Link Modal:**
```
┌─────────────────────────────────┐
│  Add Agent                        │
├─────────────────────────────────┤
│  ┌──────────────────────────┐   │
│  │ 👤 Research              │   │
│  └──────────────────────────┘   │
│  ┌──────────────────────────┐   │
│  │ 👤 Analyze               │   │
│  └──────────────────────────┘   │
│                                 │
│  (Cancel to close)             │
└─────────────────────────────────┘
```

**Modal Event Delegation:**
- Modal body contains dynamically rendered badges
- `click` event on modal body uses `e.target.closest()` to find remove buttons
- `data-action` attributes distinguish action types
- After action, modal refreshes via `openEditModal()`

#### Help Panel

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  🚀 CrewAI WebUI Help                                      │
│                                                             │
│  Sidebar                                                   │
│  • Click section headers to collapse/expand                │
│  • Use search to filter entities                          │
│  • Drag entities from sidebar onto canvas                 │
│  • Click + to create new entity                           │
│                                                             │
│  Canvas                                                    │
│  • Drag canvas background to pan                          │
│  • Drag crew nodes to reposition (header only)            │
│  • Click crew to view/edit                                │
│  • Drag entity onto crew to add it                       │
│                                                             │
│  Linking Entities                                          │
│  • Drag Tool onto Agent → links tool to agent             │
│  • Drag Task onto Agent → assigns agent to task           │
│  • Drag Task onto Task → creates dependency               │
│  • Click badges in modal to remove links                  │
│                                                             │
│  Process Modes                                             │
│  • Sequential: Tasks run left-to-right                    │
│  • Hierarchical: Manager coordinates agents               │
│                                                             │
│  [Close]                                                   │
└─────────────────────────────────────────────────────────────┘
```

#### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Escape | Close modal or help panel |

#### Click Interactions

**Single Click:**
- Click crew header → drag to move
- Click crew body (not badge) → open crew edit modal
- Click agent badge → open agent edit modal
- Click task badge → open task edit modal
- Click × on badge → remove from crew

**Double Click:**
- Double-click sidebar item → open edit modal

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
POST   /api/tools/:id/agents - Link tool to agent
DELETE /api/tools/:id/agents/:agentId - Unlink tool from agent

# Agents
POST   /api/agents            - Create agent
GET    /api/agents           - List agents
GET    /api/agents/:id        - Get agent
PUT    /api/agents/:id       - Update agent
DELETE /api/agents/:id      - Delete agent
POST   /api/agents/:id/tools - Add tool to agent
DELETE /api/agents/:id/tools/:toolId - Remove tool from agent

# Tasks
POST   /api/tasks             - Create task
GET    /api/tasks            - List tasks
GET    /api/tasks/:id        - Get task
PUT    /api/tasks/:id        - Update task
DELETE /api/tasks/:id       - Delete task
POST   /api/tasks/:id/agent - Assign agent
DELETE /api/tasks/:id/agent - Unassign agent
POST   /api/tasks/:id/context - Add context task
DELETE /api/tasks/:id/context/:contextId - Remove context task

# Crews
POST   /api/crews            - Create crew
GET    /api/crews            - List crews
GET    /api/crews/:id        - Get crew
PUT    /api/crews/:id       - Update crew
DELETE /api/crews/:id      - Delete crew
POST   /api/crews/:id/agents - Add agent to crew
DELETE /api/crews/:id/agents/:agentId - Remove agent from crew
POST   /api/crews/:id/tasks - Add task to crew
DELETE /api/crews/:id/tasks/:taskId - Remove task from crew
POST   /api/crews/:id/run   - Run crew

# Triggers
POST   /api/triggers        - Create trigger
GET    /api/triggers        - List triggers
GET    /api/triggers/:id    - Get trigger
PUT    /api/triggers/:id    - Update trigger
DELETE /api/triggers/:id   - Delete trigger
POST   /api/triggers/:id/enable - Enable trigger
POST   /api/triggers/:id/disable - Disable trigger
```

---

## Data Models

### Store Structure (Frontend State)

```javascript
const store = {
    tools: {
        'tool_name': {
            name: 'tool_name',
            description: 'description',
            type: 'decorator' | 'basetool' | 'mcp' | 'http',
            agents: ['agent_name', ...]  // inverse relation
        },
        // ...
    },
    agents: {
        'agent_name': {
            name: 'agent_name',
            role: 'role description',
            goal: 'agent objective',
            backstory: 'persona context',
            tools: ['tool_name', ...]
        },
        // ...
    },
    tasks: {
        'task_title': {
            title: 'task_title',
            description: 'task description',
            agent: 'agent_name' | null,
            context: ['other_task_title', ...]
        },
        // ...
    },
    crews: {
        'crew_name': {
            name: 'crew_name',
            process: 'sequential' | 'hierarchical',
            agents: ['agent_name', ...],
            tasks: ['task_title', ...],
            x: 50,   // canvas position
            y: 50
        },
        // ...
    },
    canvas: {
        offsetX: 0,
        offsetY: 0,
        scale: 1
    }
};
```

---

## Workflows

### Workflow 1: Create and Assemble a Crew

1. **Create Agents** (any order, before or after tools)
   - Click [+] on Agents section
   - Fill form: name, role, goal, backstory
   - Click Create

2. **Create Tools** (any order, before or after agents)
   - Click [+] on Tools section
   - Fill form: name, description, type
   - Click Create

3. **Link Tools to Agents** (drag-drop)
   - Drag tool from sidebar
   - Drop on agent badge in a crew (or in agent edit modal)
   - Both tool.agents and agent.tools updated

4. **Create Tasks**
   - Click [+] on Tasks section
   - Fill form: title, description
   - Optionally assign agent via [+ Assign Agent]

5. **Assemble Crew**
   - Click [+] on Crews section
   - Fill form: name, process
   - Drag agents/tasks from sidebar onto crew node
   - Or use [+ Add Agent] / [+ Add Task] in edit modal

6. **Run Crew**
   - Click [▶ Run Crew] button
   - In full implementation, executes selected crew

### Workflow 2: Edit Existing Configuration

1. **Edit Agent**
   - Double-click agent in sidebar OR click agent badge in crew
   - Modal opens with current values
   - Edit fields, add/remove tools
   - Click Save

2. **Edit Task**
   - Double-click task in sidebar OR click task badge in crew
   - Modal opens with current values
   - Edit fields, reassign agent, adjust dependencies
   - Click Save

3. **Edit Crew**
   - Click on crew node (not on badges)
   - Modal opens with current values
   - Change name, process mode, add/remove agents/tasks
   - Click Save

### Workflow 3: Manage Task Dependencies

1. Open task edit modal
2. Click [+ Add Dependency]
3. Select context task from list
4. Repeat for multiple dependencies
5. Click × on badge to remove dependency

---

## Comparison with Existing Solutions

| Feature | CrewAI Studio | Langflow | **This Builder** |
|---------|--------------|----------|------------------|
| Visual canvas | Limited | Yes | **Yes, Miro-style** |
| Database persistence | No | Partial | **Yes** |
| Flexible creation order | No | No | **Yes** |
| Event-driven triggers | No | No | **Yes** |
| Crew-to-crew chaining | No | No | **Yes** |
| Per-crew process mode | No | No | **Yes** |
| Separate reusable entities | No | Partial | **Yes** |
| Drag-drop tool→agent | No | No | **Yes** |
| No-code interface | Limited | Yes | **Yes** |

---

## Future Enhancements

1. **Visual Chain Lines** - Draw lines between crews to show trigger relationships
2. **Execution History** - View past runs with logs and outputs
3. **Import/Export** - Save/load configurations as JSON
4. **Team Collaboration** - Multiple users editing simultaneously
5. **LLM Configuration UI** - Visual picker for providers/models
6. **Output Schema Builder** - Define expected crew outputs visually
7. **Condition Builder UI** - Point-and-click trigger condition creation
8. **Mini-map** - Overview of large canvases
9. **Undo/Redo** - Full history support
10. **Keyboard Shortcuts** - Full keyboard navigation

---

*Document Version: 1.1 - Updated to reflect actual test-ui.html implementation*
